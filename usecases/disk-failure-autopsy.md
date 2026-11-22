# The Disk Failure Autopsy

**The scenario:** A pool degrades at 2am. By morning the rebuild already failed twice and you're restoring from backups that are stale. Drive failures almost never arrive without warning — SMART wear, growing media errors, dropping spare blocks, failed devices in md/btrfs/ZFS pools, and disappearing storage paths all wave flags for days or weeks first. Nobody has those flags on one screen.

**The app recipe:**

| App | Role |
|---|---|
| [TA-disk](https://splunkbase.splunk.com/app/10429) | Per-disk SMART: NVMe wear %, spare blocks, media errors, temperature, SATA attributes |
| [TA-zfspool](https://splunkbase.splunk.com/app/10454) | Pool state for ZFS / md RAID / btrfs: degraded, failed/corrupted devices, resync % |
| [TA-hwraid](https://splunkbase.splunk.com/app/10441) | Hardware RAID via MegaCLI/StorCLI/ssacli: controller + per-drive state, predictive failures |
| [TA-sanpath](https://splunkbase.splunk.com/app/10450) | Device-mapper multipath + iSCSI session/path health |

**Problems this kills:**
- "How did I not see it coming?" — wear and error counters trended per disk, per host.
- Silent md/btrfs degradation between scrubs.
- Failed predictive-fail drives sitting in hardware arrays no one polls.
- Multipath losing secondary paths until the primary walks into the maintenance window.

## Killer searches

Disks that are not healthy, right now, across the fleet:

```
index=main sourcetype=disk:status status!=ok latest=now
| table _time host
| dedup host
```

Every unhealthy pool/storage event across all four apps at once:

```
index=main sourcetype=zfspool:status OR sourcetype=hwraid:status OR sourcetype=sanpath:status
  status!=absent status!=ok latest=now
| table host sourcetype status
| dedup host sourcetype
```

Hardware RAID drives in predictive-fail state (excludes hosts with no controller):

```
index=main sourcetype=hwraid:status NOT status=absent latest=now
| table host controller_count drive_count failed_drives predictive_drives
| dedup host
```

Drive temperatures fleet-wide (wildcard metric — works for every disk name):

```
| mstats max(disk.*_temperature) as temp* where index=metrics earliest=-15m by host
```

## Who this is for

Homelabbers running real data on it, MSPs who inherit drives people swore were fine, and any storage admin who has ever typed `zpool status` and seen two hundred `DEGRADED` entries.

## Next

Pair with [Storage Pool Integrity Watch](pool-integrity.md) for continuous scrub/corruption trend lines, and [Backups You Can Testify Under](backup-warrant.md) for the last line of defense.
