# 05 Software Design

## 1. Purpose

This document defines the software architecture and firmware design for the Crayfish Incubator Version 1.

The software shall run on the ESP32-S3 and shall provide:

- Sensor acquisition.
- Sensor validation.
- Temperature control.
- Heating control.
- Peltier cooling control.
- Water circulation monitoring.
- Aeration control.
- Automatic water refill.
- Water-level monitoring.
- Dissolved oxygen monitoring.
- pH monitoring.
- Alarm management.
- TinyRTC time management.
- Data logging.
- Event logging.
- Local touchscreen HMI.
- Wi-Fi web interface.
- CSV export.
- Configuration management.
- Manual control.
- Safety interlocks.
- System recovery.

---

# 2. Software Design Principles

The firmware shall follow these principles:

1. Safety has the highest priority.
2. Control logic shall be independent from the user interface.
3. Sensor drivers shall be independent from application logic.
4. HMI and web controls shall use the same command system.
5. Critical control shall not depend on Wi-Fi.
6. Data logging shall not block control operations.
7. CSV export shall not block critical control operations.
8. Sensor failures shall be detected and handled.
9. Equipment shall fail toward a safe state.
10. Configuration shall be separated from control logic.
11. Hardware drivers shall be isolated from application logic.
12. Fast control timing shall use ESP32 timers or task timing, not the RTC.
13. TinyRTC shall provide persistent wall-clock time.
14. All biological operating limits shall remain configurable.

---

# 3. Software Architecture

The firmware shall use a layered architecture.

```text
┌───────────────────────────────────────────────┐
│                USER INTERFACE                 │
│                                               │
│             HMI + Web Interface               │
├───────────────────────────────────────────────┤
│              APPLICATION LAYER                │
│                                               │
│ Temperature │ Refill │ Alarm │ Operating     │
│ Control     │ Control│ Mgmt  │ Modes         │
├───────────────────────────────────────────────┤
│                SERVICE LAYER                  │
│                                               │
│ Logging │ RTC │ Config │ CSV │ Event Manager │
├───────────────────────────────────────────────┤
│                 DEVICE LAYER                  │
│                                               │
│ Sensors │ GPIO │ RS485 │ I2C │ 1-Wire │ SD   │
├───────────────────────────────────────────────┤
│                HARDWARE LAYER                 │
│                                               │
│                    ESP32-S3                   │
└───────────────────────────────────────────────┘
```

---

# 4. Recommended Firmware Structure

The firmware shall be organized into separate modules.

Suggested structure:

```text
src/
│
├── main.cpp
│
├── config/
│   ├── system_config.h
│   ├── pin_config.h
│   └── default_config.h
│
├── core/
│   ├── system_manager.cpp
│   ├── system_manager.h
│   ├── state_manager.cpp
│   ├── state_manager.h
│   ├── safety_manager.cpp
│   └── safety_manager.h
│
├── sensors/
│   ├── temperature_manager.cpp
│   ├── temperature_manager.h
│   ├── do_manager.cpp
│   ├── do_manager.h
│   ├── ph_manager.cpp
│   ├── ph_manager.h
│   ├── level_manager.cpp
│   ├── level_manager.h
│   ├── flow_manager.cpp
│   └── flow_manager.h
│
├── control/
│   ├── temperature_controller.cpp
│   ├── temperature_controller.h
│   ├── cooling_controller.cpp
│   ├── cooling_controller.h
│   ├── heater_controller.cpp
│   ├── heater_controller.h
│   ├── refill_controller.cpp
│   ├── refill_controller.h
│   └── equipment_manager.cpp
│
├── safety/
│   ├── alarm_manager.cpp
│   ├── alarm_manager.h
│   ├── interlock_manager.cpp
│   └── interlock_manager.h
│
├── hardware/
│   ├── rtc_manager.cpp
│   ├── rtc_manager.h
│   ├── sd_manager.cpp
│   ├── sd_manager.h
│   ├── rs485_manager.cpp
│   ├── rs485_manager.h
│   └── gpio_manager.cpp
│
├── logging/
│   ├── data_logger.cpp
│   ├── data_logger.h
│   ├── event_logger.cpp
│   └── event_logger.h
│
├── web/
│   ├── web_server.cpp
│   ├── web_server.h
│   ├── api.cpp
│   ├── api.h
│   ├── csv_export.cpp
│   └── csv_export.h
│
├── hmi/
│   ├── hmi_manager.cpp
│   ├── hmi_manager.h
│   └── screens/
│
└── utils/
    ├── time_utils.cpp
    ├── validation.cpp
    └── helpers.cpp
```

