# 06 Implementation Plan

## 1. Purpose

This document defines the implementation plan for the Crayfish Incubator Version 1.

The implementation shall proceed incrementally.

Each subsystem shall be:

1. Designed.
2. Built.
3. Tested independently.
4. Integrated.
5. Safety tested.
6. Validated before moving to the next major subsystem.

The system shall not be operated with live crayfish until the complete control, monitoring, safety, thermal, water, and electrical systems have been validated.

---

# 2. Implementation Strategy

The project shall follow this sequence:

```text
Requirements
    ↓
Hardware Procurement
    ↓
Controller Bring-Up
    ↓
Sensor Bring-Up
    ↓
Actuator Bring-Up
    ↓
Safety System
    ↓
Temperature Control
    ↓
Peltier Cooling
    ↓
Automatic Refill
    ↓
Data Logging
    ↓
HMI
    ↓
Web Interface
    ↓
Full Integration
    ↓
Water Testing
    ↓
Thermal Testing
    ↓
24/7 Reliability Testing
    ↓
Biological Validation
    ↓
Production Operation
```

---

# 3. Development Phases

The project shall be divided into the following phases:

| Phase | Description |
|---|---|
| 0 | Requirements and design freeze |
| 1 | Hardware procurement |
| 2 | Controller and development environment |
| 3 | Basic hardware bring-up |
| 4 | Sensor integration |
| 5 | Actuator integration |
| 6 | Safety system |
| 7 | Temperature control |
| 8 | Peltier cooling |
| 9 | Automatic refill |
| 10 | RTC and data logging |
| 11 | HMI |
| 12 | Web interface |
| 13 | Full system integration |
| 14 | Dry testing |
| 15 | Water testing |
| 16 | Thermal testing |
| 17 | Long-duration reliability testing |
| 18 | Biological commissioning |
| 19 | Documentation and release |

---

# 4. Phase 0 - Requirements and Design Freeze

## Objective

Confirm the system requirements before purchasing and fabricating hardware.

## Tasks

Confirm:

- Main tank size.
- Exact crayfish species.
- Required temperature range.
- Required DO range.
- Required pH range.
- Target water level.
- Required circulation rate.
- Refill strategy.
- Target cooling performance.
- Required heating performance.
- Logging interval.
- HMI requirements.
- Web interface requirements.
- Safety requirements.

Biological operating limits shall remain configurable until the crayfish species is confirmed.

## Deliverables

- Approved Product Requirements.
- Approved Technical Requirements.
- Approved System Architecture.
- Approved Hardware Design.
- Approved Software Design.

## Exit Criteria

All major system requirements are defined.

---

# 5. Phase 1 - Hardware Procurement

## Objective

Acquire all components required for prototype construction.

## Controller

- ESP32-S3
- 7-inch touchscreen HMI
- TinyRTC
- microSD card

## Sensors

- 2 × DS18B20
- DO probe and transmitter
- pH probe and transmitter
- Main level sensor
- Low float
- High float
- Refill reservoir low-level sensor
- Main flow sensor
- Refill flow sensor
- Peltier heatsink temperature sensor
- Ambient temperature sensor

## Actuators

- Heater
- 2 × TEC1-12706
- Peltier water block
- Heatsink
- Cooling fans
- Circulation pump
- Air pump
- Air stones
- Refill solenoid

## Electrical

- Power supplies
- DC/DC converter if required
- Relay/SSR modules
- MOSFET drivers
- TEC drivers
- Fuses
- Breakers
- RCD/GFCI
- Emergency stop
- Enclosure
- Wiring
- Connectors
- Cable glands

## Exit Criteria

All required components are available or have confirmed alternatives.

---

# 6. Phase 2 - Development Environment

## Objective

Prepare the firmware development environment.

Recommended:

- VS Code
- PlatformIO
- ESP32-S3
- ESP-IDF or Arduino framework as finalized during implementation
- LVGL for HMI
- Git

Repository structure:

```text
crayfish-incubator/
│
├── docs/
│   ├── 01-product-requirement.md
│   ├── 02-technical-requirement.md
│   ├── 03-system-architecture.md
│   ├── 04-hardware-design.md
│   ├── 05-software-design.md
│   ├── 06-implementation-plan.md
│   └── README.md
│
├── firmware/
│
├── hardware/
│
├── test/
│
└── README.md
```

## Tasks

