# 02 Technical Requirements

## 1. Purpose

This document defines the technical requirements for the Crayfish Incubator Version 1.

The system shall use an ESP32-S3 as the primary controller and shall provide automated monitoring, environmental control, water circulation, aeration, automatic refill, local HMI, Wi-Fi web interface, data logging, and CSV export.

All biological operating limits shall remain configurable because the final crayfish species and incubation requirements have not yet been finalized.

---

# 2. System Requirements

## 2.1 Controller

The system shall use:

- ESP32-S3 as the primary controller.
- Wi-Fi for local web access.
- I2C for the RTC and compatible peripherals.
- 1-Wire for DS18B20 temperature sensors.
- RS485 for industrial sensors.
- GPIO for digital inputs and outputs.
- PWM or dedicated control interfaces for Peltier control.
- SPI for microSD storage where applicable.

The controller shall operate continuously for 24 hours per day, 7 days per week.

---

# 3. Main Tank Requirements

## 3.1 Tank Capacity

Nominal tank capacity:

- 60 US gallons.
- Approximately 227 liters.

The system shall not use the physical maximum tank volume as the normal operating level.

The normal operating level shall leave sufficient freeboard for:

- Water movement.
- Aeration.
- Thermal expansion.
- Refill control.
- Emergency protection.

## 3.2 Tank Monitoring

The main tank shall provide:

- Continuous water-level measurement.
- Independent low-level safety detection.
- Independent high-level safety detection.
- Water temperature measurement.
- Water circulation flow monitoring.

---

# 4. Water Temperature Requirements

## 4.1 Temperature Sensors

The system shall use two waterproof DS18B20 sensors.

Suggested assignments:

```text
T1 = Primary water temperature
T2 = Independent verification temperature
```

The controller shall detect:

- Sensor disconnection.
- Invalid readings.
- Out-of-range readings.
- Excessive disagreement between T1 and T2.

## 4.2 Temperature Control

The system shall support configurable:

- Target temperature.
- Heating start threshold.
- Heating stop threshold.
- Cooling start threshold.
- Cooling stop threshold.
- High-temperature alarm.
- Low-temperature alarm.
- Maximum allowable temperature.
- Minimum allowable temperature.

The temperature control algorithm shall include hysteresis to prevent rapid switching.

## 4.3 Temperature Sensor Agreement

The system shall compare T1 and T2.

If the difference exceeds a configurable limit for a configurable period:

```text
T1 ≠ T2
    ↓
Temperature Sensor Warning
    ↓
Alarm
    ↓
Evaluate Control Safety
```

Critical control decisions shall not depend on a sensor identified as faulty.

---

# 5. Heating Requirements

## 5.1 Heater

The system shall use an aquarium-rated water heater.

Initial design target:

- Approximately 300 to 500 W.

The final heater rating shall be determined during thermal testing.

## 5.2 Heater Safety

The heater shall have an independent thermostat.

The ESP32-S3 shall control the heater through an electrically isolated switching device.

The heater shall be disabled when:

- Main tank water level is critically low.
- Required circulation is unavailable.
- Temperature sensors are invalid.
- High-temperature protection is active.
- Emergency stop is active.
- System power fault occurs.

The heater shall not be powered directly from an ESP32 GPIO.

---

# 6. Peltier Cooling Requirements

## 6.1 Peltier Modules

The cooling system shall use:

- 2 × TEC1-12706.

The two modules shall be independently controllable.

The system shall support:

```text
TEC 1 OFF
TEC 1 ON

TEC 2 OFF
TEC 2 ON

TEC 1 + TEC 2
```

The final cooling capacity shall be validated experimentally.

The system shall not assume that two TEC1-12706 modules are sufficient for all ambient conditions until thermal testing has been completed.

## 6.2 Cooling Architecture

The cooling system shall use:

```text
Main Tank
    ↓
Circulation Pump
    ↓
Flow Sensor
    ↓
Water Block
    ↓
Tank Return
```

