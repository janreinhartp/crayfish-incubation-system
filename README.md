# Crayfish Incubator

A prototype automated crayfish incubation and aquatic environment control system designed for continuous monitoring and control of water temperature, dissolved oxygen, pH, water level, circulation, aeration, and automatic water replenishment.

The system is designed around an **ESP32-S3** controller with a **7-inch touchscreen HMI**, local web interface, microSD data logging, and independent hardware safety protections.

---

## 1. Project Overview

The Crayfish Incubator is a prototype environmental control system for a **60 US gallon, approximately 227-liter tank**.

The system continuously monitors and controls the tank environment while providing:

- Automatic temperature control
- Heating
- Peltier-based cooling
- Dissolved oxygen monitoring
- pH monitoring
- Water-level monitoring
- Circulation monitoring
- Continuous aeration
- Automatic water refill
- Alarm management
- Local touchscreen control
- Local web monitoring
- Historical data logging
- CSV data export
- Persistent date/time using TinyRTC
- Safety interlocks
- Power-loss recovery

The system is intended to operate continuously, 24 hours per day.

---

## 2. Project Goals

The primary goals are:

1. Maintain stable water temperature.
2. Monitor dissolved oxygen.
3. Monitor pH.
4. Maintain adequate water circulation.
5. Maintain continuous aeration.
6. Automatically replenish water when required.
7. Protect the system from low-water and high-water conditions.
8. Protect the heater from unsafe operating conditions.
9. Protect the Peltier cooling system from overheating.
10. Record historical operating data.
11. Provide local HMI control.
12. Provide local web monitoring.
13. Continue automatic operation without Wi-Fi.
14. Recover safely after power interruption.
15. Provide a modular platform for future expansion.

---

## 3. Important Design Principle

This is a **safety-first automation system**.

The ESP32-S3 provides intelligent control, but critical safety conditions shall not depend solely on normal software operation.

Independent protection is provided through components such as:

- Float switches
- Independent heater thermostat
- Fuses
- Circuit protection
- RCD/GFCI
- Emergency stop
- Peltier thermal monitoring
- Flow monitoring
- Normally closed refill solenoid

Safety conditions always override normal automatic or manual commands.

---

## 4. System Architecture

```text
                         ┌──────────────────────┐
                         │    7" Touchscreen    │
                         │       HMI            │
                         └──────────┬───────────┘
                                    │
                                    ▼
                         ┌──────────────────────┐
                         │      ESP32-S3        │
                         │   Main Controller    │
                         └──────┬────┬────┬─────┘
                                │    │    │
              ┌─────────────────┘    │    └──────────────────┐
              │                      │                       │
              ▼                      ▼                       ▼
        ┌───────────┐         ┌───────────┐          ┌────────────┐
        │  Sensors  │         │ TinyRTC   │          │  microSD   │
        └─────┬─────┘         └───────────┘          └────────────┘
              │
      ┌───────┼─────────────────────────────┐
      │       │             │               │
      ▼       ▼             ▼               ▼
 Temperature  RS485        Level           Flow
 DS18B20      DO / pH      Sensors         Sensors
      │
      ▼
┌──────────────────────────────────────────────────────────┐
│                    CONTROL SYSTEM                        │
│                                                          │
│ Temperature │ Refill │ Safety │ Alarm │ Equipment       │
│ Controller  │ Control│ Manager│ Mgmt  │ Manager         │
└─────────────┬──────────────┬──────────────┬─────────────┘
              │              │              │
              ▼              ▼              ▼
           Heater        Peltier         Refill
                         Cooling         System
              │              │              │
              ▼              ▼              ▼
          Heater       TEC 1 + TEC 2   Solenoid
                       + Fans
              │
              └───────────────────────────────┐
                                              │
                              ┌───────────────┐ │
                              │ Circulation   │◄┘
                              │ Pump          │
                              └───────────────┘

                              ┌───────────────┐
                              │ Aeration Pump │
                              └───────────────┘
```