The exact folder structure may be adjusted during implementation.

---

# 5. Main Program

`main.cpp` shall remain small.

Its responsibilities shall be limited to:

- System initialization.
- Starting required services.
- Starting control tasks.
- Starting communication services.

Conceptually:

```cpp
void setup()
{
    systemManager.begin();
}

void loop()
{
    systemManager.update();
}
```

Long-running functionality shall be implemented through dedicated managers and tasks.

---

# 6. System Manager

The System Manager shall coordinate the overall firmware.

Responsibilities:

- Startup sequence.
- System state.
- Operating mode.
- Safety state.
- Service initialization.
- Task management.
- Shutdown handling.
- Recovery handling.

Conceptual flow:

```text
System Manager
      │
      ├── Sensor Services
      ├── Control Services
      ├── Safety Services
      ├── Logging Services
      ├── HMI
      └── Web Server
```

---

# 7. System State

The system shall maintain a centralized system state.

Example:

```cpp
struct SystemState
{
    OperatingMode mode;

    bool systemReady;
    bool emergencyStop;
    bool safetyFault;

    TemperatureState temperature;
    WaterQualityState waterQuality;
    WaterLevelState waterLevel;
    FlowState flow;
    RefillState refill;

    EquipmentState equipment;

    AlarmState alarms;

    RTCState rtc;
    StorageState storage;
    NetworkState network;
};
```

The actual structures shall be refined during implementation.

---

# 8. Operating Modes

The firmware shall support:

```text
AUTOMATIC
MANUAL
MAINTENANCE
ALARM
```

The Operating Mode Manager shall determine which control functions are active.

---

# 9. Automatic Mode

Automatic mode shall control:

- Temperature.
- Heater.
- Peltier cooling.
- Circulation monitoring.
- Aeration.
- Water refill.
- Safety.
- Alarms.

Automatic control shall run without requiring a web client or HMI connection.

---

# 10. Manual Mode

Manual mode shall allow the operator to control supported equipment.

Supported controls:

- Heater.
- TEC 1.
- TEC 2.
- Cooling fans.
- Circulation pump.
- Aeration pump.
- Refill solenoid.

Every command shall pass through safety validation.

Example:

```text
Manual Heater ON
       ↓
Safety Check
       ↓
Water Level OK?
       ↓
Flow OK?
       ↓
Temperature Valid?
       ↓
Allow Heater
```

---

# 11. Maintenance Mode

Maintenance mode shall allow controlled testing and servicing.

Examples:

- Sensor testing.
- Pump testing.
- Heater testing.
- Peltier testing.
- Solenoid testing.
- Flow sensor testing.

Critical safety conditions shall remain active.

---

# 12. Alarm Mode

Alarm mode shall be entered when a critical fault requires the system to place affected equipment into a safe state.

Examples:

- Emergency high water.
- No circulation.
- Peltier over-temperature.
- Critical temperature sensor failure.
- Emergency stop.

Alarm mode shall not necessarily stop all equipment.

Each fault shall define its own safe state.

---

# 13. Sensor Architecture

All sensor modules shall follow a common pattern:

```text
Initialize
    ↓
Read
    ↓
Validate
    ↓
Update State
    ↓
Report Error
```

Each sensor manager shall provide:

- Current value.
- Previous value.
- Sensor status.
- Timestamp.
- Communication status.
- Error state.

---

# 14. Temperature Manager

The Temperature Manager shall handle both DS18B20 sensors.

```text
DS18B20 T1
DS18B20 T2
     │
     ▼
Temperature Manager
     │
     ├── Validate
     ├── Compare
     ├── Filter
     └── Update State
```

The manager shall detect:

- Disconnection.
- Invalid values.
- Out-of-range values.
- Sensor disagreement.

The manager shall expose:

```text
waterTemp1
waterTemp2
temperatureValid
temperatureDifference
sensorFault
```

---

# 15. Temperature Controller

The Temperature Controller shall determine whether heating or cooling is required.

