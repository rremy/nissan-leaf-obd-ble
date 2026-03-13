# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

Home Assistant custom integration that communicates with Nissan Leaf vehicles via OBD-II Bluetooth Low Energy adapters. Exposes real-time vehicle data (battery state, tyre pressures, motor power, etc.) as Home Assistant sensors.

## Development Commands

```bash
# Lint and format (via pre-commit)
pre-commit run --all-files

# Individual tools
ruff check --fix .
ruff format .
black .
mypy .

# Run tests (no tests currently exist)
pytest
```

## Architecture

### Communication Stack

The integration uses a layered communication stack derived from python-OBD:

```
API (api.py)
    |-- queries all Leaf OBD commands and collects responses
    v
OBD (obd.py)
    |-- manages protocol session, headers, and command building
    v
ELM327 (elm327.py)
    |-- handles ELM327 AT command initialization and low-level send/receive
    v
bleserial (bleserial.py)
    |-- BLE GATT interface using bleak; emulates serial port behavior
```

### BLE Adapter Support

Three BLE service profiles are supported (auto-detected in `bleserial.py`):
- LeLink: `0000ffe0-...` / `0000ffe1-...`
- Veepeak/Vgate: `0000fff0-...` / `0000fff1-...` (read), `0000fff2-...` (write)
- Nordic UART Service: `6e400001-...` (service), `6e400002-...` (write), `6e400003-...` (read)

### Home Assistant Integration Pattern

Standard HA custom component structure:
- `__init__.py`: Entry point, discovers BLE device, creates coordinator
- `coordinator.py`: `DataUpdateCoordinator` with adaptive polling (fast/slow/ultra-slow based on car availability)
- `sensor.py` / `binary_sensor.py`: Entity definitions using `SensorEntityDescription`
- `config_flow.py`: UI configuration flow

### OBD Command System

Commands are defined in `commands.py` as `OBDCommand` objects with:
- Command bytes (sent to ELM327)
- CAN header (set via `AT SH` command)
- Decoder function from `decoders.py`

Each decoder parses raw CAN frame data and returns a dict of sensor values. The `lbc` command (Li-ion Battery Controller) returns multiple values from a multi-frame response.

### Polling Intervals

Defined in `coordinator.py`:
- Fast (10s): Car is on and responding
- Slow (5min): Car is off but in range
- Ultra-slow (1hr): Device out of range (BLE advertisement triggers reconnection)

## Key Files

- `commands.py`: All Nissan Leaf-specific OBD commands with CAN headers
- `decoders.py`: Raw CAN data parsing functions
- `protocols/protocol_can.py`: ISO 15765-4 CAN frame reassembly
- `manifest.json`: Integration metadata, BLE service UUIDs for discovery

## Reference Documentation

The `docs/ovms/` directory contains Nissan Leaf CAN bus documentation extracted from [OVMS3](https://github.com/openvehicles/Open-Vehicle-Monitoring-System-3):

- `docs/ovms/README.md`: Overview, supported models (ZE0/ZE1), quick reference tables
- `docs/ovms/can-bus-communication.md`: Bus configuration, hardware setup, protocol basics
- `docs/ovms/can-messages-reference.md`: Complete CAN message ID reference with byte parsing
- `docs/ovms/control-commands.md`: Remote control sequences (climate, charging, locks)
- `docs/ovms/metrics-reference.md`: All available sensor data and calculated values

Key CAN buses: EV Bus (CAN1, 500kbps) for battery/motor/charger, CAR Bus (CAN2, 500kbps) for doors/TPMS/gear.