---

## 5. Main Hardware

### Controller

- ESP32-S3

### HMI

- 7-inch touchscreen
- Waveshare ESP32-S3 Touch LCD 7B planned

### Temperature

- 2 × waterproof DS18B20

```text
T1 = Primary
T2 = Independent verification
```

### Dissolved Oxygen

- Laboratory/industrial DO probe
- DO transmitter
- RS485 Modbus RTU

### pH

- Replaceable laboratory/industrial pH probe
- pH transmitter
- RS485 Modbus RTU

### Timekeeping

- TinyRTC
- I2C interface
- Optional NTP synchronization

### Storage

- microSD
- Sensor data
- Events
- Alarms
- CSV export source data

### Water Level

- Continuous level sensor
- Low float switch
- High float switch
- Refill reservoir low-level sensor

### Flow

- Main circulation flow sensor
- Refill flow sensor

### Heating

- Aquarium-rated heater
- Independent thermostat

### Cooling

- Exactly 2 × TEC1-12706
- Water-cooled cold side
- Water block
- Large hot-side heatsink
- High-airflow fans
- Independent TEC control

### Circulation

- Continuous-duty circulation pump
- Initial target approximately 1,000 to 2,000 L/h

Final sizing depends on actual plumbing and head pressure.

### Aeration

- Air pump
- Manifold
- Multiple air stones
- Check valves

### Refill

- Elevated refill reservoir
- Manual shutoff valve
- Optional filter
- Normally closed solenoid
- Refill flow sensor

---

## 6. Peltier Cooling

Version 1 uses exactly:

**2 × TEC1-12706**

The cooling architecture is:

```text
Water Loop
     │
     ▼
Water Block
     │
     ▼
TEC 1 + TEC 2
     │
     ▼
Large Heatsink
     │
     ▼
High-Airflow Fans
```

The TEC modules are independently controlled.

### Important

TEC1-12706 modules are 12 V-class devices.

They **must not be connected directly to a 24 V supply**.

The final design shall use:

- Appropriate TEC drivers
- Suitable 12 V-class power
- Proper current protection
- Independent TEC channels

### Cooling Validation

Two TEC1-12706 modules may be marginal for cooling approximately 227 L of water under high ambient conditions.

The cooling system therefore requires experimental validation.

Minimum recommended test:

**24 to 72 hours**

Preferred test:

**7 days**

Testing shall include:

- Ambient temperature
- Water temperature
- TEC current
- TEC voltage
- Hot-side temperature
- Water flow
- Fan operation
- Cooling rate
- Temperature stability

---

## 7. Water Circulation

Initial water loop:

```text
Main Tank
    ↓
Coarse Filter
    ↓
Circulation Pump
    ↓
Main Flow Sensor
    ↓
Peltier Water Block
    ↓
Fine Filtration if required
    ↓
Tank Return
```

The circulation system is monitored continuously.

If flow becomes insufficient:

```text
Heater → OFF
TEC 1  → OFF
TEC 2  → OFF
Alarm  → ON
```

---

## 8. Automatic Refill

The refill system uses an elevated reservoir.

```text
Refill Reservoir
       ↓
Manual Valve
       ↓
Optional Filter
       ↓
NC Solenoid
       ↓
Refill Flow Sensor
       ↓
Main Tank
```

The solenoid is normally closed.

Therefore:

```text
Power OFF → Valve CLOSED
```

Refill requires:

- Low main-tank level
- Sufficient reservoir water
- No high-level condition
- No emergency stop
- No refill lockout
- Valid safety conditions

---

## 9. Aeration

Aeration is independent from water circulation.

```text
Air Pump
    ↓
Manifold
 ┌──┼──┬──┐
 ▼  ▼  ▼  ▼
Air Stones
```

Aeration normally remains ON.