- Create Git repository.
- Configure PlatformIO.
- Configure target ESP32-S3 board.
- Create initial firmware.
- Verify serial communication.
- Configure compiler warnings.
- Establish coding conventions.
- Create development configuration.
- Create production configuration.

## Exit Criteria

ESP32-S3 can compile, flash, boot, and communicate reliably.

---

# 7. Phase 3 - Controller Bring-Up

## Objective

Verify the ESP32-S3 and supporting hardware.

## Tasks

Test:

- ESP32-S3 boot.
- GPIO.
- I2C.
- 1-Wire.
- RS485.
- SPI.
- SD card.
- Wi-Fi.
- Display.
- Touchscreen.
- Watchdog.

## Initial Firmware

Implement:

```text
Boot
 ↓
Hardware Initialization
 ↓
Serial Logging
 ↓
Basic Diagnostics
 ↓
System Ready
```

## Exit Criteria

All required communication buses operate correctly.

---

# 8. Phase 4 - Sensor Integration

Sensor integration shall proceed one subsystem at a time.

---

## 8.1 DS18B20

Tasks:

- Detect both sensors.
- Assign sensor identities.
- Read temperature.
- Validate readings.
- Detect disconnection.
- Detect invalid readings.
- Compare T1 and T2.

Acceptance:

```text
T1 valid
T2 valid
Difference within configured limit
```

---

## 8.2 TinyRTC

Tasks:

- Initialize RTC.
- Read date/time.
- Set date/time.
- Validate date/time.
- Test power-loss time retention.
- Test NTP synchronization.

Acceptance:

RTC provides valid timestamps after controller reboot.

---

## 8.3 DO

Tasks:

- Connect RS485.
- Configure Modbus address.
- Read DO.
- Validate response.
- Detect timeout.
- Detect CRC error.
- Detect disconnected device.

Acceptance:

Stable DO readings can be obtained continuously.

---

## 8.4 pH

Tasks:

- Connect RS485.
- Configure Modbus address.
- Read pH.
- Validate response.
- Detect timeout.
- Detect communication failure.
- Implement calibration workflow.

Acceptance:

Stable pH readings are obtained and calibration can be performed.

---

## 8.5 Main Water Level

Tasks:

- Install continuous level sensor.
- Read measurement.
- Calibrate tank level.
- Configure low threshold.
- Configure normal range.
- Configure high threshold.

Then integrate:

- Low float.
- High float.

Acceptance:

The system correctly distinguishes:

- Critical low.
- Low.
- Normal.
- High.
- Emergency high.

---

## 8.6 Refill Reservoir Level

Tasks:

- Install low-level sensor.
- Detect reservoir empty condition.
- Test sensor failure.
- Test refill lockout.

Acceptance:

The refill system cannot operate when the reservoir is empty.

---

## 8.7 Main Flow Sensor

Tasks:

- Install sensor.
- Measure pulses.
- Convert to flow rate.
- Calibrate against known flow.
- Configure minimum acceptable flow.

Acceptance:

The controller reliably detects:

- No flow.
- Low flow.
- Normal flow.

---

## 8.8 Refill Flow Sensor

Tasks:

- Install sensor.
- Measure refill flow.
- Configure expected flow.
- Configure timeout.

Acceptance:

The system can determine whether refill water is actually flowing.

---

# 9. Phase 5 - Actuator Integration

Actuators shall initially be tested individually.

No automatic control shall be enabled during initial actuator testing.

---

# 10. Circulation Pump

Test:

- OFF.
- ON.
- Continuous operation.
- Flow measurement.
- Current consumption.
- Plumbing leaks.
- Pump restart after power interruption.

Acceptance:

Pump can operate continuously without overheating or abnormal vibration.

---

# 11. Aeration System

Test:

- Air pump.
- Manifold.
- Air stones.
- Check valves.
- Air distribution.

Acceptance:

All air stones receive adequate airflow.

The system shall continue aeration independently of the main circulation system.

---

# 12. Heater

Test:

- Heater OFF.
- Heater ON.
- Relay/SSR control.
- Independent thermostat.
- Water-level protection.
- Flow protection.

Do not perform unattended heater testing before independent thermal protection has been verified.

Acceptance:

The heater cannot remain energized when safety conditions are violated.

---

# 13. Refill Solenoid

Test:

- Valve OFF.
- Valve ON.
- Power loss.
- Emergency stop.
- High-level protection.

Acceptance:

The normally closed solenoid closes when power is removed.

