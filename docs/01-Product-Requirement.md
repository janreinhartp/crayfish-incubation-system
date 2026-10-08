# 01 Product Requirements

## 1. Project Overview

The Crayfish Incubator is an automated 60-gallon water-based incubation system designed to provide stable and controlled environmental conditions for crayfish incubation.

The system will continuously monitor water temperature, dissolved oxygen, pH, water level, and water flow.

It will provide automatic temperature control through heating and Peltier-based cooling, continuous water circulation, continuous aeration, and automatic water refilling.

The system will provide two user interfaces:

- Local touchscreen HMI
- Wi-Fi web interface

The system will record historical operating data using a local microSD card and a TinyRTC real-time clock.

The web interface shall allow users to export historical data in CSV format.

The primary controller will be an ESP32-S3.

## 2. Product Objectives

The system shall:

- Maintain the configured water temperature.
- Heat the water when the temperature is too low.
- Cool the water using two TEC1-12706 Peltier modules when the temperature is too high.
- Continuously circulate water.
- Continuously aerate the water.
- Monitor dissolved oxygen.
- Monitor pH.
- Monitor the main tank water level.
- Monitor the refill reservoir level.
- Monitor water circulation flow.
- Monitor refill flow.
- Automatically refill the main tank.
- Prevent the main tank from overfilling.
- Protect the heater from low-water conditions.
- Protect the Peltier system from excessive hot-side temperature.
- Detect circulation failures.
- Detect refill failures.
- Maintain accurate local date and time.
- Timestamp sensor records and system events.
- Provide local touchscreen monitoring and control.
- Provide remote monitoring through a Wi-Fi web interface.
- Store historical sensor data.
- Store equipment states.
- Store alarm events.
- Export historical data as CSV.
- Provide operator alarms for abnormal conditions.

## 3. System Capacity

### 3.1 Main Tank

Nominal capacity:

- 60 US gallons
- Approximately 227 liters

The system shall operate below the physical maximum tank capacity.

The operating level shall provide sufficient space for water movement and emergency overflow protection.

### 3.2 Refill Reservoir

The system shall use a separate elevated water reservoir.

The reservoir shall supply the main tank through a normally-closed solenoid valve.

The reservoir shall have independent water-level monitoring.

## 4. System Parameters

The system shall monitor the following primary parameters:

| Parameter | Monitoring | Control | Logging |
|---|---|---|---|
| Water temperature | Yes | Heater/Cooling | Yes |
| Dissolved oxygen | Yes | Alarm | Yes |
| pH | Yes | Alarm | Yes |
| Main water level | Yes | Refill | Yes |
| Refill reservoir level | Yes | Refill protection | Yes |
| Main water flow | Yes | Safety | Yes |
| Refill flow | Yes | Refill verification | Yes |
| Peltier heatsink temperature | Yes | Cooling safety | Yes |
| Ambient temperature | Yes | Monitoring | Yes |
| Date and time | Yes | Timestamping | Yes |

## 5. Water Temperature

The system shall continuously monitor the main tank water temperature.

The system shall use two independent water temperature sensors.

The temperature system shall:

- Display current water temperature.
- Record temperature history.
- Provide configurable temperature targets.
- Provide configurable heating limits.
- Provide configurable cooling limits.
- Control the heater.
- Control the Peltier cooling system.
- Detect abnormal temperature conditions.
- Detect temperature sensor failures.
- Generate temperature alarms.

The initial system shall use waterproof DS18B20 temperature sensors.

Temperature control values shall remain configurable because the appropriate operating temperature depends on the crayfish species and incubation requirements.

## 6. Heating System

The system shall provide active water heating.

The heating subsystem shall use an aquarium-rated heater with an independent thermostat.

The ESP32-S3 shall control the heater through an electrically isolated switching device.

The controller shall:

- Activate heating when water temperature falls below the configured heating threshold.
- Disable heating when the target temperature is reached.
- Disable heating when the water level is critically low.
- Disable heating when required circulation is unavailable.
- Disable heating when the temperature sensor is invalid.
- Record heater state changes.

The independent heater thermostat shall provide an additional layer of protection.

## 7. Cooling System

The system shall provide active water cooling using two TEC1-12706 Peltier modules.

Cooling architecture:

```text
Water
  ↓
Water Block
  ↓
TEC1-12706 × 2
  ↓
Large Hot-Side Heatsink
  ↓
High-Airflow Fans
```

The cooling subsystem shall include:

- Two TEC1-12706 modules.
- Water-cooled cold side.
- Large hot-side heatsink.
- Forced-air cooling.
- Independent control of each TEC.
- Hot-side temperature monitoring.
- Thermal protection.

The system shall reduce or disable Peltier operation when the hot-side temperature exceeds the configured safety limit.

