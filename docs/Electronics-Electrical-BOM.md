# Crayfish Incubator
## Electronics and Electrical Bill of Materials (BOM) - Philippines

**Project:** Crayfish Incubator  
**Main Tank:** 60 US gallons / approximately 227 L  
**Controller:** ESP32-S3  
**HMI:** 7-inch touchscreen  
**Cooling:** 2 × TEC1-12706 Peltier modules  
**Timekeeping:** TinyRTC / DS3231-class RTC  
**Data Logging:** microSD  
**Communication:** RS485 Modbus RTU, 1-Wire, Wi-Fi  
**Location:** Philippines  
**BOM Scope:** Electronics, electrical power, control, sensors, wiring, protection, and electrical enclosure hardware.

> **Pricing note:** Prices below are estimated Philippine retail/marketplace ranges for budgeting. Actual prices vary by supplier, brand, shipping, stock, and specifications. Confirm datasheets and electrical ratings before purchasing.

---

# 1. System Electrical Architecture

The recommended architecture separates the system into AC mains, high-current DC, and low-voltage control domains.

```text
                    AC 220-240 VAC
                          |
                    Main Isolator
                          |
                     RCD / GFCI
                          |
              +-----------+-----------+
              |                       |
         Heater Branch             DC PSU
              |                       |
        Breaker / Fuse            24 V DC
              |                       |
        SSR / Contactor      +--------+--------+
              |              |        |        |
       Aquarium Heater       |        |        |
                             |        |        |
                         ESP32-S3   Pumps    Fans
                         HMI        Valve
                         Sensors
                         RS485
                         SD
                         RTC
                             |
                       12 V TEC Supply
                             |
                   +---------+---------+
                   |                   |
              TEC Driver #1       TEC Driver #2
                   |                   |
              TEC1-12706 #1      TEC1-12706 #2
```

## Important Peltier Rule

**Never connect the TEC1-12706 modules directly to the 24 V bus.**

TEC1-12706 modules are 12 V-class devices. A typical module can draw approximately 6 A or more at high load.

Two modules can therefore require approximately:

```text
6.4 A × 2 = 12.8 A
```

Use a dedicated 12 V high-current supply or an appropriately designed 24 V-to-12 V power stage.

Each TEC should have:

- Independent driver
- Independent fuse
- Independent control
- Thermal protection
- Current-rated wiring

---

# 2. Controller and HMI

| ID | Component | Recommended Specification | Qty | Estimated Unit | Estimated Total | Priority |
|---|---|---|---:|---:|---:|---|
| CTRL-01 | ESP32-S3 development board | ESP32-S3, preferably 16 MB Flash / 8 MB PSRAM | 1 | ₱500-₱1,200 | ₱500-₱1,200 | Required |
| HMI-01 | 7-inch touchscreen | Waveshare ESP32-S3 Touch LCD 7 | 1 | ₱2,000-₱3,500 | ₱2,000-₱3,500 | Required |
| RTC-01 | RTC module | DS3231 / TinyRTC | 1 | ₱110-₱200 | ₱110-₱200 | Required |
| RTC-02 | CR2032 battery | RTC backup | 1-2 | ₱20-₱50 | ₱20-₱100 | Required |
| SD-01 | microSD card | 16-32 GB reliable brand | 1 | ₱250-₱500 | ₱250-₱500 | Required |
| SD-02 | Spare microSD | Backup | 1 | ₱250-₱500 | ₱250-₱500 | Recommended |
| IO-01 | Relay/driver board | Opto-isolated relay or MOSFET driver | 2-3 | ₱100-₱300 | ₱200-₱900 | Required |
| IO-02 | Logic-level MOSFET modules | DC load switching | 4-6 | ₱50-₱150 | ₱200-₱900 | Recommended |

### Estimated subtotal

**₱3,530-₱7,900**

---

# 3. Temperature Sensors

The system should use two independent water temperature sensors.

```text
T1 = Primary control sensor
T2 = Independent verification sensor
```

