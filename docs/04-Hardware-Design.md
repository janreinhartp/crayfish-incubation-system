# 04 Hardware Design

## 1. Purpose

This document defines the hardware architecture for the Crayfish Incubator Version 1.

The system shall continuously monitor and control:

- Water temperature
- Dissolved oxygen
- pH
- Main tank water level
- Refill reservoir level
- Main circulation flow
- Refill flow
- Peltier cooling temperature
- Ambient temperature
- Heater
- Peltier cooling
- Water circulation
- Aeration
- Automatic water refill

The system shall operate continuously with local touchscreen control, local web monitoring, data logging, and hardware safety protection.

---

# 2. System Overview

The system is based on an ESP32-S3 controller.

```text
                         ┌─────────────────────┐
                         │     7" Touch HMI     │
                         │     ESP32-S3 LCD     │
                         └──────────┬──────────┘
                                    │
                                    │
                         ┌──────────▼──────────┐
                         │      ESP32-S3       │
                         │   Main Controller   │
                         └─────┬───┬───┬──────┘
                               │   │   │
              ┌────────────────┘   │   └─────────────────┐
              │                    │                     │
              ▼                    ▼                     ▼
        ┌───────────┐        ┌───────────┐        ┌───────────┐
        │  Sensors  │        │   RTC +   │        │ microSD   │
        │           │        │   Storage │        │ Logging   │
        └─────┬─────┘        └───────────┘        └───────────┘
              │
      ┌───────┼────────────────────────────┐
      │       │             │              │
      ▼       ▼             ▼              ▼
   Temp      RS485       Level           Flow
  DS18B20   DO / pH      Sensors        Sensors

                         ┌─────────────────────┐
                         │ Equipment Control   │
                         └──────────┬──────────┘
                                    │
        ┌───────────────┬───────────┼──────────────┬──────────────┐
        ▼               ▼           ▼              ▼              ▼
     Heater          TEC 1       TEC 2        Circulation      Aeration
                                              Pump             Pump

                                    │
                                    ▼
                              Refill Solenoid
```

---

# 3. Main Tank

Target tank capacity:

- 60 US gallons
- Approximately 227 liters

The tank shall provide sufficient volume for the intended crayfish incubation experiment.

The tank shall include provisions for:

- Water circulation
- Aeration
- Temperature sensing
- DO measurement
- pH measurement
- Water-level monitoring
- Water return
- Water refill
- Drainage/maintenance access

Sensor placement shall avoid direct interference from strong water jets or air bubbles.

---

# 4. Controller

## 4.1 ESP32-S3

The ESP32-S3 shall be the primary control processor.

Responsibilities:

- Sensor acquisition
- Control logic
- Safety monitoring
- Equipment control
- HMI communication
- Web server
- Data logging
- RTC management
- Alarm management
- Configuration
- CSV export

The ESP32-S3 shall not directly drive high-power loads.

All high-power equipment shall use appropriate:

- Relays
- SSRs
- MOSFET drivers
- Motor drivers
- Isolated interfaces

as required.

---

# 5. HMI

Recommended HMI platform:

**Waveshare ESP32-S3 Touch LCD 7B**

Target characteristics:

- 7-inch touchscreen
- ESP32-S3
- LVGL-compatible
- Local user interface
- Wi-Fi capability
- SD card support

The HMI shall provide:

- Dashboard
- Temperature display
- DO display
- pH display
- Water-level display
- Flow display
- Equipment status
- Alarm display
- Historical data
- Settings
- Manual control
- System information

The HMI shall remain usable even when Wi-Fi is unavailable.

---

# 6. Temperature Sensors

## 6.1 Main Water Temperature

Use:

**2 × waterproof DS18B20**

Configuration:

```text
T1 = Primary temperature sensor
T2 = Independent verification sensor
```

Both sensors shall be installed in the main tank.

The controller shall compare the two measurements.

Example:

```text
T1 = 27.1 °C
T2 = 27.3 °C

Difference = 0.2 °C
Status = VALID
```

A configurable maximum difference shall be used to detect sensor disagreement.

---

# 7. Dissolved Oxygen Sensor

The DO measurement shall use a laboratory or industrial-grade dissolved oxygen probe and transmitter.