---

# 14. Peltier System

Test TEC modules separately.

## TEC 1

Test:

- Driver.
- Current.
- Voltage.
- Temperature response.
- Hot-side temperature.
- Fan operation.

## TEC 2

Repeat the same tests independently.

## Acceptance

Each TEC channel can be independently enabled and disabled.

Neither TEC can operate without required safety conditions.

---

# 15. Phase 6 - Safety System

Safety implementation shall occur before automatic operation.

## Safety Inputs

Implement:

- Emergency stop.
- Main low float.
- Main high float.
- Refill low level.
- Main flow.
- Peltier heatsink temperature.
- Temperature sensor validity.

---

# 16. Safety Matrix

| Condition | Heater | TEC1 | TEC2 | Refill | Pump | Aeration |
|---|---|---|---|---|---|---|
| Normal | Allowed | Allowed | Allowed | Allowed | ON | ON |
| Low water | OFF | OFF | OFF | Conditional | ON | ON |
| Emergency high | OFF | OFF | OFF | OFF | ON | ON |
| No flow | OFF | OFF | OFF | Normal | ON | ON |
| Peltier overtemp | Normal | OFF | OFF | Normal | ON | ON |
| E-stop | OFF | OFF | OFF | OFF | Safe state | Safe state |

The final pump and aeration response to emergency stop shall be confirmed during the safety review.

---

# 17. Safety Testing

Each safety condition shall be intentionally triggered.

Examples:

```text
Disconnect T1
Disconnect T2
Block circulation
Trigger low float
Trigger high float
Empty refill reservoir
Block refill line
Overheat Peltier heatsink
Press emergency stop
Disconnect flow sensor
Disconnect RS485 device
Remove SD card
Disconnect Wi-Fi
Restart controller
Remove power
```

Expected behavior shall be documented and verified.

---

# 18. Phase 7 - Temperature Control

Implement automatic temperature control after sensor and actuator testing.

Control flow:

```text
Read T1
Read T2
     ↓
Validate
     ↓
Compare With Setpoint
     ↓
Temperature Controller
     ↓
Safety Manager
     ↓
Equipment Manager
     ↓
Heater / Cooling
```

---

# 19. Heating Control

Use hysteresis initially.

Example concept:

```text
Temperature < Low Limit
        ↓
     HEATER ON

Temperature ≥ Setpoint
        ↓
     HEATER OFF
```

The exact values shall be configurable.

Do not finalize biological temperature values until the species requirement is confirmed.

---

# 20. Cooling Control

Initial strategy:

```text
Cooling Demand
     │
     ▼
TEC 1
     │
     ▼
Higher Demand?
   YES
     │
     ▼
TEC 1 + TEC 2
```

The final control algorithm shall be determined from thermal testing.

---

# 21. Heating and Cooling Interlock

The system shall prevent intentional simultaneous operation.

```text
HEATING
   ↓
Cooling OFF

COOLING
   ↓
Heating OFF
```

A configurable deadband shall prevent rapid switching.

---

# 22. Phase 8 - Peltier Cooling Development

This phase is critical because the cooling capacity of two TEC1-12706 modules must be experimentally verified.

## Step 1

Test TEC 1 alone.

Record:

- Water temperature.
- Ambient temperature.
- TEC current.
- TEC voltage.
- Hot-side temperature.
- Cooling rate.

## Step 2

Repeat using TEC 2.

## Step 3

Operate both TECs.

## Step 4

Perform continuous cooling test.

## Step 5

Perform test under high ambient temperature.

---

# 23. Thermal Test Procedure

Use a known water volume close to the actual operating volume.

Record:

- Starting water temperature.
- Ambient temperature.
- Water temperature every minute or configurable interval.
- TEC 1 state.
- TEC 2 state.
- TEC current.
- TEC voltage.
- Hot-side temperature.
- Fan state.
- Water flow.

Calculate:

- Cooling rate.
- Temperature stability.
- Heat rejection performance.
- Energy consumption.
- Maximum hot-side temperature.

---

# 24. Peltier Acceptance Criteria

The cooling system shall be considered acceptable only if it can maintain the required temperature range under realistic conditions.

Testing shall include:

- Normal room conditions.
- High room temperature.
- Continuous operation.
- 24-hour operation.
- Preferably 48 to 72 hours.

If two TEC1-12706 modules cannot maintain the required temperature, the cooling architecture shall be reviewed before biological operation.