The Peltier modules shall be thermally coupled to the water block.

The hot side shall use:

- High-performance heatsink.
- High-airflow fans.
- Hot-side temperature sensor.

## 6.3 Peltier Power

The TEC1-12706 modules shall not be connected directly to:

- ESP32 GPIO.
- 24 V supply.

The power architecture shall provide the correct operating voltage and current for the TEC modules.

The preferred architecture is:

```text
DC Power Supply
       ↓
TEC Power Driver
       ↓
┌───────────────┐
│               │
TEC 1         TEC 2
```

The final driver and power-supply ratings shall be selected based on the measured electrical requirements of the selected TEC modules.

## 6.4 Cooling Control

The controller shall use water temperature as the primary cooling control variable.

The system shall support:

- Cooling setpoint.
- Cooling hysteresis.
- TEC 1 control.
- TEC 2 control.
- Maximum TEC power.
- Hot-side temperature limit.
- Minimum circulation flow.
- Cooling fault timeout.

The controller may use staged cooling:

```text
Small cooling demand
        ↓
TEC 1

Higher cooling demand
        ↓
TEC 1 + TEC 2
```

The controller shall reduce cooling power as the target temperature is approached where practical.

## 6.5 Peltier Thermal Protection

If the hot-side temperature reaches the configured protection limit:

```text
TEC 1 OFF
TEC 2 OFF
Fans ON
Alarm
```

The cooling system shall remain disabled until the hot-side temperature returns to a safe range.

---

# 7. Circulation Requirements

## 7.1 Circulation Pump

The circulation pump shall be suitable for continuous operation.

Initial design target:

- Approximately 1,000 to 2,000 L/h.

The final pump shall be selected after accounting for:

- Pipe diameter.
- Pipe length.
- Fittings.
- Filter restriction.
- Water block restriction.
- Vertical head.
- Required flow through the cooling loop.

## 7.2 Flow Monitoring

The system shall use an inline flow sensor.

The flow sensor shall provide a measurable flow signal compatible with the ESP32-S3 system.

The system shall calculate:

```text
Flow Rate = L/min
```

The controller shall provide configurable:

- Minimum flow.
- Low-flow alarm delay.
- Flow failure timeout.

## 7.3 Circulation Fault

If the circulation pump is commanded ON but insufficient flow is detected:

```text
Low Flow
    ↓
Alarm
    ↓
Heater OFF
Cooling OFF
```

Aeration shall remain independently controlled unless a separate safety condition requires shutdown.

---

# 8. Aeration Requirements

The aeration system shall operate independently from water circulation.

The system shall include:

- Air pump.
- Air manifold.
- Multiple air stones or equivalent diffusers.
- Air tubing.

The aeration system shall normally operate continuously.

The DO measurement shall be used for monitoring and alarm generation.

Version 1 shall not automatically inject oxygen or control oxygen concentration.

---

# 9. Dissolved Oxygen Requirements

## 9.1 Sensor

The system shall use a laboratory or industrial dissolved oxygen sensor.

Preferred interface:

- RS485.
- Modbus RTU.

The final sensor shall be selected based on:

- Required accuracy.
- Measurement range.
- Water compatibility.
- Temperature compensation.
- Calibration requirements.
- Probe replacement cost.
- Availability in the Philippines.

## 9.2 DO Data

The controller shall acquire:

- DO concentration.
- Sensor status where available.
- Sensor temperature where provided.
- Communication status.

The system shall support configurable:

- Low DO alarm.
- Critical DO alarm.
- Sensor timeout.
- Communication timeout.

---

# 10. pH Requirements

## 10.1 Sensor

The system shall use a replaceable laboratory or industrial pH probe.

Preferred architecture:

```text
pH Probe
    ↓
pH Transmitter
    ↓
RS485 Modbus RTU
    ↓
ESP32-S3
```

