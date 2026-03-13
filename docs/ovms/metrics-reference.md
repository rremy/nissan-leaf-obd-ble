# Nissan Leaf Metrics Reference

Complete list of all metrics available from the Nissan Leaf CAN bus.

## Standard Metrics

These use the OVMS standard metric naming convention.

### Battery Metrics

| Metric | Unit | Source | Description |
|--------|------|--------|-------------|
| `v.b.soc` | % | 0x55B or 0x1DB | State of charge |
| `v.b.soh` | % | 0x5B3 or polling | State of health |
| `v.b.voltage` | V | 0x1DB | Pack voltage |
| `v.b.current` | A | 0x1DB | Pack current (+ = discharge) |
| `v.b.power` | kW | calculated | Pack power |
| `v.b.temp` | °C | 0x5C0 | Pack temperature |
| `v.b.range.ideal` | km | calculated | Ideal range |
| `v.b.range.est` | km | calculated | Estimated range (with temp/SOH compensation) |
| `v.b.range.full` | km | calculated | Range when full |
| `v.b.energy.used` | kWh | calculated | Energy used this trip |
| `v.b.energy.recd` | kWh | calculated | Energy recovered this trip |
| `v.b.cac` | Ah | polling | Battery capacity in Ah |
| `v.b.pack.tmin` | °C | polling | Minimum cell temperature |
| `v.b.pack.tmax` | °C | polling | Maximum cell temperature |
| `v.b.pack.tavg` | °C | polling | Average cell temperature |
| `v.b.cell.vmin` | V | polling | Minimum cell voltage |
| `v.b.cell.vmax` | V | polling | Maximum cell voltage |
| `v.b.cell.vavg` | V | polling | Average cell voltage |
| `v.b.12v.voltage` | V | hardware | 12V battery voltage |

### Charge Metrics

| Metric | Unit | Source | Description |
|--------|------|--------|-------------|
| `v.c.charging` | bool | 0x390/0x380 | Currently charging |
| `v.c.state` | string | derived | Charge state (charging/stopped/done) |
| `v.c.substate` | string | derived | Charge substate details |
| `v.c.mode` | string | 0x390 | Charge mode |
| `v.c.type` | string | derived | Charge type (type1/chademo) |
| `v.c.voltage` | V | 0x390/0x380 | Charge voltage |
| `v.c.current` | A | 0x390/0x380 | Charge current |
| `v.c.power` | kW | calculated | Charge power |
| `v.c.kwh` | kWh | calculated | Energy added this session |
| `v.c.climit` | A | 0x390 | Charge current limit |
| `v.c.pilot` | bool | 0x390/0x380 | Charge pilot signal present |
| `v.c.duration.full` | min | calculated | Time to full charge |
| `v.c.duration.soc` | min | calculated | Time to SOC limit |
| `v.c.duration.range` | min | calculated | Time to range limit |
| `v.c.limit.soc` | % | config | Target SOC limit |
| `v.c.limit.range` | km | config | Target range limit |
| `v.c.efficiency` | % | calculated | Charge efficiency |

### Vehicle State Metrics

| Metric | Unit | Source | Description |
|--------|------|--------|-------------|
| `v.e.on` | bool | 0x421 | Vehicle is on |
| `v.e.awake` | bool | 0x421 | Vehicle is awake |
| `v.e.locked` | bool | 0x421 | Doors locked |
| `v.e.handbrake` | bool | 0x5C5 | Handbrake engaged |
| `v.e.gear` | int | 0x421 | Gear position (-1=R, 0=P, 1=D) |
| `v.e.throttle` | % | 0x180 | Accelerator position |
| `v.e.footbrake` | % | 0x292 | Brake pedal position |
| `v.e.headlights` | bool | 0x60D | Headlights on |
| `v.e.charging12v` | bool | derived | 12V battery charging |

### Door Metrics

| Metric | Unit | Source | Description |
|--------|------|--------|-------------|
| `v.d.fl` | bool | 0x60D | Front left door open |
| `v.d.fr` | bool | 0x60D | Front right door open |
| `v.d.rl` | bool | 0x60D | Rear left door open |
| `v.d.rr` | bool | 0x60D | Rear right door open |
| `v.d.trunk` | bool | 0x60D | Trunk/hatch open |
| `v.d.chargeport` | bool | derived | Charge port open |