---

# 25. Phase 9 - Automatic Refill

Implement the refill state machine.

```text
IDLE
 ↓
LOW LEVEL DETECTED
 ↓
CHECK RESERVOIR
 ↓
CHECK SAFETY
 ↓
OPEN SOLENOID
 ↓
VERIFY FLOW
 ↓
REFILL
 ↓
TARGET LEVEL
 ↓
CLOSE SOLENOID
 ↓
VERIFY
 ↓
COMPLETE
```

---

# 26. Refill Fault Testing

Test:

### Empty Reservoir

Expected:

```text
Reservoir LOW
↓
Solenoid OFF
↓
Alarm
↓
Refill Lockout
```

### Blocked Refill Line

Expected:

```text
Solenoid ON
↓
No Flow
↓
Timeout
↓
Solenoid OFF
↓
Alarm
```

### High Tank Level

Expected:

```text
HIGH FLOAT
↓
Solenoid OFF
```

### Solenoid Failure

Detect where possible through abnormal flow behavior.

---

# 27. Phase 10 - RTC and Data Logging

Implement persistent timestamps.

## Logging Interval

Initial target:

**1 minute**

The interval shall be configurable.

---

# 28. Sensor Data

Log:

```text
timestamp
water_temp_1
water_temp_2
do
ph
main_level
refill_level
main_flow
refill_flow
ambient_temperature
peltier_heatsink_temperature
heater
tec1
tec2
cooling_fans
circulation_pump
aeration
refill_valve
alarm
```

---

# 29. Event Logging

Record important events:

- System start.
- System shutdown.
- Heater ON/OFF.
- TEC ON/OFF.
- Refill start.
- Refill complete.
- Refill fault.
- Low water.
- High water.
- Low flow.
- Sensor fault.
- Alarm activation.
- Alarm clear.
- Emergency stop.
- SD failure.
- Wi-Fi failure.

---

# 30. SD Card Testing

Test:

- Normal write.
- File creation.
- Append.
- Power interruption.
- SD removal.
- SD reinsertion.
- Full storage condition.
- Corrupted file handling.

Acceptance:

SD failure does not stop automatic control.

---

# 31. CSV Export

Implement web-based CSV export.

Input:

```text
Start Date/Time
End Date/Time
```

Process:

```text
User Request
    ↓
Validate Range
    ↓
Find Log Files
    ↓
Read in Chunks
    ↓
Filter Records
    ↓
Stream CSV
```

The complete dataset shall not be loaded into RAM.

Acceptance:

Large log files can be exported without interrupting control tasks.

---

# 32. Phase 11 - HMI Development

Implement the HMI after core control functions are operational.

## Screens

1. Dashboard
2. Water Quality
3. Temperature
4. Equipment
5. Historical Data
6. Alarm History
7. Settings
8. Manual Control
9. System Information

---

# 33. Dashboard Implementation

Display:

- Water temperature.
- DO.
- pH.
- Main tank level.
- Refill reservoir level.
- Main flow.
- Refill flow.
- Heater state.
- TEC 1 state.
- TEC 2 state.
- Cooling fan state.
- Circulation pump state.
- Aeration state.
- Refill valve state.
- Alarm state.
- Operating mode.
- Date/time.

Critical alarms shall be clearly visible.

---

# 34. Manual Control

Manual controls shall include:

- Heater.
- TEC 1.
- TEC 2.
- Cooling fans.
- Circulation pump.
- Aeration pump.
- Refill solenoid.

Every manual command shall pass through the same safety system as automatic commands.

Manual mode must never bypass critical safety interlocks.

---

# 35. Historical Data

Display trends for:

- Water temperature.
- DO.
- pH.
- Water level.
- Flow.
- Peltier temperature.

The HMI shall request only the required time range.

It shall not load the entire history into memory.

---

# 36. Phase 12 - Web Interface

Implement the local Wi-Fi web interface.

Initial pages:

```text
/
Dashboard

/status
System status

/history
Historical data

/events
Events

/alarms
Alarm history

/settings
Configuration

/control
Manual control

/export
CSV export
```

---

# 37. Web API

Initial API:

```text
GET  /api/status
GET  /api/sensors
GET  /api/equipment
GET  /api/alarms
GET  /api/history
GET  /api/events
GET  /api/config

POST /api/control
POST /api/config
GET  /api/export
```

Exact API structure may be refined during implementation.

---

# 38. Network Failure Testing

