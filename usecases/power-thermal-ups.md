# Power, Thermal & the UPS That Lied

**The scenario:** The rack pulls 480 W and you're billing "about 300." The UPS said "online, 100 %" until the moment it didn't, and it never had a battery-test log for the 2 a.m. conversation. Your servers cook to 97 °C, clock down, and nobody can tell you how many watts are actually going through the chip. Hardware telemetry for this exists — on every machine you own — and nobody collects it.

**The app recipe:**

| App | Role |
|---|---|
| [TA-ups](https://splunkbase.splunk.com/app/10453) | NUT/apcupsd integration: UPS state, battery, load, presence |
| [TA-rapl](https://splunkbase.splunk.com/app/10449) | RAPL powercap: per-domain package/dram/cores energy (joules → watts), AMD hsmp, hwmon power rails |
| [TA-throttle](https://splunkbase.splunk.com/app/10452) | Frequency + thermal throttle counters — what your heat costs in performance |
| [TA-sensors](https://splunkbase.splunk.com/app/10385) | Every hwmon temp sensor: VRMs, NVMe, chips, waterblocks |

**Problems this kills:**
- "What does the homelab actually cost to run?" — answerable from `pc_*_energy_uj` deltas, no plug-meter archaeology.
- Silent thermal throttle: workload slowdowns attributed to software, when it was dust.
- UPS that reports perfect health right up to a 4-second brownout failover.
- Hotspots nobody's cooling (VRM at 105 °C while CPU temp looks lovely).

## Killer searches

Per-host power infrastructure visibility:

```
| mstats latest(rapl.powercap_count) as powercaps, latest(rapl.amd_hsmp_hwmon_power1_watts) as pkg_watts where index=metrics earliest=-60m by host
```

UPS presence/absence per host (a missing UPS on a critical box is the finding):

```
| mstats latest(ups.device_count) as ups_devices where index=metrics earliest=-7d by host
```

Thermal events in the last hour, with clock cost:

```
| mstats latest(throttle.pkg_0_throttle_count) as throttle_events, latest(throttle.freq_avg_mhz) as avg_mhz where index=metrics earliest=-60m by host
```

## Who this is for

Homelab power nerds, colo operators chasing "what's in this circuit?", and anyone who has priced a server room and wanted real numbers.

## Next

GPU boxes have their own heat story — [GPU Fleet Watch](gpu-fleet.md). Crash-under-heat questions go to [Kernel Crash Readiness](crash-ready.md).
