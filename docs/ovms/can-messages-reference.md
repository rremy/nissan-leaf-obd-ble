# Nissan Leaf CAN Messages Reference

Complete reference of CAN message IDs, byte layouts, and parsing formulas.

## CAN1 (EV Bus) Messages

### 0x1D4 - Charge Status Extended

**Length:** 8 bytes
**Frequency:** During charging

| Byte | Bits | Description | Formula |
|------|------|-------------|---------|
| 6 | 7 | Charge interlock | 0 = charging allowed |

---

### 0x1DA - Motor/Inverter Data

**Length:** 8 bytes
**Frequency:** 10 Hz when car is on

| Byte | Bits | Description | Formula |
|------|------|-------------|---------|
| 2-3 | all | Motor torque | `(d[2] << 8 \| d[3]) / 4.0` Nm (signed) |
| 4-5 | all | Motor RPM | `d[4] << 8 \| d[5]` (signed, 0x7FFF/0x7FFE = invalid) |

**Calculated values:**
- Inverter power: `RPM * Torque * 0.10472 / 1000.0` kW

---

### 0x1DB - Battery Pack Data (Primary)

**Length:** 8 bytes
**Frequency:** 10 Hz

| Byte | Bits | Description | Formula |
|------|------|-------------|---------|
| 0 | 7:0 | Current (high) | See below |
| 1 | 7:5 | Current (low) | See below |
| 1 | 4:0 | Voltage (high) | See below |
| 2 | 7:0 | Voltage (mid) | See below |
| 3 | 7:6 | Voltage (low) | See below |
| 4 | 6:0 | SOC (instrument) | Direct % (ZE0 only, 0x7F = invalid) |

**Current calculation (signed, 0.5A resolution):**
```c
int16_t raw = (d[0] << 3) | (d[1] >> 5);
if (raw & 0x400) raw |= 0xF800;  // Sign extend 11-bit
float current = raw * 0.5;  // Amps (positive = discharge)
```

**Voltage calculation (0.5V resolution):**
```c
uint16_t raw = (d[2] << 2) | (d[3] >> 6);
float voltage = raw * 0.5;  // Volts
```

**Power calculation:**
```c
float power = (voltage * current) / 1000.0;  // kW
```

---

### 0x1DC - Battery Power Limits

**Length:** 8 bytes
**Frequency:** 1 Hz

| Byte | Bits | Description | Formula |
|------|------|-------------|---------|
| 0 | 7:0 | Discharge limit (high) | See below |
| 1 | 7:6 | Discharge limit (low) | See below |
| 1 | 5:0 | Regen limit (high) | See below |
| 2 | 7:4 | Regen limit (low) | See below |

**Discharge limit:**
```c
uint16_t raw = (d[0] << 2) | (d[1] >> 6);
float limit = raw / 4.0;  // kW
```

**Regen/charge limit:**
```c
uint16_t raw = ((d[1] & 0x3F) << 4) | (d[2] >> 4);
float limit = raw / 4.0;  // kW
```

---

### 0x284 - Vehicle Speed

**Length:** 8 bytes
**Frequency:** 10 Hz when car is on

| Byte | Bits | Description | Formula |
|------|------|-------------|---------|
| 4-5 | all | Speed | `(d[4] << 8 \| d[5]) / 100.0` km/h |

**Note:** Speed divisor may vary by model (default ~98-100)

---

### 0x380 - Charger Status (ZE0 2011-2012)

**Length:** 8 bytes
**Frequency:** 1 Hz during charging
**Models:** ZE0 only (presence indicates Gen 1 charger)

| Byte | Bits | Description | Value |
|------|------|-------------|-------|
| 0 | 7:0 | Line voltage type | `00` = unknown, `01` = 100V, `02` = 200V |
| 5 | 7:0 | Charge status | See charge status table |

**Charge status values (byte 5):**
| Value | Status |
|-------|--------|
| `0x28` | Idle (vehicle on) |
| `0x30` | Timer wait |
| `0x38` | Asleep/no cable |
| `0x40` | Finished |
| `0x60` | L2 Charging |
| `0x70` | Starting charge |
| `0x18` | Cable plugged (beep) |
| `0xB0` | Quick Charging |

---

### 0x390 - Charger Status (AZE0 2013+)

**Length:** 8 bytes
**Frequency:** 1 Hz
**Models:** AZE0, ZE1 (presence indicates Gen 2 charger)