Test:

1. Wi-Fi disconnected.
2. Router powered off.
3. ESP32 disconnected from network.
4. Network reconnect.
5. Web interface unavailable.

Expected:

```text
Wi-Fi OFF
    ↓
Automatic Control Continues
    ↓
HMI Continues
    ↓
Logging Continues
    ↓
RTC Continues
    ↓
Reconnect When Available
```

---

# 39. Phase 13 - Full System Integration

Combine:

- Sensors.
- Controllers.
- Safety.
- Pumps.
- Heater.
- Peltier.
- Refill.
- Aeration.
- RTC.
- SD.
- HMI.
- Web interface.

System architecture:

```text
                 ┌───────────────┐
                 │    ESP32-S3   │
                 └───────┬───────┘
                         │
        ┌────────────────┼────────────────┐
        │                │                │
        ▼                ▼                ▼
     Sensors          Safety            UI
        │                │                │
        └────────────────┼────────────────┘
                         │
                         ▼
                   Control Logic
                         │
          ┌──────────────┼──────────────┐
          ▼              ▼              ▼
       Heater         Cooling        Refill
          │              │              │
          └──────────────┼──────────────┘
                         │
                         ▼
                    Data Logger
                         │
                         ▼
                       SD
```

---

# 40. Phase 14 - Dry Testing

Dry testing shall be performed before adding water.

Test:

- Firmware boot.
- HMI.
- Buttons/touch.
- Sensor communication.
- Alarm logic.
- Relay outputs.
- Solenoid output.
- Emergency stop.
- Configuration.
- RTC.
- SD logging.
- Web interface.

Water-side equipment may be simulated where appropriate.

Acceptance:

All software functions operate without requiring live water.

---

# 41. Phase 15 - Water Testing

Fill the system with clean test water.

Do not introduce crayfish yet.

Test:

- Tank filling.
- Circulation.
- Flow.
- Aeration.
- Level monitoring.
- Refill.
- Heater.
- Cooling.
- Water temperature.
- DO.
- pH.
- Leaks.

---

# 42. Leak Testing

Perform a complete leak inspection.

Inspect:

- Tank fittings.
- Pump connections.
- Flow sensors.
- Water block.
- Tubing.
- Return connections.
- Refill line.
- Solenoid.
- Sensor penetrations.

Recommended:

Run the system for an extended period before unattended operation.

Acceptance:

No visible leaks.

---

# 43. Phase 16 - Thermal Testing

Thermal testing shall verify:

- Heating performance.
- Cooling performance.
- Temperature stability.
- Heater safety.
- Peltier safety.
- Hot-side heat rejection.
- Control hysteresis.

Test conditions:

```text
Low Ambient
Normal Ambient
High Ambient
```

Where practical, test during the hottest expected part of the day.

---

# 44. Heating Test

Measure:

- Starting water temperature.
- Ambient temperature.
- Heater power.
- Water temperature.
- Heating rate.
- Temperature overshoot.
- Time to target.

Verify:

- Heater shuts down at target.
- Heater cannot operate under low water.
- Heater cannot operate without sufficient circulation.

---

# 45. Cooling Test

Measure:

- Starting water temperature.
- Ambient temperature.
- TEC 1 current.
- TEC 2 current.
- Water temperature.
- Hot-side temperature.
- Fan operation.
- Flow rate.
- Cooling rate.

Verify:

- TEC 1 works independently.
- TEC 2 works independently.
- Both TECs work together.
- Hot-side temperature remains safe.
- Cooling stops safely.
- Fans provide adequate heat rejection.

---

# 46. Phase 17 - Long-Duration Reliability Testing

Before biological operation, operate the system continuously.

Minimum target:

**72 hours**

Preferred:

**7 days**

During testing, monitor:

- Water temperature.
- DO.
- pH.
- Flow.
- Level.
- Heater operation.
- TEC operation.
- Pump operation.
- Aeration.
- Refill.
- SD logging.
- RTC.
- Wi-Fi.
- HMI.
- Alarms.

---

# 47. Reliability Test Fault Injection

Intentionally introduce faults.

Examples:

### Sensor

- Disconnect DS18B20.
- Disconnect DO.
- Disconnect pH.
- Disconnect flow sensor.

### Water

- Reduce water level.
- Trigger high level.
- Stop circulation.
- Empty refill reservoir.

### Electrical

