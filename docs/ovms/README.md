# Nissan Leaf CAN Bus Documentation

This documentation covers the CAN bus communication protocol for Nissan Leaf electric vehicles, extracted from the Open Vehicle Monitoring System 3 (OVMS3) project.

## Supported Models

| Generation | Years | Code | Notes |
|------------|-------|------|-------|
| ZE0 | 2011-2012 | Gen 1 | 24 kWh battery, requires hardware mod for full control |
| ZE0-0/1 | 2013-2016 | Gen 1.5 | 24/30 kWh battery |
| ZE0-2 | 2016-2017 | Gen 1.5 | 30 kWh, TCU on CAR bus |
| ZE1 | 2018+ | Gen 2 | 40/62 kWh, requires CAN tap behind instrument cluster |

Also supports: **Nissan e-NV200** (all models) and custom battery packs (e.g., Muxsan)

## Documentation Index

| Document | Description |
|----------|-------------|
| [CAN Bus Communication](./can-bus-communication.md) | Bus configuration, hardware setup, and protocol basics |
| [CAN Messages Reference](./can-messages-reference.md) | Complete message ID reference with byte parsing |
| [Control Commands](./control-commands.md) | Remote control sequences and procedures |
| [Metrics Reference](./metrics-reference.md) | All available sensor data and calculated values |
| [ESPHome Integration (ESP32 DevKit)](./esphome-integration.md) | ESPHome YAML for ESP32 + external CAN module |
| [ESPHome Integration (LilyGo T-CAN485)](./esphome-lilygo-t-can485.md) | ESPHome YAML for LilyGo T-CAN485 all-in-one board |

## Quick Reference

### CAN Bus Configuration

| Bus | Name | Speed | Purpose |
|-----|------|-------|---------|
| CAN1 | EV Bus | 500 kbps | Battery, motor, charger, climate |
| CAN2 | CAR Bus | 500 kbps | Doors, odometer, TPMS, gear, locks |

### Key CAN Message IDs

| ID | Bus | Description |
|----|-----|-------------|
| `0x1db` | CAN1 | Battery voltage, current, power |
| `0x55b` | CAN1 | SOC (10-bit precision) |
| `0x5bc` | CAN1 | GIDs, capacity bars |
| `0x5b3` | CAN1 | State of Health (SOH) |
| `0x5c0` | CAN1 | Battery temperature |
| `0x390` | CAN1 | Charger status (2013+) |
| `0x421` | CAN2 | Gear position, ignition, lock state |
| `0x60d` | CAN2 | Door states |

### Remote Commands

| Command | CAN ID | Notes |
|---------|--------|-------|
| Climate On | `0x56e` | Requires wakeup sequence |
| Climate Off | `0x56e` | Requires wakeup sequence |
| Start Charge | `0x56e` | Requires wakeup sequence |
| Lock Doors | `0x56e` | 2016+ or 30kWh models |
| Unlock Doors | `0x56e` | CAR CAN must be awake |

## Resources

- [OVMS GitHub Repository](https://github.com/openvehicles/Open-Vehicle-Monitoring-System-3)
- [Nissan Leaf CAN Database (DBC files)](https://github.com/dalathegreat/leaf_can_bus_messages)
- [Nissan Leaf Car Hacking Wiki](https://nissanleaf.carhackingwiki.com)
- [MyNissanLeaf Forum - CAN Discussion](http://www.mynissanleaf.com/viewtopic.php?f=44&t=4131)
