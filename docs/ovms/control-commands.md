# Nissan Leaf Control Commands

This document describes the remote control command sequences for the Nissan Leaf.

## Prerequisites

### Enable CAN Write Mode

Before any commands can be sent, CAN write must be enabled:

```
config set xnl canwrite true
```

**Warning:** This allows sending control messages to your vehicle. Use responsibly.

### Model Year Configuration

Set your model year for correct protocol selection:

```
config set xnl modelyear 2015
```

This affects:
- Which CAN bus receives commands (CAN1 for <2016, CAN2 for ≥2016)
- Command message length (1 byte for <2016, 4 bytes for ≥2016)
- Wakeup procedure

---

## Wakeup Sequences

Before sending any remote command, the vehicle must be woken up. The procedure varies by model year.

### 2011-2012 (ZE0 Gen 1)

**Requires hardware modification** - Wire from OVMS pin 18 (12V) to TCU pin 11.

**Sequence:**
1. Send TCU wakeup frame: `0x68C` with 1 byte `0x00` on CAN1
2. Set hardware GPIO high (EV System Activation Request)
3. Wait 1 second
4. Send command repeatedly (24 times at 100ms intervals)
5. After 1 second, set GPIO low

```
Step 1: CAN1 -> 0x68C [0x00]              // TCU wakeup
Step 2: GPIO HIGH (pin 18 -> TCU pin 11)   // EV activation request
Step 3: Wait ~100ms
Step 4: Send command frame 24x at 100ms intervals
Step 5: After ~1s, GPIO LOW
```

### 2013-2017 (ZE0 Gen 1.5)

**Requires TCU disconnection** for climate control when not charging.

**Sequence:**
1. Send wakeup frames on CAN1:
   - `0x679` with 1 byte `0x00` (VCM wakeup)
   - `0x5C0` with 8 bytes all `0x00` (battery heater request spoof)
2. Send command repeatedly (24 times at 100ms intervals)

```
Step 1: CAN1 -> 0x679 [0x00]              // VCM wakeup
Step 2: CAN1 -> 0x5C0 [00 00 00 00 00 00 00 00]  // Heater request spoof
Step 3: Wait ~100ms
Step 4: Send command frame 24x at 100ms intervals
```

### 2016-2017 (ZE0-2)

Commands sent on CAN2 (CAR bus) instead of CAN1.

**Sequence:**
1. Send wakeup frames (same as above)
2. Send command on CAN2 repeatedly

### 2018+ (ZE1)

**Requires TCU CAN wire disconnection** (unpin CAN wires from TCU connector).

**Sequence:**
Same as 2013-2017, but using CAN tap behind instrument cluster.

---

## Command Frame Format

### Message ID

All commands use CAN ID **`0x56E`**

### Message Length

| Model Year | Length | Bus |
|------------|--------|-----|
| < 2016 | 1 byte | CAN1 |
| ≥ 2016 | 4 bytes | CAN2 |

### Command Data

| Command | Byte 0 | Byte 1 | Byte 2 | Byte 3 |
|---------|--------|--------|--------|--------|
| Enable Climate | `0x4E` | `0x08` | `0x12` | `0x00` |
| Disable Climate | `0x56` | `0x00` | `0x01` | `0x00` |
| Auto Disable Climate | `0x46` | `0x08` | `0x32` | `0x00` |
| Start Charging | `0x66` | `0x08` | `0x12` | `0x00` |
| Lock Doors | `0x60` | `0x80` | `0x00` | `0x00` |
| Unlock Doors | `0x11` | `0x00` | `0x00` | `0x00` |

For pre-2016 models, only send byte 0.

---

## Command Procedures

### Enable Climate Control (Pre-heat/cool)

**Purpose:** Turn on climate control remotely to pre-condition cabin.

**Full Sequence:**
```
1. Execute wakeup sequence (model-year specific)
2. Send command frame 24 times at 100ms intervals:
   - CAN ID: 0x56E
   - Data: [0x4E, 0x08, 0x12, 0x00] (4 bytes for ≥2016)
   - Data: [0x4E] (1 byte for <2016)
3. After last command, start 1-second timer
4. Send auto-disable command: [0x46, 0x08, 0x32, 0x00]
```

**Auto-disable:** Climate control automatically sends a "heartbeat" disable command after ~15 minutes to prevent draining the battery.

**Limitations by model:**
| Model | When Climate Works |
|-------|-------------------|
| 2011-2012 | Requires hardware mod; works anytime with mod |
| 2013-2016 | Anytime (with TCU disconnected) |
| 2016-2017 | Only while charging |
| 2018+ | Anytime (with TCU CAN disconnected) |

---

### Disable Climate Control

**Purpose:** Turn off climate control remotely.

**Sequence:**
```
1. Execute wakeup sequence
2. Send command frame 24 times at 100ms intervals:
   - CAN ID: 0x56E
   - Data: [0x56, 0x00, 0x01, 0x00]
```

---

### Start Charging

**Purpose:** Initiate AC charging when plugged in.

**Preconditions:**
- Vehicle must be plugged into EVSE
- Charge pilot signal must be present (`0x390` or `0x380` indicates plugged in)

**Sequence:**
```
1. Verify charge pilot is present
2. Execute wakeup sequence
3. Send command frame 24 times at 100ms intervals:
   - CAN ID: 0x56E
   - Data: [0x66, 0x08, 0x12, 0x00]
```

---

### Stop Charging

**Purpose:** Interrupt an active charge session.

**Method:** MITM (Man-in-the-Middle) approach - modifies battery current message.

**Sequence:**
```
1. Set MITM flag for 10 iterations
2. When 0x1DB message is received:
   - Copy message data
   - Modify byte 0 to 0x20 (reduces current request)
   - Send modified message back to CAN bus
3. Timer automatically disables MITM after ~1 second
```

