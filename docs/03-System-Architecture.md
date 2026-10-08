# 03 System Architecture

## 1. Purpose

This document defines the system architecture for the Crayfish Incubator Version 1.

The architecture describes:

- Major hardware subsystems.
- Software subsystems.
- Sensor interfaces.
- Control paths.
- Power architecture.
- Safety architecture.
- Data flow.
- Communication architecture.
- Local HMI.
- Web interface.
- Data logging.
- CSV export.
- Automatic control behavior.

The system shall use an ESP32-S3 as the central controller.

---

# 2. System Architecture Overview

The system shall use a centralized controller architecture.

```text
                         ┌───────────────────────┐
                         │       60L? Tank       │
                         │     60 US Gallons     │
                         │       ~227 Liters     │
                         └───────────┬───────────┘
                                     │
              ┌──────────────────────┼──────────────────────┐
              │                      │                      │
              ▼                      ▼                      ▼
       Temperature             Water Level              Water Quality
       DS18B20 ×2              Level Sensor             DO + pH
              │                      │                      │
              └──────────────────────┼──────────────────────┘
                                     │
                                     ▼
                           ┌───────────────────┐
                           │     ESP32-S3      │
                           │   Main Controller │
                           └─────────┬─────────┘
                                     │
          ┌──────────────┬───────────┼───────────┬──────────────┐
          │              │           │           │              │
          ▼              ▼           ▼           ▼              ▼
       HMI            Web UI       TinyRTC      microSD       RS485
     Touchscreen       Wi-Fi       I2C          Logging      Sensors
          │              │
          │              │
          └──────────────┘
                                     │
                  ┌──────────────────┼──────────────────┐
                  │                  │                  │
                  ▼                  ▼                  ▼
               Heating           Cooling            Refill
               System            System             System
                  │                  │                  │
                  ▼                  ▼                  ▼
               Heater          TEC ×2 + Fans       Solenoid
                                     │
                                     │
                  ┌──────────────────┼──────────────────┐
                  │                  │                  │
                  ▼                  ▼                  ▼
             Circulation         Aeration           Safety
                Pump             Pump/System         System
```

The ESP32-S3 shall coordinate monitoring, control, logging, HMI, web access, and alarms.

Safety functions shall be implemented independently where practical.

---

# 3. Major Subsystems

The system shall consist of the following major subsystems:

1. Main Tank
2. Water Circulation
3. Water Heating
4. Peltier Cooling
5. Aeration
6. Automatic Refill
7. Water Quality Monitoring
8. Water Level Monitoring
9. Temperature Monitoring
10. Main Controller
11. HMI
12. Web Interface
13. Real-Time Clock
14. Data Storage
15. Alarm System
16. Electrical Safety
17. Power Distribution

---

# 4. Main Tank Architecture

The main tank is the primary process vessel.

Nominal capacity:

```text
60 US gallons
≈ 227 liters
```

The tank shall contain:

- Crayfish incubation environment.
- Temperature sensors.
- Water-level measurement.
- Independent level safety switches.
- Circulation return.
- Aeration diffusers.
- Water circulation outlet.

The tank shall provide sufficient freeboard for normal operation and emergency protection.

---

# 5. Water Circulation Architecture

The primary circulation loop shall provide continuous water movement.

Recommended architecture:

```text
                 ┌──────────────────┐
                 │    Main Tank     │
                 └────────┬─────────┘
                          │
                          ▼
                    Coarse Filter
                          │
                          ▼
                   Circulation Pump
                          │
                          ▼
                    Flow Sensor
                          │
                          ▼
                    Water Block
                          │
                          ▼
                 Optional Filtration
                          │
                          ▼
                 ┌──────────────────┐
                 │    Main Tank     │
                 └──────────────────┘
```

The circulation loop shall provide water flow through the Peltier water block.

The exact filter arrangement shall be finalized during hardware design.

---

# 6. Water Temperature Architecture