Conceptual logic:

```text
Temperature
     │
     ▼
Compare With Setpoint
     │
 ┌───┼────┐
 │   │    │
Low Normal High
 │    │    │
 ▼    ▼    ▼
Heat None Cool
```

The controller shall use hysteresis.

Heating and cooling shall never intentionally operate simultaneously.

---

# 16. Heater Controller

The Heater Controller shall manage the heater output.

Inputs:

- Water temperature.
- Water level.
- Flow state.
- Sensor status.
- Operating mode.
- Safety state.

Outputs:

```text
HEATER_ON
HEATER_OFF
```

The heater shall automatically turn OFF when a critical safety condition occurs.

---

# 17. Cooling Controller

The Cooling Controller shall manage:

- TEC 1.
- TEC 2.
- Cooling fans.

Inputs:

- Water temperature.
- Water level.
- Flow.
- Hot-side temperature.
- Operating mode.
- Safety state.

Outputs:

```text
TEC1
TEC2
COOLING_FANS
```

---

# 18. Peltier Control Strategy

The initial strategy shall use staged cooling.

```text
Cooling Demand
       │
       ▼
Small Demand
       │
       ▼
TEC 1
```

If cooling demand increases:

```text
Higher Demand
       │
       ▼
TEC 1 + TEC 2
```

As the water approaches the target temperature, the controller shall reduce cooling demand where practical.

The exact control algorithm shall be tuned during thermal testing.

---

# 19. Peltier Safety Logic

Before enabling a TEC, the Cooling Controller shall verify:

```text
Temperature Sensor Valid
Water Level Safe
Circulation Flow Safe
Hot-Side Temperature Safe
Emergency Stop Inactive
No Critical Fault
```

If any critical condition fails:

```text
TEC 1 OFF
TEC 2 OFF
```

The cooling fans may remain ON.

---

# 20. Water Level Manager

The Water Level Manager shall combine:

- Continuous level measurement.
- Low float.
- High float.

It shall generate a normalized state:

```text
CRITICAL_LOW
LOW
NORMAL
HIGH
EMERGENCY_HIGH
SENSOR_FAULT
```

The continuous sensor shall provide measurement.

The safety floats shall provide independent protection.

---

# 21. Flow Manager

The Flow Manager shall monitor:

- Main circulation flow.
- Refill flow.

The manager shall calculate:

```text
Flow Rate
```

and determine:

```text
NO_FLOW
LOW_FLOW
NORMAL_FLOW
HIGH_FLOW
SENSOR_FAULT
```

Flow validation shall include configurable delays to prevent false alarms from short transients.

---

# 22. Dissolved Oxygen Manager

The DO Manager shall communicate with the DO transmitter through RS485 Modbus RTU.

Responsibilities:

- Poll sensor.
- Parse Modbus response.
- Validate value.
- Detect timeout.
- Detect communication errors.
- Update DO state.
- Trigger alarm conditions.

Example:

```text
RS485 Request
      ↓
DO Transmitter
      ↓
RS485 Response
      ↓
Validate
      ↓
DO State
```

---

# 23. pH Manager

The pH Manager shall communicate with the pH transmitter through RS485 Modbus RTU.

Responsibilities:

- Poll sensor.
- Parse response.
- Validate value.
- Detect timeout.
- Detect communication errors.
- Update pH state.
- Trigger alarm conditions.

---

# 24. RS485 Manager

The RS485 Manager shall provide a common communication service.

Responsibilities:

- UART initialization.
- RS485 direction control where required.
- Modbus request handling.
- Response timeout.
- Retry handling.
- CRC validation.
- Device addressing.
- Bus error detection.

The application layer shall not directly manipulate the RS485 hardware.

---

# 25. Refill Controller

The Refill Controller shall use a state machine.

States:

```text
IDLE
LOW_LEVEL_DETECTED
CHECKING_RESERVOIR
OPENING_VALVE
REFILLING
VERIFYING_FLOW
TARGET_REACHED
COMPLETE
FAULT
LOCKOUT
```

Example:

```text
IDLE
  ↓
LOW_LEVEL_DETECTED
  ↓
CHECKING_RESERVOIR
  ↓
OPENING_VALVE
  ↓
VERIFYING_FLOW
  ↓
REFILLING
  ↓
TARGET_REACHED
  ↓
COMPLETE
  ↓
IDLE
```