- Restart controller.
- Disconnect Wi-Fi.
- Remove SD card.
- Simulate power interruption.

### Cooling

- Reduce cooling airflow.
- Trigger heatsink overtemperature.

Expected responses shall be recorded.

---

# 48. Power Recovery Testing

Test:

```text
System Running
     ↓
Power Loss
     ↓
Power Restored
     ↓
ESP32 Boot
     ↓
Outputs Safe
     ↓
Sensors Initialize
     ↓
Safety Inputs Verified
     ↓
RTC Restored
     ↓
Automatic Control Resumes
```

Verify:

- Heater OFF during startup.
- TEC OFF during startup.
- Refill solenoid CLOSED during startup.
- Pumps enter defined safe states.
- Aeration enters defined safe state.
- Control resumes only after safety validation.

---

# 49. Phase 18 - Biological Commissioning

Biological commissioning begins only after successful technical testing.

Before adding crayfish:

- Confirm water quality.
- Confirm temperature stability.
- Confirm DO stability.
- Confirm pH stability.
- Confirm circulation.
- Confirm aeration.
- Confirm refill behavior.
- Confirm alarm behavior.

The operating profile shall use values appropriate for the confirmed crayfish species.

---

# 50. Initial Biological Operation

The first biological run shall be treated as a monitored commissioning period.

Monitor:

- Temperature.
- DO.
- pH.
- Water level.
- Flow.
- Animal behavior.
- Equipment operation.
- Alarms.

Avoid immediately enabling aggressive automatic control.

Allow sufficient observation time to identify unexpected interactions.

---

# 51. Configuration Management

The following parameters shall be configurable:

## Temperature

- Target temperature.
- Heating deadband.
- Cooling deadband.
- Maximum safe temperature.
- Minimum safe temperature.

## Water Level

- Low threshold.
- Refill target.
- High threshold.
- Emergency high threshold.

## Flow

- Minimum circulation flow.
- Expected refill flow.
- Refill timeout.

## Water Quality

- DO warning threshold.
- DO critical threshold.
- pH warning limits.
- pH critical limits.

Biological values shall be based on the confirmed species and experimental requirements.

---

# 52. Peltier Configuration

Configurable:

- TEC 1 enable.
- TEC 2 enable.
- Cooling demand threshold.
- Maximum hot-side temperature.
- Minimum water flow.
- Maximum runtime if required.
- Fan behavior.

TEC1 and TEC2 shall remain independently controllable.

---

# 53. Refill Configuration

Configurable:

- Refill enable.
- Low-level threshold.
- Target level.
- Maximum refill duration.
- Minimum refill flow.
- Reservoir low-level lockout.
- Refill cooldown if required.

---

# 54. Alarm Configuration

Each alarm shall have:

- Enable/disable where appropriate.
- Warning threshold.
- Critical threshold.
- Delay.
- Recovery condition.
- Logging behavior.

Critical safety alarms shall not be disableable by normal users.

---

# 55. Software Release Stages

Firmware shall use release stages.

```text
DEV
 ↓
ALPHA
 ↓
BETA
 ↓
RC
 ↓
V1.0
```

## DEV

Used for active development.

## ALPHA

Major functions implemented.

## BETA

Full system operating with hardware.

## RC

Safety and reliability testing complete.

## V1.0

Approved for biological operation.

---

# 56. Git Development Strategy

Recommended branches:

```text
main
develop
feature/*
fix/*
hardware/*
```

Example:

```text
feature/temperature-control
feature/peltier-control
feature/refill
feature/rtc
feature/data-logging
feature/hmi
feature/web-interface
```

Merge into `develop` after testing.

Release into `main` after system validation.

---

# 57. Test Documentation

Each subsystem shall have a test record.

Suggested:

```text
test/
├── controller/
├── sensors/
├── actuators/
├── safety/
├── temperature/
├── cooling/
├── refill/
├── logging/
├── hmi/
├── web/
└── system/
```

Each test should include:

- Test ID.
- Date.
- Hardware version.
- Firmware version.
- Test procedure.
- Expected result.
- Actual result.
- Pass/fail.
- Notes.

---

# 58. Fault Matrix