Two DS18B20 sensors shall monitor the water temperature.

```text
DS18B20 T1 ──┐
             ├── 1-Wire ── ESP32-S3
DS18B20 T2 ──┘
```

T1 shall be the primary temperature measurement.

T2 shall provide independent verification.

The controller shall compare both sensors.

If the difference exceeds the configured tolerance:

```text
Temperature Difference
        ↓
Sensor Warning
        ↓
Alarm
        ↓
Evaluate Safety
```

---

# 7. Heating Architecture

The heater shall be controlled through an electrically isolated switching device.

```text
                ESP32-S3
                    │
                    ▼
             Heater Driver
                    │
                    ▼
             Aquarium Heater
                    │
                    ▼
                Main Tank
```

The heater shall also have an independent thermostat.

The software shall control the heater based on water temperature.

The hardware thermostat shall provide secondary protection.

The heater shall be disabled when:

- Water level is critically low.
- Required circulation is unavailable.
- Temperature sensor is invalid.
- High temperature protection is active.
- Emergency stop is active.

---

# 8. Peltier Cooling Architecture

The system shall use two TEC1-12706 modules.

The cooling architecture shall be:

```text
                       Water Loop
                           │
                           ▼
                     Water Block
                           │
                    ┌──────┴──────┐
                    │             │
                 TEC 1          TEC 2
                    │             │
                    └──────┬──────┘
                           │
                    Hot-Side Heatsink
                           │
                    High-Airflow Fans
                           │
                           ▼
                         Air
```

Each TEC shall have independent control.

The controller shall support:

```text
TEC 1 OFF
TEC 1 ON
TEC 2 OFF
TEC 2 ON
TEC 1 + TEC 2
```

The Peltier system shall not be powered directly from the ESP32-S3.

---

# 9. Peltier Power Architecture

The Peltier system shall have a dedicated power path.

Conceptual architecture:

```text
              DC Power Supply
                     │
                     ▼
              TEC Power Driver
                ┌────┴────┐
                │         │
                ▼         ▼
              TEC 1      TEC 2
```

The final electrical design shall determine whether the system uses:

- Dedicated 12 V high-current supply.
- 24 V supply with appropriate DC-DC conversion.
- Dedicated TEC controllers.

TEC1-12706 modules shall not be connected directly to a 24 V supply.

The final driver shall support the required current and thermal operating conditions.

---

# 10. Peltier Thermal Protection

A temperature sensor shall monitor the hot-side heatsink.

```text
Hot-Side Temperature
        │
        ▼
    ESP32-S3
        │
        ├──── Normal → Cooling Allowed
        │
        └──── Overtemp → TEC OFF
                              │
                              ▼
                           Fans ON
                              │
                              ▼
                            Alarm
```

The fans shall remain active when required to remove residual heat.

The TECs shall remain disabled until the heatsink returns to a safe temperature.

---

# 11. Aeration Architecture

Aeration shall operate independently from water circulation.

```text
Air Pump
    │
    ▼
Air Manifold
    │
 ┌──┼──┬──┐
 ▼  ▼  ▼  ▼
Air Stones
    │
    ▼
Main Tank
```

The aeration system shall normally operate continuously.

Dissolved oxygen measurements shall be used for monitoring and alarm generation.

Version 1 shall not automatically dose oxygen.

---

# 12. Dissolved Oxygen Architecture

The DO system shall preferably use an industrial or laboratory sensor with an RS485 Modbus RTU transmitter.

```text
DO Probe
    │
    ▼
DO Transmitter
    │
    │ RS485
    ▼
ESP32-S3
```

The ESP32-S3 shall periodically poll the sensor.

The software shall detect:

- Communication timeout.
- Invalid values.
- Sensor errors where supported.
- Low DO.

---

# 13. pH Architecture

The pH subsystem shall use a replaceable pH probe and transmitter.

Preferred architecture:

```text
pH Probe
    │
    ▼
pH Transmitter
    │
    │ RS485 Modbus RTU
    ▼
ESP32-S3
```

The pH transmitter shall provide a conditioned measurement suitable for long cable runs.

The ESP32-S3 shall periodically poll the transmitter.

---

# 14. Main Tank Level Architecture

The main tank shall have two levels of measurement.

### Primary Measurement

Continuous level sensor.

```text
Continuous Level Sensor
          │
          ▼
      ESP32-S3
```

### Safety Measurement

Independent level switches.

```text
LOW Float ────────┐
                  ├── Safety Inputs
HIGH Float ───────┘
                       │
                       ▼
                    ESP32-S3
```

The independent switches shall be used for critical protection.

The continuous sensor shall primarily provide measurement and control information.

---

# 15. Refill Architecture

The refill system shall use an elevated reservoir.

```text
             Refill Reservoir
                    │
             Reservoir Level
                    │
                    ▼
             Manual Shutoff
                    │
                    ▼
          Normally-Closed Solenoid
                    │
                    ▼
              Refill Flow Sensor
                    │
                    ▼
                Main Tank
```

The reservoir shall be positioned above the main tank where practical to provide gravity-assisted refill.

The solenoid shall remain closed unless the controller explicitly opens it.

---

# 16. Automatic Refill Control

The refill state machine shall operate approximately as follows:

```text
                    ┌──────────────┐
                    │ Normal Level │
                    └──────┬───────┘
                           │
                           ▼
                    Level Below Low
                           │
                           ▼
                 Check Reservoir Level
                           │
                 ┌─────────┴─────────┐
                 │                   │
              Available            Empty
                 │                   │
                 ▼                   ▼
          Open Solenoid            Alarm
                 │
                 ▼
          Check Refill Flow
                 │
          ┌──────┴──────┐
          │             │
        Flow          No Flow
          │             │
          ▼             ▼
       Continue       Fault
          │             │
          ▼             ▼
   Target Level      Valve OFF
          │           Alarm
          ▼
    Valve OFF
          │
          ▼
     Normal Level
```

High-level protection shall immediately close the refill valve.

---

# 17. Refill Safety Architecture

The refill system shall have multiple independent protection mechanisms.

```text
Continuous Level
       │
       ├─────────────┐
       │             │
       ▼             ▼
 Refill Control   High Float
       │             │
       └──────┬──────┘
              ▼
        Solenoid Control
```

The system shall stop refilling if:

- High float activates.
- Emergency high level activates.
- Reservoir is empty.
- Refill flow is missing.
- Refill timeout expires.
- Emergency stop activates.
- Controller detects a refill fault.

The normally-closed solenoid shall close when controller power is removed.

---

# 18. TinyRTC Architecture

The TinyRTC shall provide the system's persistent local wall clock.

```text
                  TinyRTC
                    │
                   I2C
                    │
                    ▼
                ESP32-S3
                    │
          ┌─────────┴──────────┐
          ▼                    ▼
       Logging              Events
          │                    │
          └─────────┬──────────┘
                    ▼
                 microSD
```

The RTC shall include a backup battery.

The system shall be able to operate using the RTC when Wi-Fi is unavailable.

If Wi-Fi is available, the system may synchronize time through NTP.

The RTC shall not be used for high-speed control timing.

---

# 19. Controller Architecture

The ESP32-S3 shall be divided logically into independent software subsystems.

```text
                       ESP32-S3
                           │
       ┌───────────────────┼────────────────────┐
       │                   │                    │
       ▼                   ▼                    ▼
   Sensor Layer       Control Layer        Interface Layer
       │                   │                    │
       │                   │              ┌─────┴─────┐
       │                   │              │           │
       │                   │             HMI        Web
       │                   │
       ├─────────────┬─────┼───────────────┐
       │             │     │               │
       ▼             ▼     ▼               ▼
 Temperature       Water  Safety        Equipment
  Manager         Quality Manager        Manager
```