The hot-side fans shall continue operating when required to remove residual heat.

## 8. Water Circulation

The system shall continuously circulate water through the main water loop.

The circulation system shall include:

- Circulation pump.
- Flow sensor.
- Filtration.
- Peltier water block.
- Return line.

The controller shall monitor flow continuously.

The system shall detect:

- No flow.
- Low flow.
- Unexpected flow.
- Circulation pump failure.

If insufficient flow is detected, the system shall:

- Generate an alarm.
- Disable the heater.
- Disable Peltier cooling where required for system protection.
- Record the fault.

## 9. Aeration

The system shall provide continuous aeration independently from the circulation system.

The aeration system shall include:

- Air pump.
- Air manifold.
- Multiple air stones or equivalent diffusers.

The aeration system shall normally remain active during incubation.

The system shall use dissolved oxygen measurements to detect inadequate oxygen conditions.

A low dissolved oxygen condition shall generate an alarm.

## 10. Dissolved Oxygen Monitoring

The system shall continuously monitor dissolved oxygen.

The DO subsystem shall use a suitable laboratory or industrial dissolved oxygen probe and transmitter.

The preferred interface is RS485 Modbus RTU.

The system shall:

- Display DO concentration.
- Record DO history.
- Provide configurable low-DO limits.
- Generate low-DO alarms.
- Detect communication failures.
- Detect invalid sensor readings.

The system shall not automatically inject oxygen or chemicals in Version 1.

## 11. pH Monitoring

The system shall continuously monitor water pH.

The pH subsystem shall use a replaceable laboratory or industrial pH probe with an appropriate transmitter.

The preferred interface is RS485 Modbus RTU.

The system shall:

- Display current pH.
- Record pH history.
- Provide configurable high and low limits.
- Generate pH alarms.
- Detect sensor communication failures.
- Detect invalid sensor readings.

Automatic chemical dosing shall not be included in Version 1.

## 12. Main Tank Water-Level Monitoring

The system shall monitor the main tank water level.

The system shall use:

- Continuous water-level measurement.
- Independent low-level protection.
- Independent high-level protection.

The continuous measurement shall provide an approximate tank level for the HMI and web interface.

The independent level switches shall provide safety protection even if the primary level sensor fails.

The system shall detect:

- Critical low level.
- Normal level.
- Refill level.
- High level.
- Emergency high level.

## 13. Refill Reservoir

The system shall use an elevated water reservoir for automatic water replenishment.

The refill subsystem shall include:

- Elevated water reservoir.
- Reservoir level monitoring.
- Normally-closed solenoid valve.
- Refill flow sensor.
- Manual isolation valve.
- Appropriate plumbing protection.

The system shall prevent automatic refilling when the reservoir is empty.

## 14. Automatic Water Refill

The controller shall automatically maintain the main tank water level.

Basic sequence:

```text
Main tank level LOW
        ↓
Check refill reservoir
        ↓
Reservoir available
        ↓
Open solenoid valve
        ↓
Monitor refill flow
        ↓
Monitor main tank level
        ↓
Target level reached
        ↓
Close solenoid valve
```

The controller shall stop refilling when:

- Target level is reached.
- High-level protection is activated.
- Refill reservoir is empty.
- Refill flow is not detected.
- A refill fault occurs.
- A system safety condition requires refill shutdown.

The solenoid valve shall be normally closed.

Loss of controller power shall therefore close the refill valve.

## 15. Water Flow Monitoring

The main circulation system shall use an inline flow sensor.

The system shall display flow rate in L/min.

The system shall record flow history.

The controller shall detect:

- Pump running without flow.
- Flow below minimum.
- Unexpected flow loss.
- Flow sensor failure.

The system shall use flow status as a safety interlock for heating and cooling.

## 16. Real-Time Clock

The system shall include a TinyRTC real-time clock module.

The RTC shall provide persistent date and time information for local operation and data logging.

The RTC shall include a backup battery.

The system shall use the RTC for:

- Sensor data timestamps.
- Alarm timestamps.
- Refill event timestamps.
- Equipment state change timestamps.
- System startup timestamps.
- System shutdown timestamps.
- Historical data records.
- CSV export timestamps.

The system shall continue to maintain valid timestamps when Wi-Fi is unavailable.

The ESP32-S3 internal timing system shall handle control intervals and timing functions such as:

- Sensor sampling.
- Temperature control.
- Refill timeout.
- Flow timeout.
- Alarm delays.
- Peltier control.

The TinyRTC shall not be used as the primary timing mechanism for fast control loops.

## 17. Data Logging

The system shall store historical operating data locally using a microSD card.

The TinyRTC shall provide timestamps for stored records.

The system shall record:

- Date.
- Time.
- Water temperature 1.
- Water temperature 2.
- Dissolved oxygen.
- pH.
- Main tank level.
- Refill reservoir level.
- Main flow.
- Refill flow.
- Ambient temperature.
- Peltier heatsink temperature.
- Heater state.
- TEC 1 state.
- TEC 2 state.
- Cooling fan state.
- Circulation pump state.
- Aeration state.
- Refill valve state.
- Alarm state.

The initial permanent logging interval shall be 1 minute.

Real-time control shall use faster sensor sampling where required.

The system shall organize logged records so they can be exported through the web interface.

## 18. CSV Data Export

The web interface shall provide CSV export functionality.

The user shall be able to export historical system data from the ESP32-S3 web interface.

The CSV export shall include:

- Date.
- Time.
- Water temperature 1.
- Water temperature 2.
- Dissolved oxygen.
- pH.
- Main tank level.
- Refill reservoir level.
- Main flow.
- Refill flow.
- Ambient temperature.
- Peltier heatsink temperature.
- Heater state.
- TEC 1 state.
- TEC 2 state.
- Cooling fan state.
- Circulation pump state.
- Aeration state.
- Refill valve state.
- Alarm state.

The web interface should provide date and time range selection for exports.

Example:

```text
Export Data

Start:
2026-10-01 00:00

End:
2026-10-07 23:59

[ Export CSV ]
```

The exported file shall use a standard CSV format that can be opened using:

- Microsoft Excel.
- Google Sheets.
- LibreOffice Calc.
- Other spreadsheet applications.

The CSV file shall include a header row.

Example structure:

```text
timestamp,water_temp_1,water_temp_2,do,ph,main_level,refill_level,main_flow,refill_flow,ambient_temp,peltier_heatsink_temp,heater,tec1,tec2,cooling_fans,circulation_pump,aeration,refill_valve,alarm
```

The system shall not require cloud connectivity to generate CSV exports.

## 19. HMI

The system shall provide a local touchscreen HMI.

The HMI shall provide a main dashboard containing:

```text
Water Temperature
Dissolved Oxygen
pH
Main Tank Level
Refill Tank Level
Main Flow
Refill Flow
Heater Status
TEC 1 Status
TEC 2 Status
Cooling Fan Status
Circulation Pump Status
Aeration Status
Refill Valve Status
Alarm Status
System Mode
Date and Time
```

The HMI shall include the following screens:

1. Dashboard
2. Water Quality
3. Temperature
4. Equipment Status
5. Historical Data
6. Alarm History
7. Settings
8. Manual Control
9. System Information

## 20. Web Interface

The ESP32-S3 shall provide a Wi-Fi web interface.

The web interface shall provide:

- Live monitoring.
- Equipment status.
- Sensor graphs.
- Historical data.
- Alarm history.
- System settings.
- Manual control.
- System status.
- Current date and time.
- CSV data export.

The historical data page shall allow the user to:

- View sensor trends.
- Select a time range.
- View historical records.
- Export selected historical data as CSV.

The web interface shall be usable from:

- Desktop computers.
- Tablets.
- Mobile phones.

The system shall not depend on an external cloud service for normal operation.

## 21. Alarm System

The system shall generate alarms for:

### Temperature

- High water temperature.
- Low water temperature.
- Temperature sensor failure.

### Dissolved Oxygen

- Low dissolved oxygen.
- DO sensor failure.

### pH

- High pH.
- Low pH.
- pH sensor failure.

### Water Level

- Main tank low level.
- Main tank high level.
- Main tank emergency high level.
- Refill reservoir low level.

### Flow

- Low circulation flow.
- Circulation flow failure.
- Refill flow failure.

### Cooling

- Peltier heatsink over-temperature.
- Cooling system fault.
- Cooling fan fault.

### Equipment

- Circulation pump fault.
- Refill valve fault.
- Sensor communication fault.

### RTC

- RTC communication failure.
- Invalid RTC time.
- RTC backup power failure, if detectable.

Alarm events shall be stored in the data log with date and time.

## 22. Safety Interlocks

The system shall implement hardware and software safety interlocks.

### Low Water

```text
Low Water
    ↓
Heater OFF
Cooling OFF
Refill ON
Alarm
```

The refill system shall only activate when the refill reservoir is available and no refill safety fault exists.

### High Water

```text
High Water
    ↓
Refill Valve CLOSED
Alarm
```

### No Circulation

```text
Low Flow
    ↓
Heater OFF
Cooling OFF
Alarm
```

### Peltier Over-Temperature

```text
High Heatsink Temperature
    ↓
TEC 1 OFF
TEC 2 OFF
Fans ON
Alarm
```

### Sensor Failure

```text
Critical Sensor Failure
    ↓
Affected Control OFF
Alarm
```