**Code logic:**
```c
if (m_MITM > 0) {
  unsigned char new1db[8];
  memcpy(new1db, d, 8);  // Copy original message
  new1db[0] = 0x20;       // Modify current request
  m_can1->WriteStandard(0x1db, 8, new1db);
}
```

**Note:** This is a more aggressive method that intercepts and modifies CAN traffic.

---

### Lock Doors

**Purpose:** Lock all doors remotely.

**Preconditions:**
- CAR CAN bus must be awake (turn on A/C first if needed)
- Works reliably on 2016+ and 30kWh models

**Sequence:**
```
1. Execute wakeup sequence
2. Send command frame repeatedly until lock confirmed:
   - CAN ID: 0x56E
   - Data: [0x60, 0x80, 0x00, 0x00]
3. Monitor 0x421 for lock confirmation (byte 2, bit 4 = 1)
4. Stop sending when locked
```

**Early termination:** Command stops repeating when `ms_v_env_locked` becomes true.

---

### Unlock Doors

**Purpose:** Unlock all doors remotely.

**Preconditions:**
- CAR CAN bus must be awake

**Sequence:**
```
1. Execute wakeup sequence
2. Send command frame repeatedly until unlock confirmed:
   - CAN ID: 0x56E
   - Data: [0x11, 0x00, 0x00, 0x00]
3. Monitor 0x421 for unlock confirmation (byte 2, bit 4 = 0)
4. Stop sending when unlocked
```

---

## Timing Parameters

| Parameter | Value | Description |
|-----------|-------|-------------|
| `REMOTE_COMMAND_REPEAT_COUNT` | 24 | Times to send command after first |
| `ACTIVATION_REQUEST_TIME` | 10 | Tenths of second for GPIO activation |
| Command interval | 100 ms | Time between repeated commands |
| MITM timeout | 10 iterations | Auto-disable MITM after this many cycles |
| Climate auto-disable | ~1000 ms | Delay before sending auto-disable |

---

## Command Result Monitoring

### Charge State Changes

Monitor `0x390` (AZE0) or `0x380` (ZE0) for charge status:

| Status | Value (0x390 byte 5) |
|--------|---------------------|
| Idle | `0x08` or `0x09` |
| Timer wait | `0x0C` |
| Charging | `0x04` |
| Quick Charging | `0x01` (with QC flag) |
| Finished | `0x02` |

### Lock State Changes

Monitor `0x421` byte 2:
- Bit 4 = Driver door lock (1 = locked)
- Bit 3 = Other doors lock (1 = locked)

### Climate State Changes

Monitor multiple messages:
- `0x54B` - Fan speed and vent mode
- `0x54C` - Temperature (indicates activity)
- Custom metrics: `xnl.cc.remoteheat`, `xnl.cc.remotecool`

---

## Error Handling

### Command Failures

| Scenario | Cause | Solution |
|----------|-------|----------|
| No response | Vehicle asleep | Retry wakeup sequence |
| Climate won't start | TCU connected | Disconnect/unpin TCU |
| Lock doesn't work | CAR bus asleep | Turn on A/C first |
| Command ignored | CAN write disabled | Enable `xnl.canwrite` |

### Safety Interlocks

The vehicle has built-in safety interlocks:
- Cannot start charging without pilot signal
- Cannot enable climate if battery too low (<10%)
- Cannot unlock while driving (speed > 0)

---

## Homelink Integration

Climate control can be triggered via Homelink buttons:

| Button | Action |
|--------|--------|
| 0 | Enable Climate Control |
| 1 | Disable Climate Control |

---

## Charge Limit (Auto-stop) Feature

OVMS can automatically stop charging when a target is reached.

**Configuration:**
```
config set xnl suffsoc 80        # Stop at 80% SOC
config set xnl suffrange 150     # Stop at 150km range
config set xnl autocharge true   # Enable auto-charge control
config set xnl socdrop 5         # Allowed SOC drop before restart
config set xnl rangedrop 20      # Allowed range drop before restart
```

**Logic:**
1. Monitor SOC and range during charging
2. When target reached, send stop charge command
3. If SOC/range drops below threshold, restart charging
4. Prevents overcharging and optimizes battery longevity

---

## Complete Command Flow Example

### Remote Climate On (2015 Model)

```
Time 0ms:    Send 0x679 [0x00] on CAN1           // VCM wakeup
Time 0ms:    Send 0x5C0 [00 00 00 00 00 00 00 00] on CAN1  // Heater spoof
Time 100ms:  Send 0x56E [0x4E 0x08 0x12 0x00] on CAN2  // Climate on
Time 200ms:  Send 0x56E [0x4E 0x08 0x12 0x00] on CAN2
Time 300ms:  Send 0x56E [0x4E 0x08 0x12 0x00] on CAN2
...
Time 2400ms: Send 0x56E [0x4E 0x08 0x12 0x00] on CAN2  // 24th repeat
Time 3400ms: Send 0x56E [0x46 0x08 0x32 0x00] on CAN2  // Auto-disable scheduled

// Monitor 0x54B for fan activity to confirm success
```

---

## Wakeup Frame Reference

| Frame | CAN ID | Data | Bus | Purpose |
|-------|--------|------|-----|---------|
| VCM Wakeup | `0x679` | `[0x00]` | CAN1 | Wake up Vehicle Control Module |
| TCU Wakeup | `0x68C` | `[0x00]` | CAN1 | Wake up via TCU message (2011-2012) |
| Heater Spoof | `0x5C0` | `[0x00 x8]` | CAN1 | Spoof battery heater request |