The system shall separate sensor acquisition from control logic.

This shall make the software easier to test and maintain.

---

# 20. Software Architecture

The software shall be divided into logical layers.

```text
┌──────────────────────────────────────────────┐
│              User Interface Layer            │
│                                              │
│             HMI + Web Interface              │
├──────────────────────────────────────────────┤
│               Application Layer               │
│                                              │
│ Temperature │ Refill │ Alarm │ Operating    │
│ Control     │ Control│ Mgmt  │ Modes        │
├──────────────────────────────────────────────┤
│                 Service Layer                │
│                                              │
│ Logging │ RTC │ Configuration │ CSV Export   │
├──────────────────────────────────────────────┤
│                Device Layer                  │
│                                              │
│ DS18B20 │ RS485 │ Level │ Flow │ GPIO │ SD   │
├──────────────────────────────────────────────┤
│                 Hardware                     │
│                                              │
│ ESP32-S3 + Connected Devices                 │
└──────────────────────────────────────────────┘
```

---

# 21. Sensor Layer

The sensor layer shall provide normalized sensor values to the application layer.

The sensor layer shall handle:

- Sensor initialization.
- Sensor reading.
- Validation.
- Timeout.
- Communication errors.
- Sensor status.
- Unit conversion where required.

The application layer shall not directly access low-level hardware drivers.

---

# 22. Temperature Manager

The Temperature Manager shall handle:

- T1.
- T2.
- Temperature validation.
- Heating control.
- Cooling control.
- Temperature alarms.

Conceptual flow:

```text
T1 + T2
   │
   ▼
Temperature Validation
   │
   ▼
Temperature Manager
   │
   ├───────────────┐
   ▼               ▼
 Heater          Peltier
 Control         Control
```

The Temperature Manager shall enforce:

- Hysteresis.
- Sensor validation.
- Low-water protection.
- Low-flow protection.
- Peltier thermal protection.
- High-temperature protection.

---

# 23. Water Quality Manager

The Water Quality Manager shall handle:

- DO.
- pH.
- Sensor status.
- Communication status.
- Alarm thresholds.

The manager shall not automatically dose chemicals in Version 1.

---

# 24. Water Level Manager

The Water Level Manager shall handle:

- Continuous tank level.
- Low float.
- High float.
- Refill threshold.
- Target level.
- Emergency high-level condition.

The manager shall provide the refill controller with validated level status.

---

# 25. Refill Manager

The Refill Manager shall control:

- Reservoir availability.
- Solenoid.
- Refill flow.
- Refill timeout.
- Target level.
- High-level protection.
- Refill event logging.

The Refill Manager shall use a state machine.

Example states:

```text
IDLE
WAITING_FOR_LEVEL
CHECKING_RESERVOIR
REFILLING
VERIFYING_FLOW
COMPLETING
FAULT
LOCKOUT
```

---

# 26. Equipment Manager

The Equipment Manager shall manage:

- Heater.
- TEC 1.
- TEC 2.
- Cooling fans.
- Circulation pump.
- Aeration pump.
- Refill solenoid.

The Equipment Manager shall enforce hardware safety conditions before allowing equipment activation.

---

# 27. Alarm Manager

The Alarm Manager shall centralize system alarms.

Alarm sources include:

- Temperature.
- DO.
- pH.
- Water level.
- Flow.
- Peltier temperature.
- Sensor communication.
- Refill system.
- RTC.
- Power/system faults.

Each alarm shall have:

- Alarm ID.
- Timestamp.
- Severity.
- Source.
- Current state.
- Trigger value where applicable.

Alarm states shall include:

```text
NORMAL
WARNING
ACTIVE
ACKNOWLEDGED
CLEARED
```

---

# 28. Data Logger

The Data Logger shall store periodic measurements.

The logging path shall be:

```text
Sensors
   │
   ▼
Validated Data
   │
   ▼
Data Logger
   │
   ▼
microSD
```

The Data Logger shall not directly control equipment.

The logger shall use TinyRTC timestamps.

The logger shall operate independently from the web interface.

---

# 29. Event Logger

The Event Logger shall record important system events.

Examples:

```text
SYSTEM_START
SYSTEM_STOP
HEATER_ON
HEATER_OFF
TEC1_ON
TEC1_OFF
TEC2_ON
TEC2_OFF
PUMP_ON
PUMP_OFF
REFILL_START
REFILL_COMPLETE
REFILL_FAULT
ALARM_ACTIVE
ALARM_CLEARED
SENSOR_FAULT
SENSOR_RECOVERED
EMERGENCY_STOP
```

Events shall be stored on the microSD card.

---

# 30. CSV Export Architecture

CSV export shall be handled by a dedicated service.

```text
Web Browser
     │
     │ HTTP Request
     ▼
ESP32-S3 Web Server
     │
     ▼
CSV Export Service
     │
     ▼
microSD
     │
     ▼
CSV Stream
     │
     ▼
Web Browser
```

The system shall stream data where practical.

The export service shall not load the entire dataset into RAM.

CSV export shall support:

- Start date.
- Start time.
- End date.
- End time.

The export service shall read the required records from the microSD card.

---

# 31. HMI Architecture

The HMI shall communicate with the ESP32-S3 application layer.

```text
                 ESP32-S3
                    │
              Application State
                    │
                    ▼
                  HMI
                    │
          ┌─────────┴─────────┐
          ▼                   ▼
       Display              Touch
```

The HMI shall not directly control hardware.

User commands shall pass through the application and safety layers.

Example:

```text
Touch Input
     ↓
HMI Handler
     ↓
Command Validation
     ↓
Safety Manager
     ↓
Equipment Manager
     ↓
Hardware Output
```

---

# 32. Web Interface Architecture

The web interface shall operate as a local interface hosted by the ESP32-S3.

```text
Phone / Tablet / PC
         │
       Wi-Fi
         │
         ▼
    ESP32-S3 Web Server
         │
         ▼
 Application State
```

The web interface shall provide:

- Live monitoring.
- Historical data.
- Graphs.
- Alarm history.
- Equipment status.
- Configuration.
- Manual control.
- CSV export.

The web interface shall not directly access hardware drivers.

---

# 33. Web Control Safety

All commands from the web interface shall pass through the same safety system used by the HMI.

```text
Web Command
     ↓
Command Handler
     ↓
Safety Validation
     ↓
Equipment Manager
     ↓
Output
```

This prevents a web request from bypassing safety logic.

The same architecture shall apply to manual HMI controls.

---

# 34. Operating Mode Architecture

The Operating Mode Manager shall determine the active control strategy.

```text
                 Operating Mode
                       │
       ┌───────────────┼────────────────┐
       │               │                │
       ▼               ▼                ▼
   Automatic        Manual         Maintenance
       │               │                │
       └───────────────┼────────────────┘
                       ▼
                 Safety Manager
```

Available modes:

- Automatic.
- Manual.
- Maintenance.
- Alarm.

Safety functions shall remain active regardless of operating mode.

---

# 35. Safety Architecture

Safety shall be implemented at multiple levels.

## Level 1: Hardware Protection

Examples:

- RCD/GFCI.
- Circuit breakers.
- Fuses.
- Heater thermostat.
- Normally-closed refill solenoid.
- Independent level switches.
- Emergency stop.

## Level 2: Firmware Safety

Examples:

- Low-water interlock.
- Low-flow interlock.
- Peltier over-temperature protection.
- Sensor validation.
- Refill timeout.
- High-level protection.

## Level 3: Alarm System

The system shall notify the operator of unsafe or abnormal conditions.

---

# 36. Emergency Stop Architecture

