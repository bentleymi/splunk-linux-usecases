# The 3-Boxes-in-a-Closet Health Board

**The scenario:** You run a homelab or a small fleet — a handful of boxes doing real work: Plex, k3s, backups, that one VM nobody will admit to owning. Enterprise suites want five figures and a consultant; SNMP is a 1998 relic with a CSV habit. You want one dashboard per box that answers "is anything about to hurt?" before it does.

**The app recipe:**

| App | Role |
|---|---|
| [TA-memwatch](https://splunkbase.splunk.com/app/10444) | Free-memory %, swap thrash (pswpin/out), slab, OOM kills |
| [TA-netqual](https://splunkbase.splunk.com/app/10446) | Per-interface carrier, speed, errors, drops, link flaps |
| [TA-service](https://splunkbase.splunk.com/app/10451) | Failed systemd units |
| [TA-cronbeat](https://splunkbase.splunk.com/app/10437) | Stale/dead systemd timers and cron jobs |
| [TA-kernel](https://splunkbase.splunk.com/app/10443) | Taint flags, kdump readiness, new oops/bug/panic |
| [TA-patch](https://splunkbase.splunk.com/app/10447) | Upgradable + security packages, days-since-update |
| [TA-throttle](https://splunkbase.splunk.com/app/10452) | Per-package frequency + thermal throttle events |
| [TA-disk](https://splunkbase.splunk.com/app/10429) | SMART/wear per disk |

**Problems this kills:**
- Waking up to a box that OOM-killed your database at 4am — memory pressure has a slope, you get the slope.
- "The nightly job hasn't run in nine days" (timer went stale, nobody noticed).
- Services that failed once and died quietly because nothing was watching `systemctl --failed`.
- A CPU silently pinned at idle clocks because of thermal events nobody was polling for.

## Killer searches

One-line health row per host (paste into a `table`; every cell is a red-flag magnet):

```
| mstats latest(memwatch.mem_available_pct) as mem_pct, latest(memwatch.swap_used_pct) as swap_pct, latest(service.active_count) as services where index=metrics earliest=-15m by host
```

Hosts that taint themselves or are silently cooking:

```
| mstats latest(kernel.tainted) as tainted, latest(kernel.kdump_active) as kdump_active, latest(throttle.pkg_0_throttle_count) as throttle_count where index=metrics earliest=-60m by host
```

Anything currently alarming in the scheduled-jobs plane:

```
index=main sourcetype=cronbeat:status status!=ok latest=now
| table host status timers_stale timers_total
| dedup host
```

## Who this is for

Homelab operators, small MSPs, and anyone whose "monitoring stack" is currently SSH plus vibes.

## Install

Install apps from Splunkbase on (forwarder or single instance), enable inputs, done — collectors are stdlib-only Python. For full SMART detail let the `splunk` user run `smartctl` via the sudoers wrapper shipped in TA-disk's `deploy/`.

## Next

When one row of the board starts glowing, drill in with [GPU Fleet Watch](gpu-fleet.md), [Blame It on the Network](prove-network-blame.md), or [The Disk Failure Autopsy](disk-failure-autopsy.md).