## 23. Operating Modes

The system shall support:

### Automatic Mode

The system automatically manages:

- Temperature.
- Heating.
- Cooling.
- Water circulation.
- Aeration.
- Water refill.
- Safety functions.

### Manual Mode

The operator may manually operate supported equipment through the HMI or web interface.

Safety interlocks shall remain active.

### Maintenance Mode

Maintenance personnel may disable selected automatic functions.

Critical safety functions shall remain active.

### Alarm Mode

The system shall identify active faults and place affected equipment into a safe state.

## 24. Manual Controls

The HMI and web interface shall provide controlled manual operation of:

- Heater.
- TEC 1.
- TEC 2.
- Cooling fans.
- Circulation pump.
- Aeration pump.
- Refill solenoid.

Manual operation shall not bypass critical safety interlocks.

## 25. Power and Electrical Safety

The system shall separate low-voltage control electronics from high-power equipment.

The electrical system shall include:

- Main circuit protection.
- RCD/GFCI protection.
- Individual branch protection where appropriate.
- DC fusing.
- Emergency stop.
- Proper grounding.
- Enclosed electrical connections.
- Thermal protection for the Peltier system.

High-power loads shall not be driven directly from ESP32 GPIO pins.

## 26. System Availability

The system shall be designed for continuous operation.

Target operation:

```text
24 hours/day
7 days/week
```

The system shall support extended unattended operation while providing alarms for conditions requiring operator intervention.

## 27. Version 1 Scope

Version 1 shall include:

### Controller and Interface

- ESP32-S3 controller.
- Local touchscreen HMI.
- Wi-Fi web interface.
- TinyRTC real-time clock.
- MicroSD storage.
- CSV data export.

### Water System

- 60-gallon main tank.
- Elevated refill reservoir.
- Water circulation pump.
- Filtration.
- Aeration pump.
- Refill solenoid valve.
- Main circulation flow sensor.
- Refill flow sensor.

### Sensors

- Two DS18B20 water temperature sensors.
- Dissolved oxygen sensor and transmitter.
- pH sensor and transmitter.
- Main tank continuous level sensor.
- Main tank low-level safety switch.
- Main tank high-level safety switch.
- Refill reservoir level sensor.
- Peltier heatsink temperature sensor.
- Ambient temperature sensor.

### Temperature Control

- Aquarium heater with independent thermostat.
- Two TEC1-12706 Peltier modules.
- Peltier water block.
- Large hot-side heatsink.
- High-airflow cooling fans.
- Independent TEC control.

### Software

- Automatic temperature control.
- Automatic water refill.
- Flow monitoring.
- Water-level protection.
- Dissolved oxygen monitoring.
- pH monitoring.
- Alarm management.
- Local HMI.
- Web interface.
- Historical data logging.
- Event logging.
- RTC-based timestamps.
- CSV data export.

## 28. Excluded From Version 1

The following features are outside the initial product scope:

- Automatic pH dosing.
- Automatic chemical dosing.
- Automatic dissolved oxygen dosing.
- Automated feeding.
- Cloud-dependent operation.
- Internet-dependent control.
- Camera monitoring.
- Multi-tank management.
- Cloud data synchronization.

The architecture should allow these features to be added in future versions.

## 29. Future Expansion

The system architecture should allow future support for:

- Additional temperature sensors.
- Additional DO sensors.
- Additional water-level sensors.
- Automatic pH dosing.
- Automatic water-quality dosing.
- Additional incubation tanks.
- Remote notifications.
- Cloud data synchronization.
- Camera monitoring.
- Feeding automation.
- Batch tracking.
- Incubation profiles.
- Multiple environmental profiles.

## 30. Product-Level Success Criteria

Version 1 shall be considered operational when the system can:

1. Monitor all required water parameters.
2. Display live data on the HMI.
3. Display live data through the web interface.
4. Maintain the configured water temperature.
5. Operate the heater safely.
6. Operate both Peltier modules safely.
7. Maintain continuous water circulation.
8. Maintain continuous aeration.
9. Detect low flow.
10. Detect low water.
11. Detect high water.
12. Automatically refill the main tank.
13. Stop refilling at the configured level.
14. Detect an empty refill reservoir.
15. Detect refill flow failure.
16. Monitor dissolved oxygen.
17. Monitor pH.
18. Generate alarms for abnormal conditions.
19. Record sensor data to microSD.
20. Timestamp records using the TinyRTC.
21. Record equipment state changes.
22. Display historical data.
23. Allow historical data to be exported as CSV.
24. Allow CSV export by a selected date and time range.
25. Produce CSV files compatible with common spreadsheet applications.
26. Recover safely after power interruption.
27. Operate continuously during extended testing.
28. Protect the system when critical sensors or equipment fail.