---

# 26. Refill Fault Handling

The refill controller shall detect:

### Empty Reservoir

```text
Reservoir LOW
    ↓
Solenoid OFF
    ↓
Alarm
```

### No Refill Flow

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

### High Level

```text
High Float
    ↓
Solenoid OFF
    ↓
Alarm
```

### Refill Timeout

```text
Maximum Refill Time
        ↓
Solenoid OFF
        ↓
Alarm
```

---

# 27. Aeration Controller

The Aeration Controller shall control the air pump.

Normal automatic operation:

```text
Automatic Mode
      ↓
Aeration ON
```

The controller shall keep aeration independent from circulation.

The system shall not automatically increase aeration based on DO in Version 1 unless variable aeration hardware is added later.

---

# 28. Equipment Manager

The Equipment Manager shall be the single software interface for equipment outputs.

Equipment:

```text
Heater
TEC 1
TEC 2
Cooling Fans
Circulation Pump
Aeration Pump
Refill Solenoid
```

Application modules shall request equipment states from the Equipment Manager.

The Equipment Manager shall apply safety restrictions before changing outputs.

---

# 29. Safety Manager

The Safety Manager shall evaluate critical safety conditions.

Inputs:

- Low water.
- High water.
- Emergency high water.
- Low flow.
- Sensor failures.
- Peltier heatsink temperature.
- Emergency stop.
- System faults.

Outputs:

```text
heaterAllowed
coolingAllowed
refillAllowed
pumpAllowed
manualControlAllowed
```

---

# 30. Interlock Priority

Safety interlocks shall have higher priority than automatic commands.

Priority:

```text
1. Emergency Safety
2. Hardware Protection
3. Critical Interlock
4. Automatic Control
5. Manual Control
6. User Interface Request
```

A lower-priority request shall never override a higher-priority safety condition.

---

# 31. Alarm Manager

The Alarm Manager shall provide centralized alarm handling.

Each alarm shall contain:

```cpp
struct Alarm
{
    uint16_t id;
    AlarmSeverity severity;
    AlarmState state;
    DateTime timestamp;
    float value;
};
```

The final structure shall be refined during implementation.

---

# 32. Alarm Severity

The system shall use at least three severity levels.

### INFO

Informational events.

### WARNING

Conditions requiring operator attention but not immediate shutdown.

### CRITICAL

Conditions requiring immediate equipment shutdown or safe-state action.

---

# 33. Alarm Lifecycle

Alarm lifecycle:

```text
NORMAL
   ↓
TRIGGERED
   ↓
ACTIVE
   ↓
ACKNOWLEDGED
   ↓
CLEARED
```

Acknowledging an alarm shall not automatically clear the underlying fault.

The fault must return to a normal state before the alarm can be cleared.

---

# 34. TinyRTC Manager

The RTC Manager shall abstract the TinyRTC hardware.

Responsibilities:

- Initialize RTC.
- Read date/time.
- Validate date/time.
- Set date/time.
- Synchronize from NTP.
- Detect RTC communication failure.
- Provide timestamps to other services.

Example API concept:

```text
rtc.begin()
rtc.getDateTime()
rtc.setDateTime()
rtc.syncFromNtp()
rtc.isValid()
```

---

# 35. Time Synchronization

Startup time strategy:

```text
ESP32-S3 Boot
      ↓
Read TinyRTC
      ↓
RTC Valid?
   ┌──┴──┐
  YES    NO
   │      │
   │    Wait for
   │    network time
   │      │
   └──┬───┘
      ▼
Normal Operation
```

If Wi-Fi and NTP are available, the system may synchronize the RTC.

The RTC shall then provide time during normal operation.

---

# 36. Configuration Manager

The Configuration Manager shall provide centralized access to system settings.

Configuration categories:

```text
Temperature
Water Level
Flow
DO
pH
Peltier
Refill
Alarm
Logging
Network
System
```

Configuration shall be validated before being applied.

Invalid configuration values shall be rejected.

---

# 37. Configuration Storage

Configuration shall be stored in non-volatile storage.

The system shall maintain default values in firmware.

Example:

```text
Default Configuration
        ↓
Non-Volatile Storage
        ↓
Runtime Configuration
```