Preferred interface:

**RS485 Modbus RTU**

Architecture:

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

The transmitter should provide:

- DO concentration
- Optional saturation percentage
- Optional temperature compensation
- Sensor status
- Modbus communication

The exact probe/transmitter shall be selected based on:

- Required accuracy
- Freshwater compatibility
- Measurement range
- Calibration requirements
- Maintenance requirements
- Budget
- Modbus support

---

# 8. pH Sensor

The pH system shall use a replaceable laboratory or industrial pH probe.

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

The system shall support:

- pH measurement
- Calibration
- Sensor replacement
- Communication monitoring
- Sensor fault detection

The pH probe shall be installed where water movement is sufficient but excessive turbulence and air bubbles are avoided.

---

# 9. RS485 Interface

The ESP32-S3 shall communicate with:

- DO transmitter
- pH transmitter

through RS485.

Recommended architecture:

```text
ESP32-S3
    │
    ▼
Isolated RS485 Interface
    │
    ├──── DO Transmitter
    │
    └──── pH Transmitter
```

The RS485 bus shall include appropriate:

- Termination
- Biasing where required
- Surge/transient protection
- Grounding strategy
- Cable shielding where appropriate

Device addresses shall be unique.

Example:

```text
Address 01 = DO
Address 02 = pH
```

---

# 10. Main Tank Level Measurement

The main tank shall use two forms of level protection.

## 10.1 Continuous Level Sensor

Provides the controller with an approximate water-level percentage.

Example:

```text
0%     Empty
25%    Low
50%    Normal
75%    Normal
100%   Full
```

The exact sensor technology shall be selected based on:

- Tank geometry
- Required accuracy
- Water chemistry
- Installation method
- Cost

## 10.2 Independent Float Switches

At minimum:

```text
LOW FLOAT
HIGH FLOAT
```

The float switches provide independent hardware-level protection.

They shall not depend on the continuous level sensor.

---

# 11. Main Tank Level Safety

The system shall use the following hierarchy:

```text
Continuous Level Sensor
        │
        ▼
Normal Level Monitoring

Low Float
        │
        ▼
Low-Level Protection

High Float
        │
        ▼
Refill Protection

Emergency High-Level Condition
        │
        ▼
Immediate Refill Shutdown
```

Critical level protection shall remain functional even if the continuous level sensor fails.

---

# 12. Refill Reservoir

The refill reservoir shall be physically separate from the main tank.

The reservoir shall include:

- Water storage
- Low-level protection
- Manual isolation valve
- Optional filter
- Refill outlet

Recommended arrangement:

```text
Refill Reservoir
       │
       ▼
Manual Valve
       │
       ▼
Filter
       │
       ▼
Normally Closed Solenoid
       │
       ▼
Flow Sensor
       │
       ▼
Main Tank
```

The reservoir should be elevated where practical to reduce pump requirements.

---

# 13. Refill Solenoid

Use a **normally closed solenoid valve**.

Normal condition:

```text
Power OFF → Valve CLOSED
```

The valve shall only open when the controller confirms that:

- Main tank level is low
- Refill reservoir has sufficient water
- No high-level condition exists
- No emergency stop is active
- System is not in a refill lockout
- Refill conditions are valid

Loss of controller power shall close the valve.

---

# 14. Refill Flow Sensor

A separate flow sensor shall monitor refill water.

Purpose:

- Confirm water is actually entering the tank
- Detect blocked piping
- Detect failed solenoid
- Detect empty reservoir
- Detect abnormal refill rate

Example:

```text
Solenoid ON
     │
     ▼
Expected Flow
     │
     ├── YES → Continue
     │
     └── NO → Timeout → Close Valve → Alarm
```

---

# 15. Circulation Pump

The main circulation pump shall operate continuously during normal operation.

Initial target:

**Approximately 1,000 to 2,000 L/h**

Final pump selection shall be based on:

- Actual plumbing
- Head height
- Pipe diameter
- Filter restriction
- Peltier water block restriction
- Required turnover rate

The pump shall be rated for continuous operation.

---

# 16. Main Water Flow Sensor

An inline flow sensor shall monitor the circulation loop.