### Climate Metrics

| Metric | Unit | Source | Description |
|--------|------|--------|-------------|
| `v.e.temp` | °C | 0x54C | Ambient temperature |
| `v.e.cabintemp` | °C | 0x54F | Cabin temperature |
| `v.e.hvac` | bool | derived | HVAC system active |
| `v.e.heating` | bool | 0x54B | Heating active |
| `v.e.cooling` | bool | 0x54B | Cooling active |
| `v.e.cabinsetpoint` | °C | 0x54A | Climate setpoint |
| `v.e.cabinfan` | int | 0x54B | Fan speed (0-7) |
| `v.e.cabinvent` | string | 0x54B | Vent mode |
| `v.e.cabinintake` | string | 0x54B | Intake mode |

### Motor/Inverter Metrics

| Metric | Unit | Source | Description |
|--------|------|--------|-------------|
| `v.m.rpm` | RPM | 0x1DA | Motor RPM |
| `v.m.temp` | °C | 0x55A | Motor temperature |
| `v.i.temp` | °C | 0x55A | Inverter temperature |
| `v.i.power` | kW | calculated | Inverter power output |
| `v.i.efficiency` | % | calculated | Inverter efficiency |

### Position Metrics

| Metric | Unit | Source | Description |
|--------|------|--------|-------------|
| `v.p.speed` | km/h | 0x284 | Vehicle speed |
| `v.p.odometer` | km | 0x355/0x5C5 | Total odometer |
| `v.p.trip` | km | calculated | Trip distance |

### TPMS Metrics

| Metric | Unit | Source | Description |
|--------|------|--------|-------------|
| `v.tp.fl.p` | PSI | 0x385 | Front left pressure |
| `v.tp.fr.p` | PSI | 0x385 | Front right pressure |
| `v.tp.rl.p` | PSI | 0x385 | Rear left pressure |
| `v.tp.rr.p` | PSI | 0x385 | Rear right pressure |

### Vehicle Info

| Metric | Unit | Source | Description |
|--------|------|--------|-------------|
| `v.vin` | string | polling | Vehicle Identification Number |
| `v.type` | string | - | "NL" (Nissan Leaf) |

---

## Nissan Leaf Custom Metrics

These are Leaf-specific metrics with the `xnl.` prefix.

### Battery Custom Metrics

| Metric | Unit | Description |
|--------|------|-------------|
| `xnl.v.b.gids` | - | GIDs (battery energy units) |
| `xnl.v.b.max.gids` | - | Maximum GIDs (new car) |
| `xnl.v.b.hx` | % | HX health indicator |
| `xnl.v.b.soc.newcar` | % | SOC relative to new car capacity |
| `xnl.v.b.soc.instrument` | % | SOC from instrument cluster |
| `xnl.v.b.soc.nominal` | % | Nominal SOC |
| `xnl.v.b.soh.newcar` | % | SOH relative to new car capacity |
| `xnl.v.b.soh.instrument` | % | SOH from BMS |
| `xnl.v.b.range.instrument` | km | Range from instrument cluster |
| `xnl.v.b.e.capacity` | kWh | Battery energy capacity |
| `xnl.v.b.e.available` | kWh | Available battery energy |
| `xnl.v.b.type` | int | Battery type (0=24kWh Gen1, etc.) |
| `xnl.v.b.output.limit` | kW | Discharge power limit |
| `xnl.v.b.regen.limit` | kW | Regen power limit |
| `xnl.v.b.charge.limit` | kW | Max charge rate |
| `xnl.v.b.capacitybars` | - | Capacity bars (0-12) |

### Battery Heater Metrics

| Metric | Unit | Description |
|--------|------|-------------|
| `xnl.v.b.heaterpresent` | bool | Battery heater installed |
| `xnl.v.b.heatrequested` | bool | Heater requested |
| `xnl.v.b.heatergranted` | bool | Heater request granted |

### BMS Cell Metrics

| Metric | Unit | Description |
|--------|------|-------------|
| `xnl.bms.thermistor` | - | Thermistor values (vector) |
| `xnl.bms.temp.int` | °C | Internal temperatures (vector) |
| `xnl.bms.balancing` | bitset | Cell balancing status (96 bits) |