The emergency stop shall remove power from designated high-power equipment.

The exact electrical implementation shall be finalized during hardware design.

The architecture should allow the ESP32-S3 to remain powered where practical so it can:

- Detect the emergency state.
- Record the event.
- Display the alarm.
- Preserve system diagnostics.

The emergency stop shall not be implemented only as a software command.

---

# 37. Power Architecture

The system shall separate power into logical domains.

Conceptual architecture:

```text
                    AC INPUT
                       │
             ┌─────────┴─────────┐
             │                   │
          Protection          RCD/GFCI
             │                   │
             └─────────┬─────────┘
                       │
                Main Power Panel
                       │
       ┌───────────────┼────────────────┐
       │               │                │
       ▼               ▼                ▼
    Heater         DC Power          Other AC
                    Supply            Loads
                       │
             ┌─────────┼─────────┐
             │         │         │
             ▼         ▼         ▼
          ESP32      Sensors    TEC Driver
```

The exact voltage rails shall be finalized during hardware design.

---

# 38. Communication Architecture

The communication architecture shall use:

```text
                         ESP32-S3
                             │
        ┌────────────┬───────┼────────────┬─────────────┐
        │            │       │            │             │
        ▼            ▼       ▼            ▼             ▼
      1-Wire        I2C    RS485         SPI           GPIO
        │            │       │            │             │
        ▼            ▼       ▼            ▼             ▼
    DS18B20       TinyRTC  DO/pH        microSD     Level/Flow
```

The ESP32-S3 shall centralize data collection.

---

# 39. RS485 Architecture

The DO and pH systems shall preferably share an RS485 bus.

```text
ESP32-S3
    │
RS485 Transceiver
    │
    ├────────────── DO Transmitter
    │
    └────────────── pH Transmitter
```

Each Modbus device shall have a unique address.

The software shall poll devices sequentially.

The RS485 bus shall include appropriate termination and biasing according to the final wiring length and device requirements.

---

# 40. Data Flow Architecture

The overall data flow shall be:

```text
             Sensors
                │
                ▼
        Sensor Acquisition
                │
                ▼
          Data Validation
                │
       ┌────────┼─────────┐
       │        │         │
       ▼        ▼         ▼
    Control   Logging   Display
       │        │         │
       │        ▼         ├──── HMI
       │      microSD     │
       │                  └──── Web
       ▼
   Equipment
```

Alarm processing shall receive validated sensor data.

---

# 41. Control Flow

The main control flow shall be:

```text
Read Sensors
     ↓
Validate Sensors
     ↓
Update System State
     ↓
Evaluate Safety
     ↓
Evaluate Alarms
     ↓
Run Automatic Control
     ↓
Update Equipment
     ↓
Log Data
     ↓
Update HMI
     ↓
Update Web Interface
```

Safety evaluation shall occur before equipment activation.

---

# 42. Temperature Control Flow

```text
Read T1
Read T2
   │
   ▼
Validate Sensors
   │
   ▼
Compare T1/T2
   │
   ▼
Check Safety
   │
   ├───────────────┐
   ▼               ▼
Heating         Cooling
Decision        Decision
   │               │
   ▼               ▼
Heater          TEC Control
Control
```

The temperature controller shall prevent simultaneous heating and cooling.

---

# 43. Cooling Control Flow

```text
Water Temperature
        │
        ▼
Cooling Decision
        │
        ▼
Check:
- Water Level
- Water Flow
- Hot-Side Temperature
- Sensor Status
        │
        ▼
Cooling Allowed?
        │
   ┌────┴────┐
   │         │
  YES        NO
   │         │
   ▼         ▼
TEC Control  TEC OFF
             Fans ON if required
```

---

# 44. Refill Control Flow