| Byte | Bits | Description | Formula |
|------|------|-------------|---------|
| 1 | 5:0 | Charge power (high) | See below |
| 2 | 7:4 | Charge power (low) | See below |
| 3 | 1:0 | Line voltage type | `00` = unknown, `01` = 100V, `02` = 200V |
| 3 | 6 | QC status | 1 = Quick charging |
| 3 | 7 | V2G status | 1 = V2X/exporting |
| 5 | 3:0 | Charge state | See below |

**Charge power:**
```c
uint16_t raw = ((d[1] & 0x3F) << 4) | (d[2] >> 4);
float power_w = raw;  // Watts
```

**Charge state (byte 5, bits 3:0):**
| Value | Status |
|-------|--------|
| `0x01` | QC or V2G active |
| `0x02` | Finished |
| `0x04` | Charging |
| `0x08` | Idle |
| `0x09` | Idle |
| `0x0C` | Timer wait |

---

### 0x50A - Battery Heater Request

**Length:** 8 bytes
**Models:** 2013+

| Byte | Bits | Description | Value |
|------|------|-------------|-------|
| - | - | Battery heater request | (Heating below -17°C) |

---

### 0x54A - Climate Setpoint

**Length:** 8 bytes
**Frequency:** When HVAC active

| Byte | Bits | Description | Formula |
|------|------|-------------|---------|
| 0 | 7:0 | Temperature setpoint | `(d[0] / 2.0) - 30.0` °C |

---

### 0x54B - Climate Status

**Length:** 8 bytes
**Frequency:** 1 Hz

| Byte | Bits | Description | Formula |
|------|------|-------------|---------|
| 4 | 5:3 | Fan speed | 0-7 levels |
| 4 | 7:6 | Vent mode | See vent table |
| 5 | 7:0 | Intake mode | See intake table |

**Vent mode (byte 4 high nibble):**
| Value | Mode |
|-------|------|
| `0x80` | Off |
| `0x88` | Face |
| `0x90` | Face + Feet |
| `0x98` | Feet |
| `0xA0` | Windscreen + Feet |
| `0xA8` | Windscreen |

**Intake mode (byte 5):**
| Value | Mode |
|-------|------|
| `0x09` | Recirculate |
| `0x12` | Fresh air |
| `0x92` | Defrost |

---

### 0x54C - Ambient Temperature

**Length:** 8 bytes
**Frequency:** 1 Hz

| Byte | Bits | Description | Formula |
|------|------|-------------|---------|
| 6 | 7:0 | Ambient temp | `(d[6] / 2.0) - 40.0` °C |

---

### 0x54F - Cabin Temperature

**Length:** 8 bytes
**Frequency:** 1 Hz

| Byte | Bits | Description | Formula |
|------|------|-------------|---------|
| 0 | 7:0 | Cabin temp | `(5.0/9.0) * (d[0] - 32)` °C (from °F) |

**Note:** Add configurable offset (`cabintempoffset`) for calibration

---

### 0x55A - Motor/Inverter Temperatures

**Length:** 8 bytes
**Frequency:** 1 Hz

| Byte | Bits | Description | Formula |
|------|------|-------------|---------|
| 1 | 7:0 | Motor temp | `(5.0/9.0) * (d[1] - 32)` °C |
| 2 | 7:0 | Inverter temp | `(5.0/9.0) * (d[2] - 32)` °C |

---

### 0x55B - SOC (Battery Reported)

**Length:** 8 bytes
**Frequency:** 1 Hz

| Byte | Bits | Description | Formula |
|------|------|-------------|---------|
| 0 | 7:0 | SOC (high) | See below |
| 1 | 7:6 | SOC (low) | See below |

**SOC calculation (10-bit, 0.1% resolution):**
```c
uint16_t raw = (d[0] << 2) | (d[1] >> 6);
float soc = raw / 10.0;  // Percentage
```

**Note:** This is the actual battery SOC, never reaches 0% or 100% due to safety buffers.

---

### 0x59E - SOH by Battery Type

**Length:** 8 bytes
**Frequency:** Periodic

| Byte | Bits | Description | Formula |
|------|------|-------------|---------|
| 0-2 | varies | SOH | Depends on battery type |

**For 24kWh Gen 1:**
```c
uint16_t raw = (d[1] << 1) | (d[2] >> 7);
float soh = raw / 10.0;  // Percentage
```

---

### 0x5A9 - Range (Instrument Cluster)

**Length:** 8 bytes
**Frequency:** 1 Hz

| Byte | Bits | Description | Formula |
|------|------|-------------|---------|
| 1 | 7:0 | Range (high) | See below |
| 2 | 7:4 | Range (low) | See below |

**Range calculation:**
```c
uint16_t raw = (d[1] << 4) | (d[2] >> 4);
if (raw != 0xFFF) {  // 0xFFF = invalid
  float range_km = raw;
}
```