### Charge Custom Metrics

| Metric | Unit | Description |
|--------|------|-------------|
| `xnl.v.c.duration` | min | Charge durations (vector) |
| `xnl.v.c.chargeminutes3kW` | min | Minutes at 3kW remaining |
| `xnl.v.c.quick` | - | Quick charge indicator |
| `xnl.v.c.chargebars` | - | Remaining charge bars (0-12) |
| `xnl.v.c.limit.reason` | string | Charge limit reason |
| `xnl.v.c.count.qc` | - | Quick charge count (lifetime) |
| `xnl.v.c.count.l0l1l2` | - | AC charge count (lifetime) |
| `xnl.v.c.state.previous` | string | Previous charge state |
| `xnl.v.c.event.notification` | string | Charge notification status |
| `xnl.v.c.event.reason` | string | Charge event reason |
| `xnl.v.c.relay.qc` | - | QC relay status |
| `xnl.v.c.relay.ac` | - | AC relay status |
| `xnl.v.c.mode.raw` | int | Raw charge mode value |

### Climate Custom Metrics

| Metric | Unit | Description |
|--------|------|-------------|
| `xnl.cc.fan.only` | bool | Fan only mode (no heat/cool) |
| `xnl.cc.remoteheat` | bool | Remote heating active |
| `xnl.cc.remotecool` | bool | Remote cooling active |
| `xnl.cc.rqinprogress` | bool | Climate request in progress |
| `xnl.v.e.hvac.auto` | bool | Auto climate mode |
| `v.e.cabinfanlimit` | int | Fan speed limit |

### Position Custom Metrics

| Metric | Unit | Description |
|--------|------|-------------|
| `xnl.v.pos.odometer.start` | km | Odometer at trip start |

### V2X/Export Metrics

| Metric | Unit | Description |
|--------|------|-------------|
| `v.g.generating` | bool | V2X export active |
| `v.g.pilot` | bool | V2X pilot signal |
| `v.g.voltage` | V | V2X voltage |
| `v.g.current` | A | V2X current |
| `v.g.power` | kW | V2X power |
| `v.g.kwh` | kWh | Energy exported |
| `v.g.state` | string | V2X state |
| `v.g.substate` | string | V2X substate |
| `v.g.duration.empty` | min | Time to empty at current rate |
| `v.g.duration.soc` | min | Time to SOC limit |
| `v.g.duration.range` | min | Time to range limit |

---

## Calculated Values

### Range Calculation

```
km_per_kwh = 7.1 (default, configurable)
wh_per_gid = 80

max_kwh = battery_capacity_kwh
range_max = max_kwh * km_per_kwh

// Temperature compensation (optimal at 20°C)
avg_temp = (ambient + battery_temp*3) / 4
factor_temp = (100 - |avg_temp - 20| * 1.25) / 100

range_ideal = range_max * soc / 100
range_full = range_max * soh / 100 * factor_temp
range_est = range_ideal * soh / 100 * factor_temp
```

### Energy Available

```
energy_kwh = gids * 0.080  // 80 Wh per GID
```

### SOC (New Car Basis)

```
max_gids = configured_max_gids  // e.g., 281 for 24kWh
soc_new_car = (current_gids * 100) / max_gids
```

### Inverter Power

```
power_kw = (motor_rpm * motor_torque * 0.10472) / 1000
```

### Charge Time Remaining

```
charge_power_w = cumulative_energy_wh * 360  // Convert to watts
target_energy_wh = (target_soc - current_soc) / 100 * capacity_wh
minutes_remaining = (target_energy_wh / charge_power_w) * 60
```

---

## Metric Staleness

Metrics become "stale" when not updated for a period:

| Staleness Level | Duration | Use Case |
|-----------------|----------|----------|
| `SM_STALE_NONE` | Never stale | Counters, configuration |
| `SM_STALE_MIN` | 1 minute | Active sensors |
| `SM_STALE_MID` | 10 minutes | Periodic data |
| `SM_STALE_HIGH` | 1 hour | Slow-changing data |

The vehicle uses staleness to detect when the car has gone to sleep (e.g., `v.e.awake` goes stale → car is asleep).