| Fault | Detection | Immediate Response | Alarm |
|---|---|---|---|
| T1 failure | Sensor validation | Heating/cooling affected based on T2 strategy | Yes |
| T2 failure | Sensor validation | Continue only if configured safe | Yes |
| Both temperature sensors fail | Sensor validation | Heater OFF, cooling OFF | Critical |
| Low water | Float/level | Heater OFF, cooling OFF | Critical |
| High water | Float/level | Refill OFF | Critical |
| No circulation | Flow sensor | Heater OFF, cooling OFF | Critical |
| DO communication failure | Modbus timeout | Alarm, configurable response | Yes |
| pH communication failure | Modbus timeout | Alarm, configurable response | Yes |
| Refill reservoir empty | Level sensor | Refill OFF | Yes |
| Refill flow missing | Flow sensor | Solenoid OFF | Critical |
| Peltier overtemp | Heatsink sensor | TECs OFF, fans ON | Critical |
| SD failure | Storage error | Continue control | Warning |
| Wi-Fi failure | Network status | Continue control | Warning |
| Emergency stop | Safety input | Critical outputs OFF | Critical |

The exact response to individual sensor failures shall be finalized during commissioning.

---

# 59. Performance Measurements

The implementation shall record key performance metrics.

## Temperature

- Heating rate °C/hour.
- Cooling rate °C/hour.
- Temperature stability.
- Overshoot.
- Recovery time.

## Pump

- Actual flow.
- Current consumption.
- Continuous operating temperature.

## Peltier

- TEC current.
- TEC voltage.
- Cooling rate.
- Hot-side temperature.
- Energy consumption.

## Refill

- Refill flow.
- Refill duration.
- Volume added.
- Refill failure rate.

## System

- CPU usage.
- Memory usage.
- SD write performance.
- Wi-Fi stability.
- Reboot count.
- Alarm count.

---

# 60. Acceptance Test

The complete system shall pass the following:

### Monitoring

- [ ] Two water temperature sensors operate.
- [ ] DO measurement operates.
- [ ] pH measurement operates.
- [ ] Main water level operates.
- [ ] Refill level operates.
- [ ] Main flow operates.
- [ ] Refill flow operates.
- [ ] Peltier temperature monitoring operates.
- [ ] Ambient temperature operates.

### Control

- [ ] Heater operates.
- [ ] TEC 1 operates.
- [ ] TEC 2 operates.
- [ ] Cooling fans operate.
- [ ] Circulation pump operates.
- [ ] Aeration operates.
- [ ] Refill solenoid operates.

### Safety

- [ ] Low-water protection works.
- [ ] High-water protection works.
- [ ] No-flow protection works.
- [ ] Peltier thermal protection works.
- [ ] Emergency stop works.
- [ ] Heater independent thermostat works.
- [ ] Solenoid fails closed.
- [ ] Startup is safe.
- [ ] Power recovery is safe.

### Data

- [ ] TinyRTC operates.
- [ ] Timestamps remain valid after reboot.
- [ ] NTP synchronization works when available.
- [ ] Sensor logging works.
- [ ] Event logging works.
- [ ] Alarm logging works.
- [ ] CSV export works.
- [ ] SD failure does not stop control.

### Interface

- [ ] HMI operates.
- [ ] HMI displays real-time values.
- [ ] HMI displays alarms.
- [ ] HMI manual control works.
- [ ] Web dashboard works.
- [ ] Web control works.
- [ ] Web interface does not bypass safety.

### Reliability

- [ ] 24-hour test passed.
- [ ] 72-hour test passed.
- [ ] Fault injection passed.
- [ ] Power recovery passed.
- [ ] Wi-Fi failure passed.
- [ ] SD failure passed.

---

# 61. Commissioning Checklist

Before biological operation:

```text
[ ] Tank installed
[ ] Plumbing complete
[ ] Electrical enclosure complete
[ ] RCD/GFCI tested
[ ] Grounding verified
[ ] Fuses verified
[ ] Emergency stop verified
[ ] Pump verified
[ ] Aeration verified
[ ] Heater verified
[ ] TEC 1 verified
[ ] TEC 2 verified
[ ] Peltier fans verified
[ ] Water block verified
[ ] No leaks
[ ] Temperature sensors verified
[ ] DO verified
[ ] pH verified
[ ] Level sensors verified
[ ] Flow sensors verified
[ ] Refill verified
[ ] TinyRTC verified
[ ] SD logging verified
[ ] CSV export verified
[ ] HMI verified
[ ] Web interface verified
[ ] Alarm system verified
[ ] Power recovery verified
[ ] 72-hour test passed
[ ] Thermal test passed
[ ] Water quality verified
[ ] Species-specific operating profile configured
```