---

### 0x5B3 - State of Health (SOH)

**Length:** 8 bytes
**Frequency:** Periodic
**Models:** ZE0 only (ZE1 uses BMS polling)

| Byte | Bits | Description | Formula |
|------|------|-------------|---------|
| 1 | 7:0 | SOH (high) | See below |
| 2 | 7 | SOH (low) | See below |

**SOH calculation:**
```c
uint16_t raw = (d[1] << 1) | (d[2] >> 7);
float soh = raw / 10.0;  // Percentage (0.1% resolution)
```

---

### 0x5B9 - Remaining Charge Bars

**Length:** 8 bytes
**Frequency:** 1 Hz

| Byte | Bits | Description | Formula |
|------|------|-------------|---------|
| 1 | 7:3 | Charge bars | `(d[1] & 0xF8) >> 3` (0-12 bars) |

---

### 0x5BC - GIDs and Capacity

**Length:** 8 bytes
**Frequency:** 1 Hz

| Byte | Bits | Description | Formula |
|------|------|-------------|---------|
| 0 | 7:0 | GIDs (high) | See below |
| 1 | 7:6 | GIDs (low) | See below |
| 2 | 3:0 | Capacity bars | 0-12 bars |
| 5 | 7:5 | Message type | See below |

**GIDs calculation (battery energy units):**
```c
uint16_t gids = (d[0] << 2) | (d[1] >> 6);
float energy_kwh = gids * 0.080;  // 80 Wh per GID
```

**Message type (byte 5, bits 7:5):**
| Value | Content |
|-------|---------|
| `0x00` | GIDs (24/30kWh) or charge bars |
| `0x01` | Capacity bars |

---

### 0x5BF - Charger Status (ZE0 Gen1)

**Length:** 8 bytes
**Models:** ZE0 2011-2012 only

| Byte | Bits | Description | Value |
|------|------|-------------|-------|
| 4 | 7:0 | Charge mode | See table |

**Charge mode values:**
| Value | Status |
|-------|--------|
| `0x28` | Vehicle on, idle |
| `0x30` | Timer wait |
| `0x38` | Asleep/no cable |
| `0x40` | Finished |
| `0x60` | L2 Charging |
| `0x70` | Starting |
| `0xB0` | Quick Charging |

---

### 0x5C0 - Battery Temperature (LBC)

**Length:** 8 bytes
**Frequency:** 1 Hz

| Byte | Bits | Description | Formula |
|------|------|-------------|---------|
| 0 | 7:0 | Battery temp | `(d[0] / 2.0) - 40.0` °C |

**Note:** 7-bit precision (bottom bit always 0)

---

### 0x679 - VCM Wakeup Signal

**Length:** 1 byte
**Purpose:** J1772/VCM wakeup indicator

When this message is received, the vehicle modules are waking up.

---

## CAN2 (CAR Bus) Messages

### 0x180 - Throttle Position

**Length:** 8 bytes
**Frequency:** 10 Hz when car is on

| Byte | Bits | Description | Formula |
|------|------|-------------|---------|
| 5 | 7:0 | Throttle | `d[5] / 2.0` % (0xFF = invalid) |

---

### 0x292 - Brake Pedal

**Length:** 8 bytes
**Frequency:** 10 Hz when car is on

| Byte | Bits | Description | Formula |
|------|------|-------------|---------|
| 6 | 7:0 | Brake pressure | `d[6] / 1.39` % (0xFF = invalid) |

---

### 0x355 - Odometer

**Length:** 8 bytes
**Frequency:** 1 Hz

| Byte | Bits | Description | Formula |
|------|------|-------------|---------|
| 1 | 7:0 | Odometer (high) | See below |
| 2 | 7:0 | Odometer (mid) | See below |
| 3 | 7:0 | Odometer (low) | See below |
| 4 | 5 | Units | 0 = km, 1 = miles |

**Odometer calculation:**
```c
uint32_t odo = (d[1] << 16) | (d[2] << 8) | d[3];
// Units determined by byte 4 bit 5
```

---

### 0x385 - TPMS Pressures

**Length:** 8 bytes
**Frequency:** Periodic (if TPMS equipped)

| Byte | Bits | Description | Formula |
|------|------|-------------|---------|
| 2 | 7:0 | Front Left | `d[2] / 4.0` PSI |
| 3 | 7:0 | Front Right | `d[3] / 4.0` PSI |
| 4 | 7:0 | Rear Right | `d[4] / 4.0` PSI |
| 5 | 7:0 | Rear Left | `d[5] / 4.0` PSI |