| ID | Component | Specification | Qty | Estimated Unit | Estimated Total |
|---|---|---|---:|---:|---:|
| TEMP-01 | Waterproof DS18B20 | Stainless waterproof 1-Wire probe | 2 | ₱50-₱150 | ₱100-₱300 |
| TEMP-02 | Spare DS18B20 | Same type | 1 | ₱50-₱150 | ₱50-₱150 |
| TEMP-03 | 4.7 kΩ resistor | 1-Wire pull-up | 2 | ₱5-₱20 | ₱10-₱40 |
| TEMP-04 | Heatsink temperature sensor | DS18B20 or NTC | 1 | ₱50-₱150 | ₱50-₱150 |
| TEMP-05 | Ambient temperature sensor | Optional | 1 | ₱50-₱150 | ₱50-₱150 |

### Estimated subtotal

**₱260-₱790**

---

# 4. Dissolved Oxygen System

For this project, use a proper DO probe and transmitter rather than a basic hobby analog DO module.

## Recommended interface

**RS485 Modbus RTU**

| ID | Component | Specification | Qty | Estimated Budget |
|---|---|---|---:|---:|
| DO-01 | DO probe | Laboratory/industrial replaceable probe | 1 | ₱5,000-₱15,000 |
| DO-02 | DO transmitter | RS485 Modbus RTU preferred | 1 | ₱3,000-₱10,000 |
| DO-03 | Calibration accessories | Calibration cup, membrane/electrolyte as required | 1 set | ₱1,000-₱3,000 |
| DO-04 | Isolated RS485 interface | 3.3 V compatible | 1 | ₱300-₱1,000 |
| DO-05 | RS485 protection | TVS, termination, connectors | 1 set | ₱200-₱500 |

### Estimated subtotal

**₱9,500-₱29,500**

### Recommendation

Do not make the DO sensor the cheapest part of the system.

DO is one of the most important water-quality measurements for the incubator.

---

# 5. pH Measurement System

| ID | Component | Specification | Qty | Estimated Budget |
|---|---|---|---:|---:|
| PH-01 | pH probe | Replaceable laboratory/industrial electrode | 1 | ₱1,500-₱5,000 |
| PH-02 | pH transmitter | RS485 Modbus RTU preferred | 1 | ₱2,000-₱7,000 |
| PH-03 | Calibration solutions | pH 4.00, 7.00, optionally 10.00 | 1 set | ₱300-₱800 |
| PH-04 | Isolated RS485 interface | Shared or dedicated | 1 | ₱300-₱1,000 |
| PH-05 | Storage solution | pH electrode storage solution | 1 | ₱200-₱500 |

### Estimated subtotal

**₱4,300-₱14,300**

---

# 6. Water Level and Safety Sensors

The main tank should have both continuous level measurement and independent safety switches.

| ID | Component | Specification | Qty | Estimated Unit | Estimated Total |
|---|---|---|---:|---:|---:|
| LEVEL-01 | Main tank level sensor | 0-5 V, 4-20 mA, ultrasonic, pressure, or compatible sensor | 1 | ₱1,000-₱4,000 | ₱1,000-₱4,000 |
| LEVEL-02 | LOW float switch | Independent safety input | 1 | ₱100-₱300 | ₱100-₱300 |
| LEVEL-03 | HIGH float switch | Overflow protection | 1 | ₱100-₱300 | ₱100-₱300 |
| LEVEL-04 | Emergency HIGH float | Additional protection | 1 | ₱100-₱300 | ₱100-₱300 |
| LEVEL-05 | Refill reservoir LOW float | Dry-refill protection | 1 | ₱100-₱300 | ₱100-₱300 |
| LEVEL-06 | Refill reservoir level sensor | Optional continuous measurement | 1 | ₱500-₱2,000 | ₱500-₱2,000 |
| LEVEL-07 | Emergency-stop button | Latching industrial E-stop | 1 | ₱300-₱800 | ₱300-₱800 |

### Estimated subtotal

**₱2,200-₱8,000**

---

# 7. Flow Monitoring

Two independent flow measurements are recommended.

```text
Main tank circulation
        |
        +-- Flow Sensor #1

Refill reservoir
        |
        +-- Flow Sensor #2
```

| ID | Component | Specification | Qty | Estimated Unit | Estimated Total |
|---|---|---|---:|---:|---:|
| FLOW-01 | Main flow sensor | Sized for circulation pump | 1 | ₱300-₱1,000 | ₱300-₱1,000 |
| FLOW-02 | Refill flow sensor | Sized for refill line | 1 | ₱300-₱1,000 | ₱300-₱1,000 |
| FLOW-03 | Spare flow sensor | Replacement | 1 | ₱300-₱1,000 | ₱300-₱1,000 |