Recommended location:

```text
Tank
 ↓
Filter
 ↓
Circulation Pump
 ↓
Flow Sensor
 ↓
Peltier Water Block
 ↓
Return
```

The controller shall use the flow sensor as a safety interlock.

If circulation flow is insufficient:

```text
Heater → OFF
TEC 1 → OFF
TEC 2 → OFF
Alarm → ON
```

The circulation pump itself may remain commanded ON unless a separate fault requires shutdown.

---

# 17. Water Circulation Loop

Initial architecture:

```text
              ┌──────────────────────┐
              │      Main Tank       │
              │                      │
              │  Temperature        │
              │  DO                 │
              │  pH                 │
              │  Level              │
              └──────────┬───────────┘
                         │
                         ▼
                  Coarse Filtration
                         │
                         ▼
                  Circulation Pump
                         │
                         ▼
                    Flow Sensor
                         │
                         ▼
                  Peltier Water Block
                         │
                         ▼
                    Fine Filtration
                    if required
                         │
                         ▼
                    Tank Return
```

The final filter arrangement shall be validated during physical assembly.

---

# 18. Heater

Initial heater target:

**Approximately 300 to 500 W**

The final rating shall depend on:

- Room temperature
- Tank insulation
- Target temperature
- Heat loss
- Heating time requirement

The heater shall be aquarium/water-system rated and suitable for continuous operation.

---

# 19. Independent Heater Thermostat

The heater shall have an independent thermostat or thermal protection mechanism.

This is separate from ESP32 software control.

Architecture:

```text
ESP32
  │
  ▼
Isolated Heater Control
  │
  ▼
Independent Thermostat
  │
  ▼
Heater
```

This provides protection against software failure.

---

# 20. Heater Safety

The heater shall only operate when:

- Water level is safe
- Main circulation is sufficient
- Temperature sensors are valid
- No emergency stop is active
- No critical alarm prevents heating

The heater shall immediately turn OFF if:

- Low water is detected
- Flow is insufficient
- Temperature sensor fails
- Emergency stop activates
- Critical system fault occurs

---

# 21. Peltier Cooling System

Version 1 shall use exactly:

**2 × TEC1-12706 Peltier modules**

The system shall use water cooling on the cold side.

Architecture:

```text
Water Loop
    │
    ▼
Cold-Side Water Block
    │
    ├── TEC 1
    │
    └── TEC 2

TEC Hot Side
    │
    ▼
Large Heatsink
    │
    ▼
High-Airflow Fans
```

---

# 22. Peltier Water Block

The Peltier cooling system shall use a water block between the tank water loop and the TEC modules.

The water block should provide:

- Good thermal conductivity
- Adequate flow capacity
- Good contact with TEC modules
- Corrosion-resistant wetted surfaces
- Leak-resistant fittings

The TEC modules shall not be directly exposed to tank water.

---

# 23. Peltier Hot-Side Cooling

The hot side is critical.

The system shall include:

- Large heatsink
- High-airflow fans
- Thermal interface material
- Mechanical clamping

Target architecture:

```text
TEC
 │
 ▼
Thermal Interface
 │
 ▼
Large Heatsink
 │
 ├── Fan 1
 ├── Fan 2
 └── Additional airflow if required
```

Hot-side temperature shall be monitored.

---

# 24. Peltier Electrical Supply

TEC1-12706 modules are nominally 12 V-class devices.

They shall **not** be connected directly to a 24 V supply.

If a 24 V main power architecture is used, the Peltier subsystem shall use an appropriate power conversion or dedicated supply.

Possible architecture:

```text
24 V DC Main Supply
       │
       ├── DC/DC or dedicated 12 V supply
       │              │
       │              ├── TEC 1 Driver
       │              │
       │              └── TEC 2 Driver
       │
       └── Other 24 V Loads
```

The final TEC driver shall be selected based on actual TEC current requirements.

---

# 25. Independent TEC Control

TEC 1 and TEC 2 shall have independent control channels.

```text
ESP32
 │
 ├── TEC 1 Driver ─── TEC 1
 │
 └── TEC 2 Driver ─── TEC 2
```

This allows:

- Staged cooling
- Individual fault detection
- Reduced power consumption
- Thermal testing
- Independent maintenance

---

# 26. Peltier Cooling Fans

Cooling fans shall operate whenever the hot side requires cooling.

The controller shall be able to keep fans operating after TEC shutdown.

Example:

```text
TEC OFF
    │
    ▼
Hot Side Still Hot?
    │
   YES
    │
    ▼
Fans ON
    │
    ▼
Hot Side Safe
    │
    ▼
Fans OFF
```

---

# 27. Peltier Thermal Protection

A dedicated temperature sensor shall monitor the Peltier heatsink.

If the heatsink exceeds the configured safety limit:

```text
TEC 1 → OFF
TEC 2 → OFF
Fans → ON
Alarm → ON
```

The cooling system shall remain disabled until the fault is cleared.

---

# 28. Peltier Performance Testing

The two TEC modules are considered a prototype cooling system.

Two TEC1-12706 modules may not provide enough cooling capacity for approximately 227 L of water under high ambient conditions.

Before final operation, the system shall undergo thermal testing.

Minimum test:

**24 to 72 hours**

Testing shall include:

- Ambient temperature
- Initial water temperature
- Minimum water temperature
- TEC power
- TEC 1 operation
- TEC 2 operation
- Hot-side temperature
- Cooling fan operation
- Water flow
- Temperature recovery

Testing should include the hottest expected operating conditions.

---

# 29. Aeration System

Aeration shall be independent from the circulation system.

Components:

- Air pump
- Air manifold
- Multiple air stones
- Air tubing
- Check valves

Architecture:

```text
Air Pump
   │
   ▼
Manifold
 ┌─┼─┬─┐
 ▼ ▼ ▼ ▼
Air Stones
```

The aeration system shall normally remain ON.

---

# 30. Aeration Safety

The aeration system should remain independent from:

- Main circulation pump
- Heater
- Peltier cooling

This ensures oxygenation continues if the circulation system experiences a fault.

---

# 31. Electrical Power Architecture

Recommended high-level architecture:

```text
                  AC MAINS
                     │
              ┌──────▼──────┐
              │ Main Breaker│
              └──────┬──────┘
                     │
                  RCD/GFCI
                     │
             ┌───────┴────────┐
             │                │
             ▼                ▼
       AC High Power      DC Power Supply
       Equipment          │
             │            ├── 24 V DC
             │            ├── 12 V DC
             │            └── 5 V DC
             │
       ┌─────┼─────┐
       ▼     ▼     ▼
    Heater  PSU   Other AC
```

Exact voltages shall be finalized after component selection.

---

# 32. DC Power Domains

The system should separate power domains where practical.

Recommended:

```text
24 V DC
 ├── Solenoid
 ├── Fans
 └── Other 24 V equipment

12 V DC
 ├── TEC subsystem
 ├── Pump if required
 └── Other 12 V equipment

5 V / 3.3 V
 ├── ESP32-S3
 ├── Sensors
 ├── RTC
 └── Logic
```

The actual pump and fan voltage shall determine the final architecture.

---

# 33. Power Supply Sizing

Power supplies shall be sized using:

```text
Total Continuous Load
+
Startup Current
+
Safety Margin
```

Recommended design margin:

**At least 20 to 30%**

High-current equipment shall not share undersized wiring or connectors.

---

# 34. Fusing

Individual branches shall have appropriate fuses or circuit protection.

At minimum:

```text
Main Supply
 ├── Controller Fuse
 ├── Sensor/Logic Fuse
 ├── Pump Fuse
 ├── TEC 1 Fuse
 ├── TEC 2 Fuse
 ├── Fan Fuse
 ├── Solenoid Fuse
 └── Other Load Protection
```

Fuse ratings shall be selected from measured or specified load current.

---

# 35. Emergency Stop

An emergency stop shall be provided.

Emergency stop shall disable critical equipment.

Minimum response:

```text
Emergency Stop
      │
      ├── Heater OFF
      ├── TEC 1 OFF
      ├── TEC 2 OFF
      └── Refill OFF
```

The exact treatment of pumps and aeration shall be determined during the final safety review.

---

# 36. Hardware Safety Inputs