```text
Main Level
    │
    ▼
Below Threshold?
    │
 ┌──┴──┐
No    Yes
│      │
│      ▼
│  Reservoir Available?
│      │
│   ┌──┴──┐
│  No    Yes
│   │      │
│   ▼      ▼
│ Alarm  Open Valve
│          │
│          ▼
│     Check Flow
│          │
│     ┌────┴────┐
│    Flow     No Flow
│     │          │
│     ▼          ▼
│  Continue     Fault
│     │
│     ▼
Target Level?
│
▼
Close Valve
```

High-level safety shall override all normal refill commands.

---

# 45. Alarm Flow

```text
Sensor Data
     │
     ▼
Alarm Evaluation
     │
     ├──────────────┐
     ▼              ▼
 Normal           Fault
                    │
                    ▼
               Alarm Manager
                    │
          ┌─────────┼─────────┐
          ▼         ▼         ▼
        HMI       Web       microSD
```

Critical alarms shall also cause the affected equipment to enter a safe state.

---

# 46. Data Storage Architecture

The system shall use microSD for persistent storage.

Conceptual structure:

```text
microSD
│
├── config/
│
├── data/
│   └── sensor/
│
├── events/
│
├── alarms/
│
└── exports/
```

The exact file organization shall be finalized during software design.

The system shall avoid storing high-frequency temporary control data permanently unless required.

---

# 47. Configuration Storage

System configuration shall be stored in non-volatile memory.

Configuration shall include:

- Temperature setpoints.
- Temperature hysteresis.
- Water-level thresholds.
- Flow thresholds.
- DO limits.
- pH limits.
- Peltier limits.
- Alarm delays.
- Refill timeout.
- Logging configuration.
- System settings.

The microSD card shall not be the only copy of critical configuration.

---

# 48. Startup Architecture

After power is applied:

```text
Power On
   ↓
ESP32-S3 Boot
   ↓
Initialize GPIO
   ↓
Initialize Safety Inputs
   ↓
Initialize TinyRTC
   ↓
Initialize microSD
   ↓
Initialize Sensors
   ↓
Initialize HMI
   ↓
Initialize Wi-Fi/Web
   ↓
Validate System State
   ↓
Enter Safe Startup State
   ↓
Automatic Operation
```

The refill valve shall remain closed during startup.

The heater and Peltier system shall remain OFF until required safety conditions are verified.

---

# 49. Shutdown Architecture

During controlled shutdown:

```text
Shutdown Request
       ↓
Stop Automatic Control
       ↓
Heater OFF
       ↓
TEC OFF
       ↓
Refill Valve CLOSED
       ↓
Save Important State
       ↓
Log Shutdown Event
       ↓
System Safe State
```

The circulation pump and aeration system may remain active depending on the shutdown reason and configured safety strategy.

---

# 50. Power Recovery Architecture

After unexpected power loss, the system shall not immediately restore all equipment.

The startup sequence shall verify:

- Water level.
- Water flow.
- Temperature sensors.
- Safety switches.
- Peltier heatsink temperature.
- Refill reservoir.
- Critical sensor communication.

Only after validation shall automatic equipment operation resume.

---

# 51. Web and HMI Data Synchronization

The HMI and web interface shall use the same application state.

```text
                 Application State
                  /             \
                 /               \
                ▼                 ▼
              HMI                Web
```

This ensures that:

- Sensor values match.
- Equipment states match.
- Alarm states match.
- Configuration values match.

Neither interface shall maintain an independent copy of the actual equipment state.

---

# 52. Manual Control Architecture

Manual commands shall follow:

```text
User
 │
 ├── HMI
 │
 └── Web
       │
       ▼
Command Handler
       │
       ▼
Safety Manager
       │
       ▼
Equipment Manager
       │
       ▼
Hardware
```

A manual command shall be rejected if it violates a safety condition.

Examples:

```text
Manual Heater ON
       ↓
Check Water Level
       ↓
Check Flow
       ↓
Check Temperature
       ↓
Allowed?
```

If any critical safety condition fails, the command shall be rejected.

---

