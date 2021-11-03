
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
- [x] Change profiles (1-4) Colour, temperature, time.
- [x] Set profile colour to any sort of animation.
- [x] Set base, logo, main, glass LED brightness.
- [x] Data not shown in the app.
	- Dabs remaining until battery dies.
	- Time until battery is charged.
	- Total uptime.
	- Total preheat cycle time.
	- Device birthday.
- [x] Knowing what every bluetooth characteristic does.
- [x] Found hidden features (New device, upcoming features).
- [x] Their staging URL, which actually contained stuff never released.

### Every Single bluetooth characteristic name & offset
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

	