Critical configuration should not depend solely on the microSD card.

---

# 38. Data Logger

The Data Logger shall create periodic records.

Initial logging interval:

```text
1 minute
```

Example record:

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
ambient_temp
peltier_heatsink_temp
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

# 39. Event Logger

The Event Logger shall record system events separately from periodic sensor records.

Examples:

```text
SYSTEM_START
SYSTEM_STOP
REFILL_START
REFILL_COMPLETE
REFILL_FAULT
HEATER_ON
HEATER_OFF
TEC1_ON
TEC1_OFF
TEC2_ON
TEC2_OFF
LOW_FLOW
HIGH_WATER
LOW_WATER
ALARM_ACTIVE
ALARM_CLEARED
EMERGENCY_STOP
```

This makes troubleshooting easier than relying only on periodic sensor logs.

---

# 40. SD Card Manager

The SD Manager shall provide:

- Card initialization.
- File creation.
- File reading.
- File writing.
- File append.
- File existence checks.
- Storage error detection.
- File rotation.

The SD Manager shall prevent file operations from blocking safety-critical control for excessive periods.

---

# 41. Data File Structure

The initial storage architecture shall use separate sensor and event files.

Suggested:

```text
/data/
    sensor/
        2026-10-09.csv

/events/
    2026-10-09.csv

/alarms/
    2026-10-09.csv

/config/
    system.json
```

The final file structure may be adjusted during implementation.

---

# 42. CSV Export Service

The CSV Export Service shall handle web requests for historical data.

Input:

```text
start_datetime
end_datetime
```

Process:

```text
HTTP Request
     ↓
Validate Date Range
     ↓
Find Relevant Files
     ↓
Read Records
     ↓
Filter Date Range
     ↓
Generate CSV
     ↓
Stream Response
```

The service shall avoid loading the entire dataset into memory.

---

# 43. CSV Export Safety

CSV export shall run at low priority compared with safety and control tasks.

If storage access becomes slow:

```text
Control Tasks
     ↓
Continue Normally

CSV Export
     ↓
Process in Small Chunks
```

The export operation shall never prevent:

- Safety monitoring.
- Temperature control.
- Refill protection.
- Alarm processing.

---

# 44. Web Server

The Web Server shall provide the local user interface.

Major endpoints may include:

```text
/
 /api/status
 /api/sensors
 /api/equipment
 /api/alarms
 /api/config
 /api/history
 /api/events
 /api/control
 /api/export
```

The exact endpoint structure shall be finalized during implementation.

---

# 45. Web API Design

The web interface shall communicate with the firmware through structured API responses.

Example status response:

```json
{
    "temperature": {
        "t1": 26.4,
        "t2": 26.5
    },
    "do": 6.8,
    "ph": 7.2,
    "level": 92,
    "flow": 8.4,
    "heater": false,
    "tec1": true,
    "tec2": false,
    "pump": true,
    "aeration": true,
    "refill": false
}
```

The final API schema shall be defined during implementation.

---

# 46. Web Control API

Manual control requests shall use a common command handler.

Example:

```text
POST /api/control
```

Possible commands:

```text
heater
tec1
tec2
cooling_fans
circulation_pump
aeration
refill
```

Every request shall pass through:

```text
API
 ↓
Command Validation
 ↓
Operating Mode Check
 ↓
Safety Check
 ↓
Equipment Manager
```

---

# 47. HMI Manager

The HMI Manager shall provide:

- Screen navigation.
- Sensor display.
- Equipment status.
- Alarm display.
- Settings.
- Manual controls.
- Historical information.

The HMI shall not contain control logic.

It shall send commands to the application layer.

---

# 48. HMI Screens

The initial HMI shall contain:

```text
1. Dashboard
2. Water Quality
3. Temperature
4. Equipment Status
5. Historical Data
6. Alarm History
7. Settings
8. Manual Control
9. System Information
```

---

# 49. Dashboard

The dashboard shall display:

```text
Water Temperature
Dissolved Oxygen
pH
Main Tank Level
Refill Reservoir Level
Main Flow
Refill Flow
Heater
TEC 1
TEC 2
Cooling Fans
Circulation Pump
Aeration
Refill Valve
Alarm
System Mode
Date / Time
```

Critical alarms shall be visually prominent.

---

