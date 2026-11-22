# Before the Heatwave

**The scenario:** Every summer has the same incident: the first 95 °F week and a dusty server room starts cooking boxes one by one, "randomly" — GPU runs slow, a switch reboots nightly, the NAS checksums start drifting. Heat failures are slow-motion failures: headroom shrinks over weeks *visible in the sensors*, if someone is reading them. The pre-season audit is one question per box: what is your ceiling, what is your margin, and what does the throttle counter say already happened?

**The app recipe:**

| App | Role |
|---|---|
| [TA-sensors](https://splunkbase.splunk.com/app/10385) | Every hwmon reading: package/VRM/NVMe/switch temps, fan RPMs, voltages |
| [TA-throttle](https://splunkbase.splunk.com/app/10452) | Proof it already got hot — throttle counter deltas per package |
| [TA-rapl](https://splunkbase.splunk.com/app/10449) | Power draw context: heat you can't cool vs. power you can shed |
| [TA-disk](https://splunkbase.splunk.com/app/10429) | Drive temps — disks cook in hot bays before they report anything else |

**Problems this kills:**
- Dust-season mystery slowdowns, pre-dated to the counter that proves them.
- Pre-summer fan/airflow audit with numbers instead of a shop-vac anecdote.
- VRM temperatures nobody ever looks at right up to the random reboot.

## Killer searches

Package temperature headroom per host (the coretemp sensors that matter):

```
| mstats avg(sensors.coretemp_package_id_0) as pkg_temp, max(sensors.coretemp_package_id_0) as pkg_peak where index=metrics earliest=-24h by host
```

Thermal events already counted per package — last 24 hours:

```
| mstats latest(throttle.pkg_0_throttle_count) as throttle_total where index=metrics earliest=-24h by host
```

Disk-bay heat, the usual canary (per-disk temps, all disks, all hosts — wildcard sweep):

```
| mstats max(disk.*_temperature) as temp* where index=metrics earliest=-24h by host
```

## Who this is for

Server-room owners without environmental monitoring, homelab rack people, and storage admins tired of summer checksums.

## Next

When throttle counters climb mid-summer: [Node Fitness Check](node-fitness.md) to schedule around it; then [Storage Pool Integrity Watch](pool-integrity.md) — because failing disks in hot bays is how small problems become big ones.
