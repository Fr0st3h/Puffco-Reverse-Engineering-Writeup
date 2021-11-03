
# Puffco-Reverse-Engineering
A writeup on how I reversed engineered the puffco peak pro

------
# A Message to Puffco
>In the U.S., Section 103(f) of the Digital Millennium Copyright Act (DMCA) [(17 USC § 1201 (f) - Reverse Engineering)](https://www.law.cornell.edu/uscode/text/17/1201) specifically states that it is legal to reverse engineer and circumvent the protection to achieve interoperability between computer programs (such as information transfer between applications). Interoperability is defined in paragraph 4 of Section 103(f).
>
>It is also often lawful to reverse-engineer an artifact or process as long as it is obtained legitimately. If the software is patented, it doesn't necessarily need to be reverse-engineered, as patents require a public disclosure of invention. It should be mentioned that, just because a piece of software is patented, that does not mean the entire thing is patented; there may be parts that remain undisclosed.


Puffco, I love your products, and I mean no harm in releasing this information. I only did this as a side project so I can control the peak pro however I want. I decided to publish my findings so that anyone else who is looking to do the same has a place to start. Long story short, __please don't sue me, or DMCA this repo__. If you wish for me to take it down, __please email me or leave a issue on this repo stating that you would like it to be removed, and I will happily do so__.

Now on to the documentation/writeup!
# My Findings
### Checklist of things I've accomplished.
- [x] Change lantern Colour.
- [x] Send commands (preheat, boost, cycle profile)
- [x] Change profiles (1-4) Colour, temperature, time.
- [x] Set profile colour to any sort of animation.
- [x] Set base, logo, main, glass LED brightness.
- [x] Data not shown in the app.
	- Dabs remaining until battery dies.
	- Time until battery is charged.
	- Total uptime.
	- Total preheat cycle time.
	- Device birthday.
- [x] Knowing what every Bluetooth characteristic does.
- [x] Found hidden features (New device, upcoming features).
- [x] Their staging URL, which actually contained stuff never released.

### How I started reverse engineering the peak pro.
I started reverse engineering with no plan in mind, I just wanted to poke around the Bluetooth characteristics and see what I could accomplish. I started off by using a GATT app on my phone, changing the values in the Puffco app and then seeing what changes in the GATT app. In the end I ended up extracting the main react native file from the android app and found out what every characteristic does. I will describe the characteristics I mainly looked into. At the end of this writeup will be the base UUID as well as the offsets.

### Looking through the react native android app
When looking through the react native file, I found a lot of interesting things like their staging URL, every characteristic offset, new devices, upcoming app updates/features, and API URLS. The staging URL was quite interesting, when looking through it the first time I discovered chamber type values which contained the following (Classic, Herbal, Performance). This was interesting because at the time there was only one chamber type released (This was before the 3D was even announced). Recently there was a change to their staging URL showing the new LE-2 which was announced, just not show on Roger's live (I can say this LE is looking AMAZING) I will not post what it looks like in respect to the product team over at Puffco :). 

**Note: The following values are decimal not hexadecimal.**

###  How the lights work (Lantern and Profile)
The order in which the bytes are will change depending on if rainbow is enabled or not.

    Example:
    	0xFF, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00 
    	#RED, GREEN, BLUE, RAINBOW, MODE, IDK, IDK, IDK
    Example with rainbow:
    	0xFF, 0x1E, 0x0, 0x01, 0x00, 0x00, 0x00
    	#BRIGHTNESS, SPEED, MODE, RAINBOW, IDK, IDK, IDK
    	
    Modes i've found:
	    0x00 - Preserve?
	    0x01 - Static
	    0x05 - Breathing
	    0x06 - Rising
	    0x07 - Circling
	    0x15 - Circling Slow
### Enabling the Lantern
This one was easy, its 4 bytes long and the first byte is either a 0 or a 1.

    00, 00, 00, 00 = Turns the lantern off
    01, 00, 00, 00 = Turns the lantern on

### Setting LED Brightness
There's 4 segments to the brightness, each one goes to 255 and controls a separate LED. 

    FF, FF, FF, FF
    RING LED, UNDER GLASS LED, MAIN LED, BATTERY LED

### Set Custom Profile Values
There's only one set of offsets to control every profile value (temperature, time, colour, preheat colour, name)
To change the selected profile, you need to set 0-3 to the heatCyclePointer characteristic (offset 61). This will change which profile is active in the app, not current selected profile on the peak.

    Changing heatCyclePointer
    	0x00, 0x00, 0x00, 0x00 = Profile 1
    	0x01, 0x00, 0x00, 0x00 = Profile 2
    	0x02, 0x00, 0x00, 0x00 = Profile 3
    	0x03, 0x00, 0x00, 0x00 = Profile 4
    	
    Following are examples
        heatCycleName (Offset 62)
        	- The profile name can be set as ASCII.
        heatCycleTemp (Offset 64)
        	- 0x00, 0x80, 0x8F, 0x43 (Little Endian Float, value is 287 (Celsius))
        heatCycleTime (Offset 64)
        	- 0x00, 0x00, 0xb8, 0x42 (Little Endian Float, value is 92 (Seconds))
        heatCycleColor (Offset 65)
        	- 0x16, 0xE9, 0x9C, 0x00, 0x00, 0x00, 0x00, 0x00 (Greenish Colour)
        heatCycleActiveColor (Offset 6B)
        	- 0x00, 0x00, 0x00, 0xFF, 0x00, 0x00, 0x00, 0x00 (This is being set to last colour I think)

it seems heatCyclePreheatColor doesn't work, ActiveColor seems to override it or something.

### Charging State Values
    0 - Charging (0x00, 0x00, 0x00, 0x00)
    2 - Done Charging (0x00, 0x00, 0x00, 0x40)
    3 - Over Temp? (0x00, 0x00, 0x40, 0x40)
    4 - Disconnected (0x00, 0x00, 0x80, 0x40)

### Sending commands to the device
Command List:

0 - master off - 0x00, 0x00, 0x00, 0x00
1 - sleep - 0x00, 0x00, 0x80, 0x3F
2 -idle - 0x00, 0x00, 0x00, 0x040
3 - tempSelectBegin - 0x00, 0x00, 0x40, 0x40
4 - tempSelectStop - 0x00, 0x00, 128 0x40
5 - showBatterylevel - 0x00, 0x00, 0xA0, 0x40
6 - showVersion - 0x00, 0x00, 0xC0, 0x40
7 - heatCycleStart - 0x00, 0x00, 0xE0, 0x40
8 - heatCycleAbort - 0x00, 0x00, 0x00, 0x41
9 - heatCycleBoost - 0x00, 0x00, 0x10, 0x41
10 - factoryTest - 0x00, 0x00, 0x20, 0x41
11 - bonding - 0x00, 0x00, 0x30, 0x41





	    
	
	




### Every Single Bluetooth characteristic name & offset
The base characteristic UUID is f9a98c15-c651-4f34-b656-d100bf5800

    mfgDate: '00',
    EUID: '01',
    gitHash: '02',
    batterySOC: '20',
    batteryVoltage: '21',
    operatingState: '22',
    stateElapsedTime: '23',
    stateTotalTime: '24',
    heaterTemp: '25',
    heaterTempCommand: '26',
    activeLEDColor: '27',
    heaterPower: '28',
    heaterDuty: '29',
    heaterVoltage: '2a',
    heaterCurrent: '2b',
    safetyThermalEstTemp: '2c',
    heaterResistance: '2d',
    batteryChargeCurrent: '2e',
    totalHeatCycles: '2f',
    totalHeatCycleTime: '30',
    batteryChargeState: '31',
    batterChargeElapseTime: '32',
    batterChargeEstTimeToFull: '33',
    batteryTemp: '34',
    upTime: '35',
    magneticField: '36',
    inputCurrent: '37',
    batteryCapacity: '38',
    batteryCurrent: '39',
    approxDabsRemaining: '3a',
    dabsPerDay: '3b',
    rawHeaterTemp: '3c',
    rawHeaterTempCommand: '3d',
    batteryChargeSource: '3e',
    chamberType: '3f',
    modeCommand: '40',
    heatCycleSelect: '41',
    stealthMode: '42',
    deleteBondings: '43',
    UTCTime: '44',
    temperatureOverride: '45',
    timeOverride: '46',
    lanternPatternSetting: '47',
    lanternColorSetting: '48',
    lanternTimeSetting: '49',
    lanternStart: '4a',
    LEDbrightness: '4b',
    readyModeCycleSelect: '4c',
    deviceName: '4d',
    deviceBirthday: '4e',
    factoryReset: '4f',
    terminateBondingAnimation: '50',
    tripHeatCycles: '51',
    tripHeatCycleTime: '52',
    previewColor: '53',
    heatCycleCount: '60',
    heatCyclePointer: '61',
    heatCycleName: '62',
    heatCycleTemp: '63',
    heatCycleTime: '64',
    heatCycleColor: '65',
    heatCyclePattern: '66',
    heatCycleBoostTemp: '67',
    heatCycleBoostTime: '68',
    heatCycleThreshholdTemp: '69',
    heatCyclePreheatColor: '6A',
    heatCycleActiveColor: '6B',
    auditLogPointer: 'c1',
    auditLogEntry: 'c2',
    auditLogBegin: 'c3',
    auditLogEnd: 'c4',
    faultLogPointer: 'd1',
    faultLogEntry: 'd2',
    faultLogBegin: 'd3',
    faultLogEnd: 'd4'

	