**Note:** Value of 0 means no reading

---

### 0x421 - Gear and Ignition State

**Length:** 8 bytes
**Frequency:** 10 Hz

| Byte | Bits | Description | Value |
|------|------|-------------|-------|
| 0 | 5:3 | Gear position | See table |
| 1 | 2:1 | Ignition state | See table |
| 2 | 4 | Driver door lock | 1 = locked |
| 2 | 3 | Other doors lock | 1 = locked |

**Gear position (byte 0, bits 5:3):**
| Value | Gear |
|-------|------|
| 0 | Park |
| 1 | Reverse |
| 2 | Neutral |
| 3 | Drive |
| 4 | Eco/B mode |

**Ignition state (byte 1, bits 2:1):**
| Value | State |
|-------|-------|
| 0 | Off |
| 1 | Accessory |
| 2 | On (not ready) |
| 3 | Ready to drive |

---

### 0x5C5 - Handbrake and Odometer

**Length:** 8 bytes
**Frequency:** 1 Hz

| Byte | Bits | Description | Value |
|------|------|-------------|-------|
| 0 | 2 | Handbrake | 1 = engaged |
| 1-3 | all | Odometer | Same as 0x355 |

---

### 0x60D - Door States

**Length:** 8 bytes
**Frequency:** On change

| Byte | Bits | Description | Value |
|------|------|-------------|-------|
| 0 | 7 | Trunk/Hatch | 1 = open |
| 0 | 6 | Rear Right | 1 = open |
| 0 | 5 | Rear Left | 1 = open |
| 0 | 4 | Front Right | 1 = open |
| 0 | 3 | Front Left | 1 = open |
| 0 | 1 | Headlights (dip) | 1 = on |
| 0 | 0 | Headlights (high) | 1 = on |

---

## BMS Polling Responses (ISO-TP)

### Group 0x01 - Battery Status

**TX:** `0x79B`, **RX:** `0x7BB`
**Request:** `21 01`
**Response length:** 39 bytes (ZE0) or 51 bytes (ZE1)

**ZE0 (39 bytes):**
| Offset | Description | Formula |
|--------|-------------|---------|
| 0-1 | HX value | `(d[0] << 8 \| d[1]) / 100.0` |
| 5-7 | Ah capacity | `(d[5] << 16 \| d[6] << 8 \| d[7]) / 10000.0` Ah |

**ZE1 (51 bytes):**
| Offset | Description | Formula |
|--------|-------------|---------|
| 28-29 | HX value | `(d[28] << 8 \| d[29]) / 102.4` |
| 31-33 | SOC | `(d[31] << 16 \| d[32] << 8 \| d[33]) / 10000.0` % |
| 35-37 | Ah capacity | `(d[35] << 16 \| d[36] << 8 \| d[37]) / 10000.0` Ah |

---

### Group 0x02 - Cell Voltages

**TX:** `0x79B`, **RX:** `0x7BB`
**Request:** `21 02`
**Response length:** 196 bytes

Contains voltages for all 96 cells (2 bytes each, big-endian):
```c
for (int i = 0; i < 96; i++) {
  uint16_t mv = (data[i*2] << 8) | data[i*2 + 1];
  float voltage = mv / 1000.0;  // Volts
}
```

---

### Group 0x04 - Cell Temperatures

**TX:** `0x79B`, **RX:** `0x7BB`
**Request:** `21 04`
**Response length:** 14 bytes

Contains temperature data from thermistors in the battery pack.

---

### Group 0x06 - Cell Balancing (Shunts)

**TX:** `0x79B`, **RX:** `0x7BB`
**Request:** `21 06`
**Response length:** 96 bytes

Contains balancing status for each cell (1 = balancing active).

---

### Group 0x61 - SOH (ZE1 Only)

**TX:** `0x79B`, **RX:** `0x7BB`
**Request:** `21 61`

Contains SOH data for ZE1 models.

---

### PID 0x1203 - Quick Charge Count

**TX:** `0x797`, **RX:** `0x79A`
**Request:** `22 12 03`
**Response length:** 2 bytes

```c
uint16_t qc_count = (data[0] << 8) | data[1];
```

---

### PID 0x1205 - L0/L1/L2 Charge Count

**TX:** `0x797`, **RX:** `0x79A`
**Request:** `22 12 05`
**Response length:** 2 bytes

```c
uint16_t ac_count = (data[0] << 8) | data[1];
```

---

### PID 0x81 - VIN

**TX:** `0x797`, **RX:** `0x79A`
**Request:** `21 81`
**Response length:** 19 bytes

VIN is returned as ASCII string (17 characters + 2 header bytes).

