# The Reboot Archaeologist

**The scenario:** Two weeks from now, someone asks "did that box reboot last Tuesday?" "No." "Are you sure?" "Pretty." A uptime metric answers this forensically: `uptime_s` is an odometer that resets to zero exactly at boot — every unexplained dip is a reboot, and the neighboring telemetry (was `apt` running? did a maintenance timer fire? did the services layer fail?) distinguishes a reboot you chose from one the kernel chose for you.

**The app recipe:**

| App | Role |
|---|---|
| [TA-kernel](https://splunkbase.splunk.com/app/10443) | `uptime_s` odometer + new panic/oops counters |
| [TA-patch](https://splunkbase.splunk.com/app/10447) | Was a package transaction in flight when the clock reset? |
| [TA-cronbeat](https://splunkbase.splunk.com/app/10437) | Maintenance timers: the usual suspects for planned reboots |
| [TA-service](https://splunkbase.splunk.com/app/10451) | Failed-unit aftermath around each boot |

**Problems this kills:**
- The gaslighting reboot ("that never happens on this host") with a timestamped count.
- Kernel panics reclassified as reboots in the monthly uptime report because the panic wasn't caught when `kdump` was dead.
- `needrestart`-style surprises: the service failed and nothing brought it back.

## Killer searches

Uptime min/max per host over a week — a wide gap is a reboot (or several):

```
index=main sourcetype=kernel:status earliest=-7d latest=now
| stats min(uptime_s) as min_uptime max(uptime_s) as max_uptime count by host
```

Cross-check against the maintenance plane: any patch activity at that hour?

```
| mstats latest(patch.upgradable) as pkgs where index=metrics earliest=-24h by host
```

## Who this is for

Every ops team whose uptime claims depend on someone's memory, and every "who power-cycled the hypervisor?" investigation.

## Next

[Kernel Crash Readiness](crash-ready.md) makes unplanned boots productive via vmcores; [The One-Query Incident Timeline](incident-timeline.md) surrounds each reboot with context.