### Estimated subtotal

**₱900-₱3,000**

---

# 8. Heating System

The heater must never be controlled directly by an ESP32 GPIO.

| ID | Component | Specification | Qty | Estimated Budget |
|---|---|---|---:|---:|
| HEAT-01 | Aquarium heater | Approximately 300-500 W | 1 | ₱1,000-₱2,500 |
| HEAT-02 | Independent thermostat | Heater-integrated or external | 1 | ₱500-₱2,000 |
| HEAT-03 | AC SSR | Zero-cross, properly rated | 1 | ₱300-₱1,000 |
| HEAT-04 | SSR heatsink | Correctly sized | 1 | ₱150-₱500 |
| HEAT-05 | AC contactor | Optional additional isolation | 1 | ₱500-₱1,500 |
| HEAT-06 | Heater branch breaker/fuse | Correctly sized | 1 | ₱200-₱500 |

### Estimated subtotal

**₱2,650-₱8,000**

### Safety

The independent thermostat should still protect the tank if:

- ESP32 crashes
- Temperature sensor fails
- Relay fails
- Software enters an invalid state
- Communication fails

---

# 9. Peltier Cooling System

Exactly two TEC1-12706 modules are used.

| ID | Component | Specification | Qty | Estimated Unit | Estimated Total |
|---|---|---|---:|---:|---:|
| TEC-01 | TEC1-12706 | 40×40 mm, 12 V-class | 2 | ₱130-₱200 | ₱260-₱400 |
| TEC-02 | TEC driver #1 | High-current TEC-capable driver | 1 | ₱500-₱1,500 | ₱500-₱1,500 |
| TEC-03 | TEC driver #2 | Independent driver | 1 | ₱500-₱1,500 | ₱500-₱1,500 |
| TEC-04 | 12 V high-current PSU | Sized for both TECs | 1 | ₱2,000-₱5,000 | ₱2,000-₱5,000 |
| TEC-05 | TEC fuse #1 | Approximately 8-10 A, final value after testing | 1 | ₱50-₱150 | ₱50-₱150 |
| TEC-06 | TEC fuse #2 | Approximately 8-10 A | 1 | ₱50-₱150 | ₱50-₱150 |
| TEC-07 | Heatsink temperature sensor | DS18B20/NTC | 1 | ₱50-₱150 | ₱50-₱150 |
| TEC-08 | Thermal cutoff | Independent thermal protection | 1 | ₱200-₱600 | ₱200-₱600 |
| TEC-09 | High-current cable | Correct gauge for actual current | 1 lot | ₱300-₱800 | ₱300-₱800 |
| TEC-10 | Thermal compound/pads | Thermal interface material | 1 set | ₱150-₱500 | ₱150-₱500 |

### Estimated subtotal

**₱4,060-₱11,250**

### Critical thermal warning

Two TEC1-12706 modules may be marginal for cooling approximately 227 L of water under hot Philippine ambient conditions.

The prototype should undergo:

1. 24-hour thermal test
2. 72-hour thermal test
3. Preferably 7-day continuous test

Measure:

- Ambient temperature
- Water temperature
- TEC voltage
- TEC current
- Heatsink temperature
- Hot-side temperature
- Cold-side temperature
- Cooling rate
- Temperature stability

Do not place animals in the tank until the thermal system has been validated.

---

# 10. Peltier Cooling Fans

| ID | Component | Specification | Qty | Estimated Budget |
|---|---|---|---:|---:|
| FAN-01 | High-airflow DC fan | 12 V or 24 V based on design | 2-4 | ₱250-₱600 each |
| FAN-02 | Fan MOSFET driver | Logic-level MOSFET | 2 | ₱100-₱300 |
| FAN-03 | Fan fuse | Correct DC rating | 1 | ₱50-₱150 |
| FAN-04 | Fan connectors | Locking/screw terminals | 1 lot | ₱100-₱300 |

### Estimated subtotal

**₱750-₱3,150**

Use at least two fans if required by the heatsink thermal design.

---

# 11. Circulation, Aeration, and Refill Controls