Version 1 does not automatically increase aeration based on DO unless variable aeration hardware is added later.

---

## 10. Safety Architecture

Safety has the highest priority.

### Critical Inputs

- Main low-water float
- Main high-water float
- Refill reservoir low-level sensor
- Main circulation flow
- Peltier heatsink temperature
- Temperature sensor validity
- Emergency stop

### Safety Responses

| Condition | Response |
|---|---|
| Critical low water | Heater OFF, cooling OFF |
| High water | Refill OFF |
| No circulation | Heater OFF, cooling OFF |
| Peltier overtemperature | TECs OFF, fans ON |
| Refill flow missing | Solenoid OFF |
| Empty refill reservoir | Refill OFF |
| Emergency stop | Critical outputs OFF |
| Temperature sensor failure | Safe temperature state |
| Power loss | Safe startup after recovery |

---

## 11. Software Architecture

The firmware follows a layered architecture:

```text
USER INTERFACE
    │
    ├── HMI
    └── Web Interface
          │
          ▼
APPLICATION
    │
    ├── Temperature Control
    ├── Refill Control
    ├── Alarm Management
    ├── Operating Modes
    └── Safety
          │
          ▼
SERVICES
    │
    ├── Logging
    ├── RTC
    ├── Configuration
    ├── CSV Export
    └── Event Management
          │
          ▼
DEVICE LAYER
    │
    ├── Sensors
    ├── RS485
    ├── I2C
    ├── 1-Wire
    ├── GPIO
    └── SD
          │
          ▼
HARDWARE
    │
    └── ESP32-S3
```

---

## 12. Firmware Structure

Recommended structure:

```text
firmware/
└── src/
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
    │   ├── state_manager.cpp
    │   └── safety_manager.cpp
    │
    ├── sensors/
    │   ├── temperature_manager.cpp
    │   ├── do_manager.cpp
    │   ├── ph_manager.cpp
    │   ├── level_manager.cpp
    │   └── flow_manager.cpp
    │
    ├── control/
    │   ├── temperature_controller.cpp
    │   ├── heater_controller.cpp
    │   ├── cooling_controller.cpp
    │   ├── refill_controller.cpp
    │   └── equipment_manager.cpp
    │
    ├── safety/
    │   ├── alarm_manager.cpp
    │   └── interlock_manager.cpp
    │
    ├── hardware/
    │   ├── rtc_manager.cpp
    │   ├── sd_manager.cpp
    │   ├── rs485_manager.cpp
    │   └── gpio_manager.cpp
    │
    ├── logging/
    │   ├── data_logger.cpp
    │   └── event_logger.cpp
    │
    ├── web/
    │   ├── web_server.cpp
    │   ├── api.cpp
    │   └── csv_export.cpp
    │
    ├── hmi/
    │   ├── hmi_manager.cpp
    │   └── screens/
    │
    └── utils/
        ├── time_utils.cpp
        ├── validation.cpp
        └── helpers.cpp
```

---

## 13. Operating Modes

The system supports:

### AUTOMATIC

Normal operation.

The system automatically controls:

- Temperature
- Heater
- Peltier cooling
- Refill
- Pumps
- Safety
- Alarms

### MANUAL

Allows controlled testing of equipment.

All safety interlocks remain active.

### MAINTENANCE

Used for:

- Sensor testing
- Pump testing
- Heater testing
- Peltier testing
- Solenoid testing

Critical safety remains active.

### ALARM

Used when a critical fault requires affected equipment to enter a safe state.

---

## 14. Data Logging

Initial logging interval:

**1 minute**

Logged values include:

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

## 15. Event Logging

The system records events such as:

- System startup
- System shutdown
- Heater ON/OFF
- TEC 1 ON/OFF
- TEC 2 ON/OFF
- Refill start
- Refill completion
- Refill failure
- Low water
- High water
- Low flow
- Sensor failure
- Alarm activation
- Alarm clearing
- Emergency stop
- SD failure
- Wi-Fi failure