---

# 62. Implementation Milestones

## M1 - Controller Operational

ESP32-S3 boots and communicates with required peripherals.

## M2 - Sensor Platform Operational

All major sensors provide validated readings.

## M3 - Actuator Platform Operational

All pumps, heater, TECs, fans, and solenoid operate independently.

## M4 - Safety Operational

Critical interlocks are validated.

## M5 - Temperature Control Operational

Automatic heating and cooling work.

## M6 - Refill Operational

Automatic refill operates safely.

## M7 - Logging Operational

RTC, SD logging, events, alarms, and CSV export work.

## M8 - HMI Operational

Local touchscreen interface complete.

## M9 - Web Interface Operational

Local web interface complete.

## M10 - Integrated System Operational

All subsystems operate together.

## M11 - Reliability Validation Complete

24 to 72+ hour testing passes.

## M12 - Biological Commissioning Complete

System is approved for normal operation.

---

# 63. Recommended Development Order

The practical development order shall be:

```text
1. ESP32-S3
2. HMI
3. TinyRTC
4. microSD
5. DS18B20
6. RS485
7. DO
8. pH
9. Level sensors
10. Flow sensors
11. Circulation pump
12. Aeration
13. Refill
14. Heater
15. Safety system
16. TEC 1
17. TEC 2
18. Peltier fans
19. Temperature controller
20. Refill controller
21. Data logger
22. HMI
23. Web interface
24. Full integration
25. Thermal testing
26. Reliability testing
27. Biological commissioning
```

This order minimizes the risk of debugging multiple unknown subsystems simultaneously.

---

# 64. Definition of Done

The Crayfish Incubator Version 1 implementation is complete when:

1. Hardware is assembled.
2. Firmware is stable.
3. All required sensors are operational.
4. All required actuators are operational.
5. Safety interlocks are verified.
6. Temperature control is stable.
7. Peltier cooling has been thermally validated.
8. Automatic refill operates reliably.
9. Aeration operates continuously.
10. TinyRTC provides persistent time.
11. Data is logged to microSD.
12. Events and alarms are logged.
13. CSV export works.
14. HMI is operational.
15. Web interface is operational.
16. Wi-Fi failure does not stop the system.
17. SD failure does not stop the system.
18. Power recovery is safe.
19. Fault injection tests pass.
20. At least 72 hours of continuous system testing passes.
21. No unacceptable leaks are present.
22. Electrical safety verification is complete.
23. Species-specific operating parameters are configured.
24. Biological commissioning is complete.
25. Version 1 firmware is tagged and documented.

---

# 65. Final Project Flow

The complete implementation lifecycle is:

```text
                 REQUIREMENTS
                      │
                      ▼
              SYSTEM ARCHITECTURE
                      │
                      ▼
               HARDWARE DESIGN
                      │
                      ▼
               SOFTWARE DESIGN
                      │
                      ▼
              HARDWARE PROCUREMENT
                      │
                      ▼
              CONTROLLER BRING-UP
                      │
                      ▼
              SENSOR INTEGRATION
                      │
                      ▼
             ACTUATOR INTEGRATION
                      │
                      ▼
                SAFETY SYSTEM
                      │
                      ▼
            TEMPERATURE CONTROL
                      │
                      ▼
              PELTIER VALIDATION
                      │
                      ▼
               AUTO REFILL
                      │
                      ▼
              DATA + LOGGING
                      │
                      ▼
                   HMI
                      │
                      ▼
              WEB INTERFACE
                      │
                      ▼
             FULL INTEGRATION
                      │
                      ▼
                WATER TEST
                      │
                      ▼
               THERMAL TEST
                      │
                      ▼
             RELIABILITY TEST
                      │
                      ▼
          BIOLOGICAL COMMISSIONING
                      │
                      ▼
                  VERSION 1
```

# 66. Final Implementation Principle

The system shall be developed as a **safety-first control system**, not simply as an ESP32 automation project.

The ESP32 shall provide intelligent monitoring and control, while independent hardware protections shall prevent dangerous conditions.

The most important validation areas are:

1. Water level protection.
2. Circulation flow protection.
3. Heater protection.
4. Peltier thermal protection.
5. Refill protection.
6. Electrical safety.
7. Cooling capacity.
8. Long-duration reliability.
9. Data integrity.
10. Safe recovery after faults and power interruption.

The system shall only transition from prototype testing to biological operation after these areas have been validated.