## 10.2 pH Data

The controller shall acquire:

- pH value.
- Sensor status where available.
- Temperature compensation data where available.
- Communication status.

The system shall support configurable:

- Low pH alarm.
- High pH alarm.
- Critical pH alarm.
- Sensor timeout.
- Communication timeout.

Automatic pH dosing is outside Version 1.

---

# 11. Main Tank Level Requirements

## 11.1 Continuous Level Sensor

The system shall use a continuous level sensor.

The sensor shall provide enough resolution to:

- Display approximate tank level.
- Detect the refill threshold.
- Detect the target refill level.
- Detect abnormal level changes.

The final sensor technology shall be selected during hardware design.

Candidate technologies include:

- Ultrasonic.
- Pressure-based level sensor.
- Other suitable non-contact or water-compatible level measurement technologies.

## 11.2 Safety Level Switches

The system shall include independent level switches:

```text
LOW FLOAT
HIGH FLOAT
```

The switches shall operate independently of the continuous level sensor.

The low-level switch shall protect:

- Heater.
- Circulation system.
- Peltier water block.

The high-level switch shall protect against overfilling.

---

# 12. Refill Reservoir Requirements

The system shall use an elevated refill reservoir.

The reservoir shall include:

- Water storage container.
- Level monitoring.
- Normally-closed solenoid valve.
- Manual shutoff valve.
- Refill flow sensor.
- Appropriate plumbing.

The reservoir shall have a low-level protection condition.

If the reservoir is empty:

```text
Refill Request
     ↓
Reservoir LOW
     ↓
Do Not Open Solenoid
     ↓
Alarm
```

---

# 13. Automatic Refill Requirements

The refill system shall use a normally-closed solenoid valve.

The controller shall initiate refill when:

```text
Main Level < Refill Threshold
```

Before opening the valve, the controller shall verify:

- Reservoir available.
- No emergency high-level condition.
- No refill fault.
- System operating mode permits automatic refill.

After opening the valve, the controller shall verify refill flow.

If the solenoid is ON and flow is not detected within the configured timeout:

```text
Refill Flow Missing
    ↓
Close Solenoid
    ↓
Alarm
```

The controller shall close the solenoid when:

- Target level is reached.
- High-level switch activates.
- Reservoir becomes empty.
- Refill timeout occurs.
- Emergency stop activates.
- Controller detects a refill fault.

---

# 14. Refill Flow Requirements

The refill line shall use a separate inline flow sensor.

The system shall monitor:

- Refill flow rate.
- Total refill duration.
- Flow presence.
- Flow failure.

The system should record the estimated volume added during each refill event.

Each refill event should contain:

- Start timestamp.
- End timestamp.
- Duration.
- Flow status.
- Estimated volume.
- Reason for refill.
- Result.
- Fault status if applicable.

---

# 15. TinyRTC Requirements

The system shall include a TinyRTC module with battery backup.

The RTC shall communicate with the ESP32-S3 through I2C.

The system shall use the RTC as the persistent wall-clock source.

The RTC shall provide:

- Year.
- Month.
- Day.
- Hour.
- Minute.
- Second.

The controller shall validate the RTC time during startup.

The system shall detect:

- I2C communication failure.
- Invalid date/time.
- RTC read failure.
- Backup battery failure where detectable.

## 15.1 RTC Synchronization

When Wi-Fi is available, the system may synchronize the RTC with an NTP time source.

After synchronization:

```text
NTP Time
   ↓
ESP32-S3
   ↓
TinyRTC
```

The TinyRTC shall continue providing time when Wi-Fi or Internet connectivity is unavailable.

If no valid network time is available, the system shall continue operating using the RTC.

## 15.2 RTC Usage

The RTC shall be used for:

- Data logging timestamps.
- Alarm timestamps.
- Equipment event timestamps.
- Refill event timestamps.
- Startup timestamps.
- Shutdown timestamps.
- CSV export timestamps.