The actual pumps are part of the mechanical/fluid BOM. Their electrical control hardware is included here.

| ID | Component | Specification | Qty | Estimated Budget |
|---|---|---|---:|---:|
| PUMP-CTRL-01 | Circulation pump driver | MOSFET or properly rated relay | 1 | ₱100-₱500 |
| AIR-CTRL-01 | Air pump driver | Relay/MOSFET depending on pump | 1 | ₱100-₱500 |
| REFILL-CTRL-01 | Solenoid driver | MOSFET/relay | 1 | ₱100-₱400 |
| REFILL-CTRL-02 | NC solenoid valve | Normally closed | 1 | ₱300-₱1,000 |
| REFILL-CTRL-03 | Flyback diode | For DC solenoid | 1 | ₱10-₱30 |
| PUMP-CTRL-02 | Pump fuse | Properly sized | 2 | ₱50-₱150 each |

### Estimated subtotal

**₱810-₱2,880**

---

# 12. RS485 Communication

DO and pH should preferably communicate using RS485 Modbus RTU.

| ID | Component | Specification | Qty | Estimated Budget |
|---|---|---|---:|---:|
| RS485-01 | Isolated RS485 transceiver | 3.3 V logic | 1-2 | ₱300-₱1,000 each |
| RS485-02 | Terminal blocks | Pluggable/screw | 2-4 | ₱50-₱150 |
| RS485-03 | Shielded twisted-pair cable | Industrial preferred | 1 lot | ₱300-₱1,000 |
| RS485-04 | 120 Ω termination resistor | Bus termination | 2 | ₱5-₱20 |
| RS485-05 | Bias resistors | Fail-safe bias | 1 set | ₱20-₱50 |
| RS485-06 | RS485 TVS protection | Surge protection | 1 set | ₱50-₱200 |

### Estimated subtotal

**₱725-₱3,420**

---

# 13. Main DC Power System

Recommended architecture:

```text
24 V DC Main Bus
       |
       +-- ESP32/HMI
       +-- Sensors
       +-- Pumps
       +-- Solenoid
       +-- Cooling Fans
       |
       +-- 12 V TEC Power Stage
```

| ID | Component | Specification | Qty | Estimated Budget |
|---|---|---|---:|---:|
| PSU-01 | Main DC power supply | 24 V, approximately 15-20 A | 1 | ₱1,500-₱5,000 |
| PSU-02 | TEC power supply | 12 V high-current | 1 | ₱2,000-₱5,000 |
| PSU-03 | 5 V DC-DC converter | 24 V to 5 V | 1 | ₱100-₱400 |
| PSU-04 | 3.3 V regulator | Only if required | 1 | ₱100-₱300 |
| PSU-05 | DC distribution block | Fused | 1 | ₱300-₱1,000 |
| PSU-06 | Positive/negative bus bars | Proper current rating | 1 set | ₱200-₱800 |
| PSU-07 | DC branch fuses | Individual load protection | 1 lot | ₱300-₱800 |
| PSU-08 | Spare fuses | Replacement stock | 1 lot | ₱100-₱300 |

### Estimated subtotal

**₱4,600-₱13,600**

---

# 14. AC Mains Protection

Because the equipment operates around water, AC protection is a critical part of the design.

| ID | Component | Specification | Qty | Estimated Budget |
|---|---|---|---:|---:|
| AC-01 | Main breaker | 2-pole, properly rated | 1 | ₱500-₱1,500 |
| AC-02 | RCD/GFCI | 30 mA personnel protection | 1 | ₱1,000-₱3,000 |
| AC-03 | Main isolator | Lockable preferred | 1 | ₱500-₱1,500 |
| AC-04 | AC contactor | High-power branch isolation | 1 | ₱500-₱1,500 |
| AC-05 | Heater breaker/fuse | Properly sized | 1 | ₱200-₱500 |
| AC-06 | AC terminal blocks | Finger-safe preferred | 1 lot | ₱300-₱800 |
| AC-07 | PE/ground terminal | DIN rail | 1 | ₱100-₱300 |
| AC-08 | AC cable | Proper 3-core cable | 1 lot | ₱300-₱1,000 |
| AC-09 | Cable glands | IP-rated | 1 lot | ₱200-₱600 |

### Estimated subtotal

**₱3,600-₱10,700**

