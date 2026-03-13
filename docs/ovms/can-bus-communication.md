# Nissan Leaf CAN Bus Communication

## Overview

The Nissan Leaf uses two separate CAN buses operating at 500 kbps:

| Bus | Name | Purpose | Access |
|-----|------|---------|--------|
| **CAN1** | EV Bus | Battery, motor, charger, climate control | OBD-II port (ZE0) or CAN tap (ZE1) |
| **CAN2** | CAR Bus | Body electronics, doors, TPMS, gear | OBD-II port (ZE0) or CAN tap (ZE1) |

## Hardware Connection

### ZE0 Models (2011-2017)

Connect via standard OBD-II port using the OVMS cable (1779000).

**OBD-II Pinout:**
| Pin | Signal | Description |
|-----|--------|-------------|
| 6 | CAN1_H | EV CAN High |
| 14 | CAN1_L | EV CAN Low |
| 12 | CAN2_H | CAR CAN High (2013+ required for some features) |
| 13 | CAN2_L | CAR CAN Low |
| 16 | +12V | Battery positive |
| 4/5 | GND | Ground |

### ZE1 Models (2018+)

The OBD-II port cannot be used because the CAN gateway module powers down when ignition is off. You must tap the CAN buses behind the instrument cluster.

**CAN Gateway Location:** Behind instrument cluster (24-pin connector)

**Required Wiring (8 wires):**
| Wire | DB9 Pin | Gateway Pin | Signal |
|------|---------|-------------|--------|
| 1 | 7 | 1 | CAN1_H (EV) |
| 2 | 2 | 2 | CAN1_L (EV) |
| 3 | 5 | 5 | CAN2_H (CAR) |
| 4 | 4 | 6 | CAN2_L (CAR) |
| 5 | 9 | 15 | +12V |
| 6 | 3 | 16 | Ground |
| 7 | - | - | Additional ground |
| 8 | - | - | Ignition sense (optional) |

## CAN Bus Protocol

### Standard 11-bit CAN (CAN 2.0A)

All Nissan Leaf CAN communication uses standard 11-bit identifiers.

```
+--------+--------+--------+--------+--------+--------+--------+--------+--------+--------+--------+
|  SOF   |   11-bit ID   | RTR | IDE | r0 |   DLC   |    0-8 Data Bytes    |   CRC   | ACK | EOF |
+--------+--------+--------+--------+--------+--------+--------+--------+--------+--------+--------+
```

### ISO-TP (ISO 15765-2)

For diagnostic polling (OBD-II), the Leaf uses ISO-TP for multi-frame messages:

| TX ID | RX ID | ECU | Description |
|-------|-------|-----|-------------|
| `0x79B` | `0x7BB` | BMS | Battery Management System |
| `0x797` | `0x79A` | OBC | On-Board Charger |
| `0x7DF` | - | Broadcast | Broadcast query |

**Polling PIDs:**
| PID | Type | Description |
|-----|------|-------------|
| `0x01` | Group | Battery status (39-51 bytes) |
| `0x02` | Group | Cell voltages (196 bytes) |
| `0x04` | Group | Cell temperatures (14 bytes) |
| `0x06` | Group | Cell balancing/shunts (96 bytes) |
| `0x61` | Group | SOH data (ZE1 only) |
| `0x81` | VIN | Vehicle Identification Number |
| `0x1203` | Extended | Quick charge count |
| `0x1205` | Extended | L0/L1/L2 charge count |

## Poll States

The OVMS uses different polling intervals based on vehicle state:

| State | Value | Condition | Poll Frequency |
|-------|-------|-----------|----------------|
| OFF | 0 | Car is off | No polling |
| ON | 1 | Car is on (not driving) | Normal |
| RUNNING | 2 | Car in Drive/Reverse | Normal |
| CHARGING | 3 | Car is charging | Focused on charge data |

## Message Timing

### Broadcast Messages (Passive Reading)

These messages are transmitted continuously by the vehicle:

| Message | Frequency | Notes |
|---------|-----------|-------|
| `0x1db` (Battery) | 10 Hz | High frequency during driving |
| `0x1da` (Motor) | 10 Hz | Only when car is on |
| `0x284` (Speed) | 10 Hz | Only when car is on |
| `0x5bc` (GIDs) | 1 Hz | Always when awake |
| `0x55b` (SOC) | 1 Hz | Always when awake |
| `0x5c0` (Temp) | 1 Hz | Always when awake |
| `0x390` (Charger) | 1 Hz | Only during charging |

### Polled Messages

These require sending a request and waiting for response:

| Request | Response Time | Notes |
|---------|---------------|-------|
| BMS Group 01 | ~50-100ms | Battery pack status |
| BMS Group 02 | ~200-300ms | All cell voltages |
| BMS Group 04 | ~50ms | Temperatures |
| VIN Query | ~100ms | One-time query |

## CAN Write Mode

By default, CAN write is disabled for safety. Enable with:

```
config set xnl canwrite true
```

**Warning:** CAN write allows sending control commands to the vehicle. Use with caution.

## Model Year Detection

The firmware auto-detects some model features based on CAN traffic:

| Message | Indicates |
|---------|-----------|
| `0x380` present | ZE0 (2011-2012) charger |
| `0x390` present | AZE0 (2013+) charger |
| Different SOC source | ZE1 uses BMS polling, ZE0 uses `0x1db` |

## CRC Calculation

Some messages use Nissan's proprietary CRC (last byte):

```c
unsigned char leafcrc(int length, unsigned char* data) {
  const unsigned char crcTable[] = {
    0x00, 0x85, 0x8F, 0x0A, 0x9B, 0x1E, 0x14, 0x91,
    0xB3, 0x36, 0x3C, 0xB9, 0x28, 0xAD, 0xA7, 0x22,
    // ... (full 256-byte table)
  };
  unsigned char crc = 0;
  for (int i = 0; i < length; i++) {
    crc = crcTable[crc ^ data[i]];
  }
  return crc;
}
```

## Battery Pack Configurations

| Pack | Capacity | Max GIDs | Max Ah | Wh/GID |
|------|----------|----------|--------|--------|
| 24 kWh (Gen 1) | 24 kWh | 281 | 66 | 80 |
| 30 kWh | 30 kWh | 356 | 79 | 80 |
| 40 kWh | 40 kWh | 502 | 115 | 80 |
| 62 kWh | 62 kWh | 775 | 176 | 80 |

## Configuration Parameters

| Parameter | Default | Description |
|-----------|---------|-------------|
| `xnl.modelyear` | 2012 | Model year for protocol selection |
| `xnl.canwrite` | false | Enable CAN write mode |
| `xnl.maxGids` | 281 | Max GIDs for SOC calculation |
| `xnl.newCarAh` | 66 | New car Ah capacity |
| `xnl.whPerGid` | 80 | Watt-hours per GID |
| `xnl.kmPerKWh` | 7.1 | Efficiency for range calculation |
| `xnl.ze1` | false | Enable ZE1-specific features |
| `xnl.soc.newcar` | false | Use calculated SOC vs instrument |
| `xnl.speeddivisor` | 98.0 | Speed calculation divisor |