The ESP32-S3 internal timers shall be used for short-duration control timing.

---

# 16. Data Logging Requirements

The system shall use a microSD card for local data storage.

The minimum permanent logging interval shall be:

```text
1 minute
```

The control system may sample sensors more frequently.

The data logger shall record:

- Timestamp.
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

## 16.1 Event Logging

The system shall also record important events.

Examples:

- System startup.
- System shutdown.
- Heater ON/OFF.
- TEC 1 ON/OFF.
- TEC 2 ON/OFF.
- Pump ON/OFF.
- Aeration state changes.
- Refill started.
- Refill completed.
- Refill fault.
- Alarm activated.
- Alarm cleared.
- Sensor communication failure.
- Sensor recovery.
- Emergency shutdown.

Event records shall include:

- Timestamp.
- Event type.
- Event source.
- Event state.
- Relevant value where applicable.

---

# 17. CSV Export Requirements

The ESP32-S3 web interface shall provide CSV export.

The user shall be able to select:

- Start date.
- Start time.
- End date.
- End time.

The system shall generate a CSV file containing records within the selected range.

The CSV shall include a header row.

Minimum fields:

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

The CSV shall use standard comma-separated formatting.

The exported file shall be compatible with common spreadsheet applications.

The system shall not require cloud connectivity to export data.

For large exports, the firmware shall avoid loading the complete dataset into RAM.

The system should stream records from the microSD card to the HTTP response where practical.

---

# 18. HMI Requirements

The local HMI shall display real-time system information.

The main dashboard shall display:

- Water temperature.
- Dissolved oxygen.
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
- System mode.
- Date and time.

The HMI shall provide:

1. Dashboard.
2. Water Quality.
3. Temperature.
4. Equipment Status.
5. Historical Data.
6. Alarm History.
7. Settings.
8. Manual Control.
9. System Information.

---

# 19. Web Interface Requirements

The ESP32-S3 shall host the web interface locally.

The web interface shall provide:

- Dashboard.
- Live sensor values.
- Equipment status.
- Historical graphs.
- Historical records.
- Alarm history.
- Settings.
- Manual control.
- System information.
- Date and time.
- CSV export.

The interface shall support:

- Desktop browsers.
- Tablet browsers.
- Mobile browsers.

The system shall continue operating if no client is connected.

Loss of Wi-Fi connectivity shall not stop automatic control.

---

# 20. Configuration Requirements

The following values shall be configurable:

### Temperature

- Target temperature.
- Heating threshold.
- Cooling threshold.
- High-temperature alarm.
- Low-temperature alarm.
- Maximum safe temperature.

### Water Level

- Refill threshold.
- Target level.
- High-level threshold.
- Emergency high-level condition.

### Flow

- Minimum circulation flow.
- Minimum refill flow.
- Flow timeout.
- Refill timeout.

### Dissolved Oxygen

- Low DO alarm.
- Critical DO alarm.

### pH

- Low pH alarm.
- High pH alarm.
- Critical pH alarm.

### Peltier

- TEC 1 maximum power.
- TEC 2 maximum power.
- Hot-side temperature limit.
- Cooling hysteresis.

Configuration values shall be stored in non-volatile memory.

---

# 21. Operating Mode Requirements

The system shall support:

## Automatic

All automatic control functions are active.

## Manual

The operator may manually control supported equipment.

Safety interlocks shall remain active.

## Maintenance

Selected automatic functions may be disabled.

Critical safety functions shall remain active.

## Alarm

Affected equipment shall be placed into the configured safe state.

---

# 22. Safety Requirements

The system shall implement multiple levels of protection.

## 22.1 Electrical Protection

The system shall include:

- Main circuit breaker.
- RCD/GFCI protection.
- DC fuses.
- Branch protection.
- Proper grounding.
- Enclosed electrical terminals.
- Emergency stop.