> Final mains wiring, grounding, breaker sizing, and enclosure construction should be verified by a qualified electrician.

---

# 15. Electrical Enclosure and Panel Hardware

| ID | Component | Specification | Qty | Estimated Budget |
|---|---|---|---:|---:|
| PANEL-01 | Electrical enclosure | IP65 or better | 1 | ₱1,500-₱4,000 |
| PANEL-02 | DIN rail | 35 mm | 1-2 m | ₱200-₱500 |
| PANEL-03 | DIN terminal blocks | Power/signal distribution | 1 lot | ₱500-₱1,500 |
| PANEL-04 | Wire ferrules | Assorted sizes | 1 lot | ₱150-₱400 |
| PANEL-05 | Crimp terminals | Ring/fork/spade | 1 lot | ₱100-₱300 |
| PANEL-06 | Wire duct | Slotted | 1 lot | ₱200-₱600 |
| PANEL-07 | Cable glands | IP-rated | 1 lot | ₱300-₱800 |
| PANEL-08 | Heat-shrink tubing | Assorted | 1 lot | ₱100-₱300 |
| PANEL-09 | Wire labels | Electrical labeling | 1 lot | ₱100-₱300 |
| PANEL-10 | Cable ties | UV-resistant preferred | 1 lot | ₱100-₱250 |
| PANEL-11 | Indicator lamps | Power/status | 2-4 | ₱50-₱200 |
| PANEL-12 | Alarm buzzer | Audible alarm | 1 | ₱50-₱150 |
| PANEL-13 | Alarm acknowledge button | Panel control | 1 | ₱100-₱300 |

### Estimated subtotal

**₱3,450-₱9,600**

---

# 16. Low-Voltage Protection

| ID | Component | Specification | Qty | Estimated Budget |
|---|---|---|---:|---:|
| PROT-01 | 24 V TVS protection | Main DC bus | 1 | ₱100-₱300 |
| PROT-02 | DC EMI filter | Main DC input | 1 | ₱200-₱600 |
| PROT-03 | Ferrite cores | Sensor/power cable filtering | 1 lot | ₱100-₱300 |
| PROT-04 | Bulk capacitors | Near switching loads | 1 lot | ₱100-₱300 |
| PROT-05 | Flyback diodes | DC inductive loads | 1 lot | ₱50-₱150 |
| PROT-06 | Optocouplers | Isolation where required | 1 lot | ₱100-₱300 |

### Estimated subtotal

**₱650-₱1,950**

---

# 17. Development and Test Equipment

These are not necessarily installed in the final machine.

| ID | Component | Qty | Estimated Budget |
|---|---|---:|---:|
| DEV-01 | USB cables | 2-3 | ₱200-₱500 |
| DEV-02 | Breadboard | 2 | ₱100-₱300 |
| DEV-03 | Dupont wires | 1 set | ₱100-₱300 |
| DEV-04 | Screw terminals | 1 set | ₱100-₱300 |
| DEV-05 | Digital multimeter | 1 | ₱500-₱2,000 |
| DEV-06 | DC clamp meter | 1 | ₱800-₱2,500 |
| DEV-07 | USB-to-RS485 adapter | 1 | ₱300-₱800 |
| DEV-08 | USB-to-TTL adapter | 1 | ₱100-₱300 |
| DEV-09 | Spare ESP32-S3 | 1 | ₱500-₱1,200 |
| DEV-10 | Spare relay/MOSFET modules | 1 lot | ₱300-₱800 |

### Estimated subtotal

**₱3,000-₱8,700**

---

# 18. Recommended Initial Procurement

Do not buy every component at once.

The first development batch should be:

| Priority | Component | Qty | Target Budget |
|---:|---|---:|---:|
| 1 | ESP32-S3 | 1 | ₱500-₱1,200 |
| 2 | 7-inch Waveshare touchscreen | 1 | ₱2,000-₱3,500 |
| 3 | DS18B20 | 3 | ₱150-₱450 |
| 4 | TinyRTC / DS3231 | 1 | ₱110-₱200 |
| 5 | microSD | 1 | ₱250-₱500 |
| 6 | Isolated RS485 interface | 1-2 | ₱300-₱2,000 |
| 7 | Main level sensor | 1 | ₱1,000-₱4,000 |
| 8 | Float switches | 3-4 | ₱300-₱1,200 |
| 9 | Flow sensors | 2 | ₱600-₱2,000 |
| 10 | 24 V PSU | 1 | ₱1,500-₱5,000 |
| 11 | 5 V DC-DC | 1 | ₱100-₱400 |
| 12 | Relay/MOSFET modules | 1 lot | ₱300-₱1,000 |
| 13 | TEC1-12706 | 2 | ₱260-₱400 |
| 14 | TEC drivers | 2 | ₱1,000-₱3,000 |
| 15 | 12 V TEC supply | 1 | ₱2,000-₱5,000 |
| 16 | Heater + control | 1 set | ₱2,000-₱5,000 |
| 17 | Emergency stop | 1 | ₱300-₱800 |
| 18 | RCD/GFCI + breaker | 1 set | ₱1,500-₱4,500 |
| 19 | Electrical enclosure | 1 | ₱1,500-₱4,000 |
| 20 | Wiring/terminals/fuses/glands | 1 lot | ₱1,500-₱4,000 |

### Initial procurement target

**Approximately ₱16,000-₱48,000**

This excludes the more expensive DO and pH instrumentation.

---

# 19. Full Electronics and Electrical Budget

| Category | Estimated Range |
|---|---:|
| Controller + HMI + storage + RTC | ₱3,530-₱7,900 |
| Temperature sensing | ₱260-₱790 |
| DO system | ₱9,500-₱29,500 |
| pH system | ₱4,300-₱14,300 |
| Level and safety | ₱2,200-₱8,000 |
| Flow monitoring | ₱900-₱3,000 |
| Heater system | ₱2,650-₱8,000 |
| Peltier system | ₱4,060-₱11,250 |
| Cooling fans | ₱750-₱3,150 |
| Pump/refill controls | ₱810-₱2,880 |
| RS485 | ₱725-₱3,420 |
| DC power system | ₱4,600-₱13,600 |
| AC protection | ₱3,600-₱10,700 |
| Panel hardware | ₱3,450-₱9,600 |
| Low-voltage protection | ₱650-₱1,950 |

## Estimated Total

**Approximately ₱41,000-₱134,000**

The large range is primarily caused by the DO and pH instrumentation.

---

# 20. Recommended Project Budget

## Option A: Student / Prototype Build

### Target: ₱45,000-₱60,000

Use:

- ESP32-S3
- 7-inch HMI
- DS18B20
- Affordable Modbus DO
- Affordable Modbus pH
- 24 V power supply
- Basic TEC system
- Basic electrical enclosure
- Proper RCD/GFCI
- Proper fusing

This is suitable for a prototype and engineering validation.

---

## Option B: Better Prototype

### Target: ₱60,000-₱90,000

Use:

- Better DO transmitter
- Better pH transmitter
- Industrial-quality PSU
- Better RS485 isolation
- Better terminal blocks
- Better enclosure
- Better electrical protection
- Spare sensors
- Better wiring

This is the recommended target for a serious long-duration prototype.

---

## Option C: Laboratory-Oriented Build

### Target: ₱90,000+

Use:

- Laboratory-grade DO probe/transmitter
- Laboratory-grade pH instrumentation
- Industrial DIN-rail power supplies
- Industrial signal isolation
- Higher-grade electrical protection
- Higher-quality wiring and connectors
- Calibrated measurement equipment

---

# 21. Components Excluded From This BOM

These should be included in a separate mechanical/plumbing BOM:

- 60-gallon tank
- Circulation pump
- Air pump
- Air stones
- Air manifold
- Water block
- Heat exchanger
- Tubing
- PVC pipe
- PVC fittings
- Valves
- Filters
- Bulkhead fittings
- Refill reservoir
- Mounting brackets
- Peltier heatsink mechanical assembly
- Thermal insulation
- Mechanical supports

---

# 22. Components That Require Careful Selection

Do not choose these based only on price:

1. DO probe
2. DO transmitter
3. pH probe
4. pH transmitter
5. Main DC power supply
6. TEC power supply
7. TEC drivers
8. RCD/GFCI
9. Main breaker
10. Heater
11. Heater SSR
12. Main level sensor
13. Flow sensors
14. Electrical enclosure

---