Safety inputs shall include:

- Main tank low float
- Main tank high float
- Refill reservoir low-level sensor
- Emergency stop
- Peltier heatsink temperature
- Main flow status

Safety inputs shall be designed so that a disconnected or invalid signal can be detected where practical.

---

# 37. TinyRTC

The system shall include a TinyRTC module.

Purpose:

- Persistent date/time
- Event timestamps
- Sensor log timestamps
- Alarm timestamps
- CSV data timestamps
- Power-loss time retention

Architecture:

```text
ESP32-S3
    │
    │ I2C
    ▼
 TinyRTC
```

The RTC shall provide time even when Wi-Fi is unavailable.

When Wi-Fi is available, NTP may be used to synchronize the RTC.

---

# 38. microSD Storage

microSD storage shall be used for:

- Sensor history
- Alarm history
- Event history
- Configuration backup where appropriate
- CSV export source data

Suggested structure:

```text
/data/
    sensor/
    events/
    alarms/

/config/
    system.json
```

The exact file structure may change during implementation.

---

# 39. Data Logging Hardware Requirements

The storage subsystem shall support continuous operation.

Requirements:

- Industrial or high-endurance microSD preferred
- Proper filesystem handling
- Write error detection
- Safe file closing
- Periodic flushing
- File rotation

Storage failure shall not stop the incubator's control system.

---

# 40. Flow Sensor Hardware

Two flow measurements are recommended.

### Main Flow

Measures:

```text
Main Tank → Circulation Loop
```

### Refill Flow

Measures:

```text
Refill Reservoir → Main Tank
```

The sensors shall be selected based on:

- Actual pipe diameter
- Expected flow rate
- Water compatibility
- Pressure drop
- Output interface
- ESP32 compatibility

---

# 41. Sensor Interface Summary

| Sensor | Quantity | Interface | Purpose |
|---|---:|---|---|
| DS18B20 | 2 | 1-Wire | Main water temperature |
| DO Probe + Transmitter | 1 | RS485 Modbus | Dissolved oxygen |
| pH Probe + Transmitter | 1 | RS485 Modbus | pH |
| Main Level Sensor | 1 | Analog/industrial interface | Continuous tank level |
| Main Low Float | 1 | Digital | Low-water protection |
| Main High Float | 1 | Digital | High-water protection |
| Refill Low Sensor | 1 | Digital/level | Reservoir protection |
| Main Flow Sensor | 1 | Pulse/flow output | Circulation monitoring |
| Refill Flow Sensor | 1 | Pulse/flow output | Refill verification |
| Peltier Heatsink Sensor | 1 | Digital | Thermal protection |
| Ambient Temperature | 1 | Digital | Environmental monitoring |
| TinyRTC | 1 | I2C | Persistent time |

---

# 42. Actuator Summary

| Equipment | Quantity | Control |
|---|---:|---|
| Heater | 1 | Relay/SSR |
| TEC1-12706 | 2 | Independent high-current drivers |
| Cooling Fans | 2 or more | MOSFET/driver |
| Circulation Pump | 1 | Relay/MOSFET/driver |
| Aeration Pump | 1 | Relay/MOSFET/driver |
| Refill Solenoid | 1 | Relay/MOSFET |
| Emergency Stop | 1 | Safety input/circuit |

Final actuator driver selection depends on voltage and current ratings.

---

# 43. Hardware Interlocks

Software safety shall be supported by hardware protection.

At minimum:

```text
LOW WATER
   ↓
Heater disabled
Cooling disabled

HIGH WATER
   ↓
Refill disabled

EMERGENCY STOP
   ↓
Critical outputs disabled

PELTIER OVER-TEMP
   ↓
TEC disabled

NO FLOW
   ↓
Heater disabled
Cooling disabled
```

Where practical, critical protection should not rely solely on software.

---

# 44. Wiring Architecture

Low-voltage sensor wiring shall be separated from high-current power wiring.

Recommended physical separation:

```text
CONTROL AREA
├── ESP32-S3
├── RS485
├── RTC
├── Sensor Interfaces
└── SD

POWER AREA
├── AC Protection
├── DC Power Supplies
├── Heater Control
├── TEC Drivers
└── Pump Drivers
```