# 53. Network Architecture

The system shall operate as a local Wi-Fi device.

Conceptual architecture:

```text
                Local Wi-Fi Network
                       │
        ┌──────────────┼──────────────┐
        │              │              │
        ▼              ▼              ▼
     Laptop         Tablet         Phone
        │              │              │
        └──────────────┼──────────────┘
                       │
                       ▼
                   ESP32-S3
                  Web Server
```

Internet connectivity shall not be required for normal operation.

NTP synchronization shall be optional.

---

# 54. System State Model

The system shall maintain a centralized system state.

Example:

```text
System State
│
├── Operating Mode
├── Temperature State
├── Water Quality State
├── Water Level State
├── Flow State
├── Refill State
├── Heating State
├── Cooling State
├── Aeration State
├── Alarm State
├── RTC State
├── Storage State
└── Network State
```

The centralized state shall be available to the HMI, web interface, logger, and alarm manager.

---

# 55. Fault Handling Architecture

Faults shall be categorized by severity.

### Informational

Example:

- Wi-Fi disconnected.

### Warning

Example:

- Temperature sensor disagreement.

### Critical

Examples:

- Main tank emergency high level.
- No circulation.
- Peltier over-temperature.
- Critical sensor failure.
- Refill failure.

Critical faults shall force affected equipment into a safe state.

---

# 56. Architecture Principles

The system shall follow these principles:

1. Safety has priority over automatic control.
2. Automatic control has priority over manual commands.
3. Hardware protection shall supplement software protection.
4. Critical control shall not depend on Wi-Fi.
5. Sensor failures shall be detected where possible.
6. High-power equipment shall use appropriate drivers.
7. HMI and web control shall use the same safety logic.
8. Data logging shall not interfere with control.
9. CSV export shall not interfere with control.
10. The TinyRTC shall provide persistent local time.
11. Biological limits shall remain configurable.
12. The architecture shall allow future expansion.
13. The system shall fail toward a safe state.
14. Version 1 shall operate without cloud dependency.

---

# 57. Version 1 Architecture Summary

The complete Version 1 architecture is:

```text
                         ┌─────────────────────┐
                         │      MAIN TANK      │
                         │     ~227 Liters     │
                         └──────────┬──────────┘
                                    │
          ┌─────────────────────────┼────────────────────────┐
          │                         │                        │
          ▼                         ▼                        ▼
     DS18B20 ×2               Level Sensor             Water Quality
          │                         │                  DO + pH
          │                         │                        │
          └─────────────────────────┼────────────────────────┘
                                    │
                                    ▼
                         ┌─────────────────────┐
                         │      ESP32-S3       │
                         │   Main Controller   │
                         └──────────┬──────────┘
                                    │
          ┌────────────┬────────────┼────────────┬─────────────┐
          │            │            │            │             │
          ▼            ▼            ▼            ▼             ▼
        TinyRTC       microSD      HMI          Wi-Fi         RS485
          │            │            │            │             │
          │            │            │            │        DO + pH
          │            │            │            │
          └────────────┴────────────┴────────────┴─────────────┘
                                    │
                 ┌──────────────────┼──────────────────┐
                 │                  │                  │
                 ▼                  ▼                  ▼
              HEATER             PELTIER            REFILL
                 │                  │                  │
                 │             TEC 1 + TEC 2       Solenoid
                 │                  │                  │
                 │             Hot Heatsink        Flow Sensor
                 │                  │                  │
                 └──────────────────┼──────────────────┘
                                    │
                         ┌──────────┴──────────┐
                         │                     │
                         ▼                     ▼
                   CIRCULATION             AERATION
                      PUMP                    PUMP
                         │                     │
                      Flow Sensor          Air Stones
                         │
                         ▼
                    WATER BLOCK
                         │
                         ▼
                      MAIN TANK
```

This architecture provides the foundation for the hardware design, software architecture, electrical design, and implementation plan.