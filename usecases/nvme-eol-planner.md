# NVMe End-of-Life Planner

**The scenario:** NVMe drives don't die loudly; they fade — endurance wears down, spare blocks get spent, and the manufacturer's warranty clock runs while nobody's looking. The drive that says `percentage_used: 97` is not a failure, it's an appointment you missed. The data to replace drives *before* they fail is already in SMART, sitting unread — until the minute it matters.

**The app recipe:**

| App | Role |
|---|---|
| [TA-disk](https://splunkbase.splunk.com/app/10429) | Per-NVMe: `percentage_used` wear, `available_spare` vs threshold, `media_errors`, `power_on_hours`, temperature |
| [TA-zfspool](https://splunkbase.splunk.com/app/10454) | Which pool each disk actually backs |
| [TA-sanpath](https://splunkbase.splunk.com/app/10450) | Path redundancy so a planned swap is a non-event |

**Problems this kills:**
- "Drive X lasted how long?" answered before the drive answers for you.
- Spare-block exhaustion discovered by a failed rebuild instead of a spare-block counter.
- Fleet-wide "which 20 of my 200 disks should be ordered this quarter?" in one table.

## Killer searches

Wear percentage for every NVMe in the fleet (wildcard across all disk series):

```
| mstats latest(disk.*_percentage_used) as wear* where index=metrics earliest=-24h by host | head 12
```

Per-disk temperature is the leading indicator of wear acceleration — same wildcard pattern with `_temperature`.

## Who this is for

Homelabbers whose NAS is the family photo archive, flash-array people who bought TLC thinking it was SLB, and anyone planning a disk-refresh budget from telemetry instead of vibes.

## Next

[Storage Pool Integrity Watch](pool-integrity.md) before pulling a member in; [The Disk Failure Autopsy](disk-failure-autopsy.md) when the appointment gets skipped.