---

## 16. Data Storage

Suggested structure:

```text
/data/
    sensor/
        YYYY-MM-DD.csv

/events/
    YYYY-MM-DD.csv

/alarms/
    YYYY-MM-DD.csv

/config/
    system.json
```

The exact storage structure may change during implementation.

---

## 17. CSV Export

The system shall provide CSV export through the local web interface.

Users can specify:

```text
Start Date/Time
End Date/Time
```

The system then:

```text
Validate Request
       ↓
Find Relevant Files
       ↓
Read Data in Chunks
       ↓
Filter Date Range
       ↓
Stream CSV
```

The complete dataset shall not be loaded into RAM.

CSV export shall never block:

- Safety monitoring
- Temperature control
- Refill protection
- Alarm processing
- Data logging

---

## 18. TinyRTC

TinyRTC provides persistent timekeeping.

It is used for:

- Sensor timestamps
- Event timestamps
- Alarm timestamps
- CSV timestamps
- Historical records

When Wi-Fi is available:

```text
NTP
 ↓
ESP32-S3
 ↓
TinyRTC
```

When Wi-Fi is unavailable, TinyRTC continues to provide time.

---

## 19. HMI

The planned HMI is a 7-inch touchscreen.

Main screens:

1. Dashboard
2. Water Quality
3. Temperature
4. Equipment
5. Historical Data
6. Alarm History
7. Settings
8. Manual Control
9. System Information

The HMI shall not directly manipulate GPIO.

All commands shall pass through the application and safety layers.

---

## 20. Web Interface

The system shall provide a local web interface over Wi-Fi.

Planned functionality:

- Live dashboard
- Sensor monitoring
- Equipment status
- Alarm history
- Event history
- Historical data
- Configuration
- Manual control
- CSV export

The system does not require cloud connectivity for normal operation.

---

## 21. Network Behavior

Wi-Fi is not required for core operation.

If Wi-Fi fails:

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
```

The system shall automatically attempt to reconnect.

---

## 22. Power Recovery

After power interruption:

```text
Power Restored
      ↓
ESP32 Boot
      ↓
Initialize Hardware
      ↓
Outputs Safe
      ↓
Read Configuration
      ↓
Read TinyRTC
      ↓
Initialize Sensors
      ↓
Validate Safety Inputs
      ↓
Start Services
      ↓
Resume Automatic Operation
```

The following shall remain OFF during unsafe startup:

- Heater
- TEC 1
- TEC 2
- Refill solenoid

The final pump and aeration startup behavior shall be defined by the safety configuration.

---

## 23. Project Documentation

The project documentation is divided into:

| Document | Purpose |
|---|---|
| `01-product-requirement.md` | Product requirements and system goals |
| `02-technical-requirement.md` | Technical specifications and requirements |
| `03-system-architecture.md` | Overall hardware and software architecture |
| `04-hardware-design.md` | Hardware design and component architecture |
| `05-software-design.md` | Firmware architecture and software design |
| `06-implementation-plan.md` | Build, integration, testing, and commissioning plan |
| `README.md` | Project overview and entry point |

---

## 24. Implementation Phases

The project follows this sequence:

```text
Phase 0
Requirements
    ↓
Phase 1
Hardware Procurement
    ↓
Phase 2
Development Environment
    ↓
Phase 3
Controller Bring-Up
    ↓
Phase 4
Sensor Integration
    ↓
Phase 5
Actuator Integration
    ↓
Phase 6
Safety System
    ↓
Phase 7
Temperature Control
    ↓
Phase 8
Peltier Cooling
    ↓
Phase 9
Automatic Refill
    ↓
Phase 10
RTC + Data Logging
    ↓
Phase 11
HMI
    ↓
Phase 12
Web Interface
    ↓
Phase 13
Full Integration
    ↓