# 50. Historical Data

The HMI historical screen shall provide access to locally stored data.

The HMI may display:

- Temperature trend.
- DO trend.
- pH trend.
- Water-level trend.
- Flow trend.
- Peltier temperature trend.

The HMI shall not need to load the complete dataset into memory.

---

# 51. Alarm History

The alarm history shall display:

- Timestamp.
- Alarm name.
- Severity.
- Trigger value.
- Current state.
- Clear time where available.

---

# 52. System Task Architecture

The ESP32-S3 firmware may use FreeRTOS tasks.

Recommended logical tasks:

```text
Sensor Task
Control Task
Safety Task
Logging Task
Alarm Task
HMI Task
Web Task
Communication Task
```

The final task allocation shall be optimized during implementation.

Not every module requires its own FreeRTOS task.

---

# 53. Task Priority

Safety and control shall have higher priority than user-interface and export tasks.

Conceptual priority:

```text
Highest
│
├── Safety
├── Control
├── Sensor Acquisition
├── Alarm
├── Communication
├── Logging
├── HMI
└── Web / CSV Export
│
Lowest
```

The exact FreeRTOS priorities shall be determined during implementation and testing.

---

# 54. Non-Blocking Design

The firmware shall avoid long blocking operations.

The following shall not block critical control:

- SD operations.
- CSV export.
- Web requests.
- HMI rendering.
- RS485 retries.
- Wi-Fi operations.

Timeouts shall be used for external devices.

---

# 55. Watchdog Strategy

The firmware shall use watchdog protection.

The system shall detect tasks that become stuck.

The watchdog shall not be used as the primary safety mechanism.

Before a watchdog reset, the system should place controllable outputs into a safe state where practical.

After reboot, the system shall execute the safe startup sequence.

---

# 56. Sensor Sampling Strategy

Initial target sampling behavior:

| Sensor/System | Suggested Sampling |
|---|---:|
| DS18B20 | 1 to 5 seconds |
| Water level | 1 to 5 seconds |
| Main flow | Continuous/high frequency |
| Refill flow | Continuous during refill |
| DO | 2 to 10 seconds |
| pH | 2 to 10 seconds |
| Peltier heatsink | 1 to 2 seconds |
| Ambient temperature | 5 to 30 seconds |
| Permanent logging | 60 seconds |

These are initial firmware targets.

Actual rates shall be adjusted according to sensor response time and system performance.

---

# 57. Control Timing

Control functions shall use independent timing.

Example:

```text
Temperature Control
Every 1 second

Flow Monitoring
Continuous / high frequency

Refill Control
Event driven + periodic verification

Peltier Protection
Fast monitoring

Permanent Logging
Every 60 seconds
```

The RTC shall not be used for these short control intervals.

---

# 58. Temperature Control State Machine

Example:

```text
NORMAL
  │
  ├── Temperature LOW
  │        ↓
  │      HEATING
  │
  └── Temperature HIGH
           ↓
        COOLING
```

Safety faults shall override all states.

Example:

```text
HEATING
   ↓
LOW WATER
   ↓
SAFE
```

---

# 59. Cooling State Machine

```text
COOLING_OFF
     │
     ▼
COOLING_REQUEST
     │
     ▼
SAFETY_CHECK
     │
 ┌───┴────┐
NO       YES
│          │
▼          ▼
FAULT    TEC1
           │
           ▼
     Higher Demand?
        │       │
       NO      YES
        │       │
        ▼       ▼
      TEC1   TEC1 + TEC2
```

Hot-side over-temperature shall transition immediately to a safe cooling state.

---

# 60. Refill State Machine

```text
IDLE
 ↓
LOW_LEVEL
 ↓
CHECK_RESERVOIR
 ↓
OPEN_VALVE
 ↓
VERIFY_FLOW
 ↓
REFILLING
 ↓
TARGET_REACHED
 ↓
CLOSE_VALVE
 ↓
IDLE
```

Any critical fault shall transition to:

```text
FAULT
```

The solenoid shall be OFF in the FAULT state.

---

# 61. Configuration Validation

Every configurable value shall have:

- Minimum allowed value.
- Maximum allowed value.
- Default value.
- Unit.
- Validation rule.

Example:

```text
Temperature Setpoint
Minimum: TBD
Maximum: TBD
Default: TBD
Unit: °C
```