## 22.2 Heater Protection

The heater shall have:

- Independent thermostat.
- Software temperature control.
- Low-water interlock.
- Low-flow interlock.
- Sensor-failure interlock.

## 22.3 Peltier Protection

The Peltier system shall have:

- Correct voltage/current supply.
- Independent TEC control.
- Hot-side temperature monitoring.
- Cooling fan monitoring where practical.
- Minimum water-flow interlock.
- Thermal shutdown.

## 22.4 Refill Protection

The refill system shall have:

- Normally-closed solenoid.
- High-level float switch.
- Continuous level monitoring.
- Reservoir low-level detection.
- Refill flow monitoring.
- Refill timeout.
- Emergency shutoff.

---

# 23. Power Failure Requirements

After power is restored, the system shall:

1. Initialize the ESP32-S3.
2. Initialize the TinyRTC.
3. Validate the RTC time.
4. Initialize sensors.
5. Initialize the microSD card.
6. Initialize the HMI.
7. Initialize the web server.
8. Verify safety inputs.
9. Determine the current water conditions.
10. Restore the appropriate automatic operating state.

High-power equipment shall not automatically restart until required safety conditions have been verified.

The refill solenoid shall remain closed during initialization.

---

# 24. Sensor Failure Requirements

The system shall detect sensor failures where technically possible.

Sensor failure conditions may include:

- Communication timeout.
- Invalid reading.
- Out-of-range reading.
- Disconnected sensor.
- Implausible sensor value.

When a critical sensor fails, the system shall place affected control functions into a safe state.

Examples:

```text
Temperature Sensor Failure
        ↓
Heater OFF
Cooling OFF
Alarm
```

```text
Flow Sensor Failure
        ↓
Heater OFF
Cooling OFF
Alarm
```

```text
Level Sensor Failure
        ↓
Refill Disabled
Alarm
```

The exact fallback strategy shall be finalized during software design.

---

# 25. Communication Requirements

The system shall use:

| Device | Interface |
|---|---|
| DS18B20 sensors | 1-Wire |
| TinyRTC | I2C |
| DO transmitter | RS485 Modbus RTU |
| pH transmitter | RS485 Modbus RTU |
| Continuous level sensor | TBD |
| Main flow sensor | Pulse or compatible signal |
| Refill flow sensor | Pulse or compatible signal |
| HMI | ESP32-S3 integrated/display interface |
| microSD | SPI or display-integrated SD interface |
| Web interface | Wi-Fi |

RS485 communication shall support:

- Configurable baud rate.
- Device address configuration.
- Communication timeout.
- Retry handling.
- Invalid response detection.
- Sensor offline detection.

---

# 26. Data Retention Requirements

The system shall store historical data on microSD.

The firmware shall be designed to prevent uncontrolled growth of temporary files.

The data logger shall support long-term operation.

The exact file rotation strategy shall be defined during software design.

Possible structure:

```text
/data/
    /sensor/
        2026-10-01.csv
        2026-10-02.csv
        2026-10-03.csv

/events/
        2026-10-01.csv
        2026-10-02.csv
        2026-10-03.csv
```

The final storage structure shall be selected during software design.

---

# 27. Performance Requirements

The system shall prioritize control reliability over web-interface responsiveness.

Target behavior:

- Critical safety inputs: continuously monitored.
- Temperature control: faster than permanent logging interval.
- Flow monitoring: continuous or high-frequency sampling.
- Refill monitoring: continuous during refill.
- DO/pH: according to sensor response and Modbus polling requirements.
- Permanent logging: every 1 minute.
- Web dashboard: near-real-time updates.
- CSV export: must not interfere with safety control.

A large CSV export shall not block the main control loop.

---

# 28. Recovery Requirements

The system shall recover from:

- Wi-Fi disconnection.
- Web browser disconnection.
- Sensor communication interruption.
- Temporary sensor recovery.
- Controller restart.
- Power interruption.