Phase 14
Dry Testing
    ↓
Phase 15
Water Testing
    ↓
Phase 16
Thermal Testing
    ↓
Phase 17
Reliability Testing
    ↓
Phase 18
Biological Commissioning
    ↓
Phase 19
Version 1 Release
```

---

## 25. Testing Strategy

Testing is performed at multiple levels.

### Unit Testing

Examples:

- Sensor parsing
- Sensor validation
- State machines
- Alarm logic
- Configuration validation
- CSV generation

### Integration Testing

Examples:

- Sensor → Controller
- Controller → Equipment
- RTC → Logger
- SD → CSV
- HMI → Control
- Web → Control

### System Testing

Complete incubator operation.

### Fault Testing

Examples:

- Sensor disconnect
- No flow
- Low water
- High water
- Refill failure
- Peltier overtemperature
- SD failure
- Wi-Fi failure
- Power interruption

---

## 26. Reliability Testing

Before biological operation, the complete system shall undergo continuous testing.

Minimum:

**72 hours**

Preferred:

**7 days**

The test shall monitor:

- Temperature
- DO
- pH
- Level
- Flow
- Heater
- TECs
- Pumps
- Aeration
- Refill
- Logging
- RTC
- HMI
- Wi-Fi
- Alarms

---

## 27. Biological Commissioning

The system shall not be considered ready for biological operation until:

- Temperature control is stable.
- DO monitoring is stable.
- pH monitoring is stable.
- Circulation is stable.
- Aeration is stable.
- Refill is reliable.
- Cooling is validated.
- Heating is validated.
- Safety systems pass testing.
- Data logging works.
- Power recovery is verified.
- Long-duration testing passes.

Species-specific operating parameters shall be configured before biological commissioning.

---

## 28. Current Design Constraints

### Tank

Approximately:

**227 L / 60 US gallons**

### Controller

**ESP32-S3**

### Temperature

**2 × DS18B20**

### DO

**Industrial/laboratory DO probe + RS485 Modbus transmitter**

### pH

**Industrial/laboratory pH probe + RS485 Modbus transmitter**

### Cooling

**Exactly 2 × TEC1-12706**

### RTC

**TinyRTC**

### Storage

**microSD**

### Refill

**Automatic refill using normally closed solenoid**

### Circulation

**Continuous-duty circulation pump**

### Aeration

**Independent air pump and air stones**

### HMI

**7-inch touchscreen**

### Network

**Local Wi-Fi**

### Cloud

**Not required for Version 1**

---

## 29. Known Engineering Risks

### Peltier Cooling Capacity

Two TEC1-12706 modules may not have sufficient capacity to cool approximately 227 L under high ambient temperature.

This must be experimentally validated.

### Peltier Heat Rejection

Poor hot-side cooling can significantly reduce cooling performance.

The heatsink and airflow are therefore critical.

### Water and Electrical Safety

The system combines:

- Water
- Pumps
- Heater
- High-current TECs
- AC power

Electrical protection and physical separation are mandatory.

### Sensor Reliability

DO and pH probes require proper calibration and maintenance.

### Flow Reliability

Flow sensors and plumbing must be selected according to the actual pump and piping.

### Refill Reliability

The system must detect:

- Empty reservoir
- Blocked line
- Failed solenoid
- No flow
- High tank level

---

## 30. Future Expansion

Potential future features:

- Automatic pH dosing
- Automatic DO control
- Variable aeration
- Feeding automation
- Camera monitoring
- Multiple tanks
- Batch tracking
- Cloud synchronization
- Remote notifications
- Mobile application
- Additional water-quality sensors
- Additional incubation profiles
- Automated water-change system

These are outside the primary Version 1 scope.

---

## 31. Version 1 Definition of Done

Version 1 is complete when:

- [ ] ESP32-S3 controller operational
- [ ] 7-inch HMI operational
- [ ] Two DS18B20 sensors operational
- [ ] DO monitoring operational
- [ ] pH monitoring operational
- [ ] Main level monitoring operational
- [ ] Independent low/high level protection operational
- [ ] Refill reservoir protection operational
- [ ] Main flow monitoring operational
- [ ] Refill flow monitoring operational
- [ ] Circulation pump operational
- [ ] Aeration operational
- [ ] Heater operational
- [ ] Heater independent protection operational
- [ ] TEC 1 operational
- [ ] TEC 2 operational
- [ ] Peltier fans operational
- [ ] Peltier thermal protection operational
- [ ] Automatic refill operational
- [ ] TinyRTC operational
- [ ] microSD logging operational
- [ ] Event logging operational
- [ ] Alarm logging operational
- [ ] CSV export operational
- [ ] Local web interface operational
- [ ] Manual control operational
- [ ] Safety interlocks validated
- [ ] Power recovery validated
- [ ] Wi-Fi failure validated
- [ ] SD failure validated
- [ ] Thermal testing passed
- [ ] Leak testing passed
- [ ] 72-hour reliability test passed
- [ ] Biological commissioning completed

---

## 32. Repository Structure

Recommended final repository:

```text
crayfish-incubator/
│
├── README.md
│
├── docs/
│   ├── 01-product-requirement.md
│   ├── 02-technical-requirement.md
│   ├── 03-system-architecture.md
│   ├── 04-hardware-design.md
│   ├── 05-software-design.md
│   └── 06-implementation-plan.md
│
├── firmware/
│   ├── platformio.ini
│   ├── src/
│   ├── include/
│   └── test/
│
├── hardware/
│   ├── schematics/
│   ├── pcb/
│   ├── wiring/
│   ├── enclosure/
│   └── bom/
│
├── test/
│   ├── controller/
│   ├── sensors/
│   ├── actuators/
│   ├── safety/
│   ├── thermal/
│   ├── refill/
│   └── system/
│
└── tools/
```

---

## 33. Project Status

Current project status:

**Architecture and design phase**

Completed:

- [x] Product requirements
- [x] Technical requirements
- [x] System architecture
- [x] Hardware design
- [x] Software design
- [x] Implementation plan
- [x] TinyRTC requirement
- [x] CSV export requirement
- [x] Automatic refill requirement
- [x] Dual DS18B20 temperature monitoring
- [x] RS485 DO monitoring
- [x] RS485 pH monitoring
- [x] Dual TEC1-12706 cooling architecture
- [x] Independent water-level protection
- [x] Flow-based heater/cooling interlock
- [x] Local HMI architecture
- [x] Local web architecture
- [x] Safety architecture

Next major activities:

- [ ] Confirm crayfish species
- [ ] Finalize biological operating limits
- [ ] Select final DO sensor
- [ ] Select final pH sensor
- [ ] Select level sensors
- [ ] Select flow sensors
- [ ] Size circulation pump
- [ ] Select heater
- [ ] Select TEC drivers
- [ ] Size TEC power supply
- [ ] Select heatsink and fans
- [ ] Finalize plumbing
- [ ] Finalize electrical architecture
- [ ] Create wiring diagram
- [ ] Create BOM
- [ ] Begin hardware procurement
- [ ] Begin firmware implementation

---

## 34. Project Principle

The Crayfish Incubator shall prioritize:

```text
SAFETY
   ↓
RELIABILITY
   ↓
WATER QUALITY
   ↓
TEMPERATURE STABILITY
   ↓
DATA INTEGRITY
   ↓
AUTOMATION
   ↓
USER CONVENIENCE
```

The system shall remain operational and safe even when non-critical services such as Wi-Fi or microSD logging fail.

The goal of Version 1 is not simply to automate equipment.

The goal is to create a **reliable, measurable, testable, and fail-safe incubation environment** that can operate continuously with minimal manual intervention.