The actual biological values shall be finalized after the crayfish species is confirmed.

---

# 62. Error Handling

The firmware shall avoid silent failures.

Errors shall be:

1. Detected.
2. Classified.
3. Logged.
4. Displayed where appropriate.
5. Handled according to severity.

Example:

```text
RS485 Timeout
     ↓
Sensor Error
     ↓
Log Event
     ↓
Alarm
     ↓
Apply Safety Response
```

---

# 63. Sensor Communication Recovery

For temporary communication failures:

```text
Communication Failure
        ↓
Retry
        ↓
Retry
        ↓
Successful?
    ┌───┴───┐
   YES      NO
    │        │
    ▼        ▼
 Recover   Sensor Fault
```

The system should distinguish between temporary communication loss and persistent sensor failure.

---

# 64. Power Recovery

After reboot:

```text
BOOT
 ↓
Initialize Hardware
 ↓
Read Configuration
 ↓
Read RTC
 ↓
Initialize Sensors
 ↓
Validate Safety Inputs
 ↓
Initialize Outputs OFF
 ↓
Initialize Services
 ↓
Evaluate Current Conditions
 ↓
Automatic Mode
```

The refill solenoid shall remain closed during startup.

Heater and Peltier outputs shall remain OFF until safety checks pass.

---

# 65. Data Integrity

The data logger shall reduce the risk of corrupted records.

The firmware should:

- Append complete records.
- Avoid partially written records where possible.
- Flush data periodically.
- Detect SD write errors.
- Report storage faults.
- Use separate event and sensor records.

A logging failure shall not stop environmental control.

---

# 66. Storage Failure Behavior

If the microSD card fails:

```text
SD Failure
    ↓
Storage Alarm
    ↓
Control Continues
```

The system shall continue operating essential control functions.

The HMI and web interface shall report that data logging is unavailable.

---

# 67. Network Failure Behavior

If Wi-Fi is lost:

```text
Wi-Fi Lost
    ↓
Web Interface Unavailable
    ↓
Automatic Control Continues
    ↓
HMI Continues
    ↓
Logging Continues
    ↓
RTC Continues
```

The system shall attempt network reconnection.

---

# 68. CSV Export Failure Behavior

If a CSV export fails:

- Automatic control shall continue.
- Logging shall continue.
- The error shall be reported to the user.
- The failure shall be logged where possible.

A CSV export shall never stop the data logger.

---

# 69. Security Considerations

Version 1 shall operate primarily on a local network.

The web interface should provide at least basic access protection before remote access is enabled.

The system shall not expose control endpoints to the public Internet by default.

Future versions may add:

- Authentication.
- User accounts.
- Role-based access.
- HTTPS.
- Remote access security.

---

# 70. Software Testing Strategy

The software shall be tested at multiple levels.

## Unit Testing

Test individual:

- Sensor parsers.
- Validation functions.
- State machines.
- Alarm logic.
- Configuration validation.
- CSV generation.

## Integration Testing

Test:

- Sensor to controller.
- Controller to equipment.
- RTC to logger.
- SD to CSV export.
- HMI to command system.
- Web to command system.

## System Testing

Test the complete incubator.

## Fault Testing

Test:

- Sensor disconnection.
- Flow loss.
- Low water.
- High water.
- Refill failure.
- Peltier over-temperature.
- SD failure.
- Wi-Fi failure.
- Power interruption.

---

# 71. Simulation and Development Mode

The firmware should provide a development or simulation mode.

Simulation mode may generate:

- Temperature values.
- DO values.
- pH values.
- Water level.
- Flow values.
- Sensor failures.

This allows software development before all physical sensors are installed.

Simulation mode shall never be enabled accidentally on a production system.

---

# 72. Logging Levels

The firmware should support:

```text
ERROR
WARNING
INFO
DEBUG
```

Production operation should use a limited logging level to avoid excessive storage writes.

Debug logging may be enabled during development.

---

# 73. Firmware Update Strategy

The firmware should support future firmware updates.

Possible mechanisms:

- USB/serial firmware update.
- OTA update through local Wi-Fi.

OTA shall not be required for Version 1 operation.

The system shall ensure that a failed update does not leave the controller permanently unusable where practical.

---

# 74. Software Configuration Interface