A temporary loss of Wi-Fi shall not disable:

- Temperature control.
- Water circulation.
- Aeration.
- Water-level protection.
- Automatic refill safety.
- Alarm processing.
- Data logging.

---

# 29. Version 1 Technical Acceptance Criteria

The system shall pass the following tests before Version 1 is considered complete:

### Temperature

- Two DS18B20 sensors provide stable readings.
- Temperature disagreement is detected.
- Heater control operates correctly.
- Heater safety interlocks operate correctly.
- Peltier cooling operates correctly.
- Peltier thermal protection operates correctly.

### Water Circulation

- Pump operates continuously.
- Flow sensor provides valid readings.
- Low-flow condition is detected.
- Heater shuts down during insufficient flow.
- Peltier cooling shuts down during insufficient flow.

### Water Refill

- Low-level condition starts the refill sequence.
- Reservoir availability is verified.
- Solenoid opens correctly.
- Refill flow is detected.
- Target level closes the solenoid.
- High-level switch closes the solenoid.
- Empty reservoir prevents refill.
- No-flow refill fault is detected.

### Water Quality

- DO data is received through RS485.
- pH data is received through RS485.
- Communication failures are detected.
- Configured alarms operate correctly.

### RTC

- RTC time is retained after controller restart.
- RTC continues operating without Wi-Fi.
- Timestamps are correctly stored.
- RTC synchronization works when network time is available.

### Data Logging

- Sensor records are stored every minute.
- Equipment events are stored.
- Alarm events are stored.
- Data survives controller restart.
- Long-duration logging does not interfere with control.

### CSV Export

- User can select a date/time range.
- Historical data can be exported.
- CSV contains the required headers.
- CSV data matches stored records.
- CSV opens correctly in spreadsheet software.
- Large exports do not stop automatic control.

### User Interface

- HMI displays live data.
- HMI displays alarms.
- HMI provides required controls.
- Web interface displays live data.
- Web interface displays historical data.
- Web interface provides CSV export.
- Manual controls respect safety interlocks.

### Safety

- Emergency stop operates correctly.
- Low-water protection operates correctly.
- High-water protection operates correctly.
- Low-flow protection operates correctly.
- Peltier thermal protection operates correctly.
- Heater safety protection operates correctly.
- Refill safety protection operates correctly.
- High-power equipment cannot be directly controlled from ESP32 GPIO.

---

# 30. Requirements Pending Final Component Selection

The following technical values shall remain TBD until the final components are selected:

- Exact DO sensor model.
- Exact pH sensor model.
- Exact continuous level sensor.
- Exact circulation pump.
- Exact circulation flow sensor.
- Exact refill flow sensor.
- Exact heater model.
- Exact Peltier power driver.
- Exact DC power supply.
- Exact hot-side heatsink.
- Exact cooling fan.
- Exact electrical protection components.
- Exact HMI model if different from the selected Waveshare display.
- Exact pipe diameter.
- Exact filtration system.
- Exact water block.
- Exact control-loop parameters.

These values shall be finalized in the Hardware Design and Software Design documents.

# 31. Design Constraints

The system shall follow these constraints:

1. ESP32-S3 shall remain the primary controller.
2. Version 1 shall operate locally without cloud dependency.
3. Critical control functions shall not depend on Wi-Fi.
4. High-power loads shall use appropriate drivers and electrical isolation.
5. The heater shall have independent hardware protection.
6. The Peltier system shall have independent thermal protection.
7. The refill valve shall be normally closed.
8. The main tank shall have independent high and low safety switches.
9. The system shall maintain local time using TinyRTC.
10. Historical data shall be stored locally.
11. CSV export shall work without cloud connectivity.
12. CSV generation shall not block critical control functions.
13. All biological operating limits shall remain configurable.
14. The system shall be designed for continuous operation.
15. Safety interlocks shall remain active in Manual and Maintenance modes.