High-current wiring shall not run parallel with sensitive analog or communication wiring where avoidable.

---

# 45. Grounding

The system shall use a deliberate grounding strategy.

Consider:

- DC negative distribution
- Protective earth
- Shield connections
- RS485 reference
- Metal enclosure grounding
- Heater grounding
- Pump grounding

Protective earth shall not be substituted with signal ground.

The final grounding design shall be reviewed before mains power is connected.

---

# 46. Enclosure

The electronics shall be housed in a suitable enclosure.

The enclosure shall provide:

- Protection against water splash
- Cable management
- Ventilation for heat-producing components
- Fuse access
- Emergency-stop access
- Service access
- Separation between mains and low-voltage wiring

Power electronics producing significant heat shall have adequate ventilation.

---

# 47. Waterproofing

The electronics enclosure shall be protected from:

- Water splash
- Condensation
- Humidity
- Accidental spills

Cable entries should use appropriate:

- Cable glands
- Strain relief
- Waterproof connectors where needed

Sensors and water-side components shall use appropriate waterproof connections.

---

# 48. Electrical Safety

Because the system combines water and electrical power:

- Use RCD/GFCI protection.
- Properly ground applicable equipment.
- Use appropriate circuit breakers.
- Fuse DC branches.
- Use waterproof cable routing near the tank.
- Keep mains wiring physically separated from low-voltage wiring.
- Do not place exposed mains connections near the tank.
- Use appropriately rated enclosures.
- Use strain relief.
- Never rely on software as the only protection against electrical hazards.

---

# 49. Hardware Communication Architecture

```text
                         ESP32-S3
                            │
        ┌───────────────────┼────────────────────┐
        │                   │                    │
        ▼                   ▼                    ▼
      I2C                 1-Wire                RS485
        │                   │                    │
        ▼                   ▼              ┌─────┴─────┐
     TinyRTC             DS18B20           DO          pH
                            │
                            │
                            ▼
                     Temperature State
```

Additional interfaces:

```text
ESP32-S3
 ├── Digital Inputs → Float switches
 ├── Pulse Inputs → Flow sensors
 ├── Analog/Interface → Level sensor if applicable
 ├── Driver Outputs → Pumps/solenoid
 ├── High-current drivers → TECs
 ├── Relay/SSR → Heater
 └── SPI/SD → Data storage
```

---

# 50. Initial Hardware Bill of Materials

| Category | Component | Qty |
|---|---|---:|
| Controller | ESP32-S3 | 1 |
| HMI | 7-inch touchscreen | 1 |
| Temperature | Waterproof DS18B20 | 2 |
| RTC | TinyRTC | 1 |
| Storage | microSD card | 1 |
| DO | DO probe + transmitter | 1 |
| pH | pH probe + transmitter | 1 |
| RS485 | Isolated RS485 interface | 1 |
| Level | Continuous level sensor | 1 |
| Level Safety | Low float switch | 1 |
| Level Safety | High float switch | 1 |
| Reservoir | Low-level sensor | 1 |
| Flow | Main flow sensor | 1 |
| Flow | Refill flow sensor | 1 |
| Heater | Aquarium heater | 1 |
| Cooling | TEC1-12706 | 2 |
| Cooling | Water block | 1 |
| Cooling | Large heatsink | 1 |
| Cooling | High-airflow fans | 2+ |
| Pump | Circulation pump | 1 |
| Aeration | Air pump | 1 |
| Aeration | Air manifold | 1 |
| Aeration | Air stones | Multiple |
| Refill | NC solenoid valve | 1 |
| Refill | Manual valve | 1 |
| Power | DC power supplies | As required |
| Control | Relay/SSR modules | As required |
| Control | MOSFET/TEC drivers | As required |
| Safety | Emergency stop | 1 |
| Safety | RCD/GFCI | 1 |
| Safety | Breakers/fuses | As required |
| Enclosure | Electrical enclosure | 1 |

---

# 51. Hardware Selection Still Required

The following components require final selection and sizing:

1. Exact DO probe/transmitter.
2. Exact pH probe/transmitter.
3. Continuous tank level sensor.
4. Refill reservoir level sensor.
5. Main flow sensor.
6. Refill flow sensor.
7. Circulation pump.
8. Heater.
9. TEC drivers.
10. 12 V TEC power supply or DC/DC converter.
11. Cooling fans.
12. Peltier water block.
13. Main DC power supply.
14. Relay/SSR ratings.
15. Enclosure.
16. Wiring and connectors.
17. RCD/GFCI and circuit protection.

These selections shall be finalized before electrical and mechanical fabrication.

---

# 52. Hardware Bring-Up Order

Hardware integration should follow this order:

```text
1. ESP32-S3
      ↓
2. HMI
      ↓
3. TinyRTC
      ↓
4. microSD
      ↓
5. DS18B20
      ↓
6. RS485
      ↓
7. DO
      ↓
8. pH
      ↓
9. Level Sensors
      ↓
10. Flow Sensors
      ↓
11. Circulation Pump
      ↓
12. Aeration
      ↓
13. Refill System
      ↓
14. Heater
      ↓
15. TEC Cooling
      ↓
16. Full Safety System
```

High-power equipment shall be integrated only after the low-voltage control system is verified.

---

# 53. Hardware Testing

Each subsystem shall be tested independently.

## Controller

- Boot
- GPIO
- Communication
- Watchdog
- Recovery

## Sensors

- Normal readings
- Disconnection
- Invalid readings
- Communication loss

## Pumps

- Start
- Stop
- Continuous operation
- Current draw

## Heater

- ON/OFF
- Independent thermostat
- Low-water protection
- Flow interlock

## Peltier

- TEC 1
- TEC 2
- Fans
- Hot-side temperature
- Thermal shutdown

## Refill

- Valve operation
- Flow detection
- Low reservoir
- High tank level
- Timeout

---

# 54. Thermal Testing

The complete cooling system shall be tested before biological operation.

Test variables:

- Ambient temperature
- Water volume
- Initial water temperature
- Target temperature
- TEC 1 current
- TEC 2 current
- Hot-side temperature
- Water temperature
- Water flow
- Fan speed
- Cooling rate
- Temperature stability

The objective is to determine whether the two TEC1-12706 modules can maintain the required water temperature under realistic environmental conditions.

---

# 55. Final Hardware Safety Verification

Before connecting the biological system:

1. Verify AC protection.
2. Verify RCD/GFCI.
3. Verify grounding.
4. Verify fuses.
5. Verify emergency stop.
6. Verify heater thermostat.
7. Verify low-water protection.
8. Verify high-water refill protection.
9. Verify flow interlock.
10. Verify Peltier thermal shutdown.
11. Verify solenoid fail-closed behavior.
12. Verify power-loss behavior.
13. Verify safe startup.
14. Verify wiring insulation.
15. Perform leak testing.

---

# 56. Hardware Design Acceptance Criteria

The hardware design is considered complete when:

1. ESP32-S3 controller is selected.
2. 7-inch HMI is selected.
3. Two DS18B20 sensors are installed.
4. DO system supports RS485 Modbus.
5. pH system supports RS485 Modbus.
6. Main tank has continuous level monitoring.
7. Main tank has independent low/high protection.
8. Refill reservoir has low-level protection.
9. Main circulation has flow monitoring.
10. Refill has flow monitoring.
11. Circulation pump is sized for the actual plumbing.
12. Heater has independent thermal protection.
13. Exactly two TEC1-12706 modules are provided.
14. TECs have independent control.
15. Peltier hot-side temperature is monitored.
16. Peltier hot-side cooling is adequately sized.
17. Aeration is independent from circulation.
18. Refill uses a normally closed solenoid.
19. TinyRTC provides persistent timekeeping.
20. microSD provides local data storage.
21. CSV export is supported by the storage architecture.
22. Emergency stop is provided.
23. Critical DC branches are fused.
24. AC protection includes appropriate RCD/GFCI and breaker protection.
25. High-power and low-voltage wiring are separated.
26. Water-side electrical connections are protected.
27. Power-loss behavior is safe.
28. Thermal performance is experimentally verified.
29. Leak testing is completed.
30. Full hardware safety testing is completed before biological operation.