The HMI and web interface shall expose configuration values appropriate for operators.

Configuration shall be grouped by category:

```text
Temperature
Water Quality
Water Level
Flow
Refill
Cooling
Alarms
Logging
Network
System
```

Advanced hardware configuration should not be exposed to normal operators.

---

# 75. Software Safety Rules

The following rules shall always apply:

```text
IF emergency_stop
    THEN heater = OFF
    AND TEC1 = OFF
    AND TEC2 = OFF
    AND refill = OFF
```

```text
IF critical_low_water
    THEN heater = OFF
    AND cooling = OFF
```

```text
IF insufficient_flow
    THEN heater = OFF
    AND cooling = OFF
```

```text
IF peltier_overtemperature
    THEN TEC1 = OFF
    AND TEC2 = OFF
    AND fans = ON
```

```text
IF high_water
    THEN refill = OFF
```

```text
IF refill_flow_missing
    THEN refill = OFF
    AND alarm = ON
```

These rules shall have priority over normal control logic.

---

# 76. Main Control Loop

Conceptual control cycle:

```text
┌──────────────────────┐
│ Read Sensors         │
└──────────┬───────────┘
           ↓
┌──────────────────────┐
│ Validate Sensor Data │
└──────────┬───────────┘
           ↓
┌──────────────────────┐
│ Update System State  │
└──────────┬───────────┘
           ↓
┌──────────────────────┐
│ Evaluate Safety      │
└──────────┬───────────┘
           ↓
┌──────────────────────┐
│ Evaluate Alarms      │
└──────────┬───────────┘
           ↓
┌──────────────────────┐
│ Run Automatic Logic  │
└──────────┬───────────┘
           ↓
┌──────────────────────┐
│ Apply Equipment      │
└──────────┬───────────┘
           ↓
┌──────────────────────┐
│ Update HMI/Web State │
└──────────┬───────────┘
           ↓
┌──────────────────────┐
│ Log Periodic Data    │
└──────────────────────┘
```

The actual implementation may distribute these functions across FreeRTOS tasks.

---

# 77. Software Dependency Direction

Dependencies shall generally flow downward.

```text
UI
 ↓
Application
 ↓
Services
 ↓
Device Drivers
 ↓
Hardware
```

Hardware drivers shall not depend on the web interface or HMI.

The sensor layer shall not directly control equipment.

The web interface shall not directly manipulate GPIO.

---

# 78. API Abstraction

Hardware-dependent modules should expose simple interfaces.

Example:

```text
temperature.getPrimary()
temperature.getSecondary()

doSensor.getValue()
phSensor.getValue()

level.getPercent()
flow.getMainFlow()

heater.setState()
tec1.setPower()
tec2.setPower()

rtc.getDateTime()
logger.write()
```

The exact API shall be defined during implementation.

---

# 79. Future Expansion

The software architecture shall allow future additions such as:

- Automatic pH dosing.
- DO control.
- Additional tanks.
- Additional sensors.
- Feeding automation.
- Camera integration.
- Cloud synchronization.
- Remote notifications.
- Additional incubation profiles.
- Batch tracking.

New features should be implemented as separate modules where practical.

---

# 80. Software Design Acceptance Criteria

The software design shall be considered complete when:

1. All major hardware subsystems have a software interface.
2. Sensor acquisition is separated from control logic.
3. Safety logic has higher priority than normal control.
4. HMI and web commands use the same safety path.
5. Temperature control supports heating and Peltier cooling.
6. Two TEC channels can be controlled independently.
7. Refill uses a state machine.
8. Water flow is used as a heater and cooling interlock.
9. Water level is used as a refill and heater/cooling safety condition.
10. DO and pH use RS485 Modbus architecture.
11. TinyRTC provides persistent timestamps.
12. NTP can optionally synchronize the RTC.
13. Data is stored locally on microSD.
14. Event and alarm records are stored.
15. CSV export can stream historical data.
16. CSV export cannot block critical control.
17. Wi-Fi failure does not stop automatic operation.
18. SD failure does not stop automatic operation.
19. Power recovery enters a safe startup state.
20. Watchdog protection is implemented.
21. Sensor failures are detected and handled.
22. Manual controls cannot bypass critical safety interlocks.
23. The firmware is modular enough to support future expansion.