# 23. Components Where Generic Parts Are Acceptable

The following can usually be sourced from local Philippine electronics suppliers:

- DS18B20
- DS3231
- Float switches
- Flow sensors
- MOSFET modules
- Relay modules
- Terminal blocks
- Ferrules
- Cable glands
- Fuse holders
- Fuses
- Wire duct
- Heat-shrink tubing
- Connectors
- Buzzers
- Indicator lamps
- TEC1-12706 modules

---

# 24. Philippine Procurement Strategy

Use different suppliers based on the type of component.

## Electronics and Development

Suitable sources include:

- e-Gizmo
- Local electronics stores
- Maker suppliers
- Philippine marketplace sellers

Good candidates:

- ESP32-S3
- DS18B20
- DS3231
- MOSFET modules
- Relay modules
- Connectors
- Terminal blocks
- Small DC converters

## Industrial Components

Prefer established distributors for:

- Breakers
- RCD/GFCI
- Contactors
- DIN-rail power supplies
- Industrial terminals
- Protection devices
- Industrial sensors

Potential sources include:

- RS Philippines
- DigiKey Philippines
- Mouser
- Authorized local distributors

## Commodity Components

Lazada and Shopee Philippines can be useful for:

- Fans
- Wires
- Ferrules
- Heat-shrink
- Cable glands
- Float switches
- Flow sensors
- Generic modules
- TEC modules
- Enclosures

Always verify the datasheet and seller specifications.

---

# 25. Recommended Final Electrical Architecture

```text
                     220-240 VAC
                          |
                  +-------+-------+
                  | Main Isolator |
                  +-------+-------+
                          |
                     RCD / GFCI
                          |
             +------------+-------------+
             |                          |
       Heater Branch                DC PSU
             |                          |
       Breaker/Fuse                 24 V DC
             |                          |
       SSR / Contactor        +---------+---------+
             |                |         |         |
      Aquarium Heater         |         |         |
                              |         |         |
                          ESP32-S3    Pumps     Fans
                          HMI         Valve
                          Sensors
                          RS485
                          SD
                          RTC
                              |
                        12 V TEC Supply
                              |
                    +---------+---------+
                    |                   |
              TEC Driver #1       TEC Driver #2
                    |                   |
               TEC1-12706 #1      TEC1-12706 #2
```

---

# 26. ESP32-S3 Electrical I/O Concept

A possible high-level allocation is:

```text
ESP32-S3
│
├── 1-Wire
│   ├── DS18B20 T1
│   ├── DS18B20 T2
│   └── Heatsink temperature
│
├── RS485
│   ├── DO transmitter
│   └── pH transmitter
│
├── I2C
│   └── TinyRTC
│
├── SPI
│   └── microSD
│
├── Digital Inputs
│   ├── Main LOW float
│   ├── Main HIGH float
│   ├── Emergency HIGH float
│   ├── Refill LOW float
│   ├── Emergency stop
│   └── Flow pulse inputs
│
├── Analog / ADC
│   └── Continuous level sensor if analog
│
├── Digital Outputs
│   ├── Heater control
│   ├── Circulation pump
│   ├── Air pump
│   ├── Refill solenoid
│   └── Alarm
│
└── PWM / Driver Outputs
    ├── TEC #1
    ├── TEC #2
    └── Cooling fans
```

The exact GPIO assignment should be finalized during the hardware design phase after the selected HMI and peripheral interfaces are confirmed.

---

# 27. Critical Safety Interlocks

The following conditions should be implemented before automatic operation:

## Heater OFF when:

- Main water level is LOW
- Main water level is CRITICAL LOW
- Circulation flow is absent
- Temperature sensor is invalid
- T1/T2 disagreement exceeds configured limit
- Emergency stop is active
- Critical alarm is active

## Peltier OFF when:

- Main water level is LOW
- Circulation flow is absent
- Water temperature sensor is invalid
- Heatsink temperature is too high
- Emergency stop is active
- Critical alarm is active

## Refill OFF when:

- Refill reservoir is empty
- Main HIGH float is active
- Emergency HIGH float is active
- Refill flow is missing
- Emergency stop is active
- Refill timeout occurs

---

# 28. Recommended Spare Parts

Keep the following on hand:

| Component | Recommended Spare Qty |
|---|---:|
| DS18B20 | 2 |
| Float switch | 2 |
| Flow sensor | 1 |
| TEC1-12706 | 2 |
| TEC driver | 1 |
| Relay module | 2 |
| MOSFET module | 2 |
| Fuses | 1 full set |
| CR2032 | 2 |
| microSD | 1 |
| RS485 transceiver | 1 |
| Solenoid valve | 1 |
| Fan | 1 |

This will make troubleshooting considerably easier.

---

# 29. Final Recommended Procurement Sequence

Do not purchase everything simultaneously.

## Stage 1 - Controller

Purchase:

```text
ESP32-S3
7" HMI
DS18B20
TinyRTC
microSD
RS485 interface
```

Goal:

```text
ESP32-S3
   ↓
HMI
   ↓
Temperature
   ↓
RTC
   ↓
SD
   ↓
RS485
```

---

## Stage 2 - Water Monitoring

Purchase:

```text
DO
pH
Level
Float switches
Flow sensors
```

Validate all measurements before connecting high-power equipment.

---

## Stage 3 - Power System

Purchase:

```text
24 V PSU
12 V TEC PSU
DC distribution
Fuses
RCD/GFCI
Breaker
Enclosure
DIN rail
Terminal blocks
```

Build and test the electrical panel separately from the tank.

---

## Stage 4 - Actuators

Connect one at a time:

```text
Circulation pump
       ↓
Aeration
       ↓
Refill solenoid
       ↓
Heater
       ↓
Cooling fans
       ↓
TEC #1
       ↓
TEC #2
```

---

## Stage 5 - Full Integration

Run:

```text
Sensor testing
       ↓
Safety testing
       ↓
Actuator testing
       ↓
Temperature control
       ↓
Refill control
       ↓
Peltier testing
       ↓
Data logging
       ↓
HMI
       ↓
Web interface
       ↓
24-72 hour test
       ↓
7-day reliability test
       ↓
Biological commissioning
```

---

# 30. Final Baseline BOM

For the first complete prototype, the recommended baseline is:

```text
CONTROLLER
1 × ESP32-S3
1 × 7" touchscreen
1 × TinyRTC / DS3231
1 × 32 GB microSD

TEMPERATURE
2 × waterproof DS18B20
1 × spare DS18B20
1 × heatsink temperature sensor

WATER QUALITY
1 × DO probe
1 × DO Modbus transmitter
1 × pH probe
1 × pH Modbus transmitter

LEVEL
1 × continuous tank level sensor
1 × LOW float
1 × HIGH float
1 × emergency HIGH float
1 × refill LOW float

FLOW
1 × circulation flow sensor
1 × refill flow sensor
1 × spare flow sensor

HEATING
1 × 300-500 W aquarium heater
1 × independent thermostat
1 × AC SSR
1 × SSR heatsink
1 × heater fuse/breaker

COOLING
2 × TEC1-12706
2 × independent TEC drivers
1 × 12 V high-current supply
2 × TEC fuses
1 × heatsink temperature sensor
2-4 × high-airflow fans
1 × thermal cutoff

REFILL
1 × normally-closed solenoid
1 × solenoid driver
1 × flyback diode

CONTROL
1 × circulation pump driver
1 × air pump driver
1 × alarm buzzer
1 × E-stop

POWER
1 × 24 V / 15-20 A PSU
1 × 12 V high-current TEC supply
1 × 24 V-to-5 V converter
1 × DC distribution block
1 × fuse distribution system

SAFETY
1 × main breaker
1 × RCD/GFCI
1 × main isolator
1 × AC contactor
1 × PE/ground terminal

PANEL
1 × IP65 electrical enclosure
1 × DIN rail
1 lot DIN terminals
1 lot ferrules
1 lot cable glands
1 lot wire duct
1 lot heat-shrink
1 lot labels
1 lot wiring
1 lot fuses
```

## Target Complete Prototype Budget

### Electronics + Electrical

**Approximately ₱45,000-₱90,000**

This is the budget I would use for planning the project before purchasing.

The biggest cost drivers are:

1. DO instrumentation
2. pH instrumentation
3. Peltier power system
4. Power supplies
5. Electrical protection
6. Enclosure and wiring

The mechanical and plumbing BOM should be created separately so that the total project cost can be tracked cleanly.