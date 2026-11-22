# Storage Pool Integrity Watch

**The scenario:** md RAID rebuilds itself at 1 % and nobody notices for a week. A btrfs checksum error lands in dmesg where nobody reads dmesg. The "mirrored" pool has had one leg missing since the day it was commissioned. ZFS is honest about corruption, md is shy, btrfs is optimistic — none of them have a dashboard. Data loss doesn't announce itself; it accumulates.

**The app recipe:**

| App | Role |
|---|---|
| [TA-zfspool](https://splunkbase.splunk.com/app/10454) | Unified pool state across ZFS, md, btrfs: degraded/failed devices, uncorrectable/corrupted read counters, resync/scrub %, spares |
| [TA-sanpath](https://splunkbase.splunk.com/app/10450) | The paths into the pool: multipath topology, failed paths, iSCSI sessions |
| [TA-disk](https://splunkbase.splunk.com/app/10429) | The members underneath it: SMART per physical disk |
| [TA-hwraid](https://splunkbase.splunk.com/app/10441) | Pools you don't own: hardware RAID controllers behind which the above are blind |

**Problems this kills:**
- Silent md degradation between scrubs (the `degraded=1` state that persists for months).
- Uncorrectable-read counters incrementing while everything says "healthy."
- Half your multipath LUNs on a dead secondary path.
- The "resilient" RAID that has had a failed drive in it since 2024.

## Killer searches

md RAID arrays with failed devices, per host (wildcard over every array name):

```
| mstats latest(zfspool.md_*_failed_devices) as failed* where index=metrics earliest=-15m by host
```

bFS/btrfs pool corruption counters trending (corrupted + uncorrectable reads):

```
| mstats max(zfspool.btrfs_*_corrupted) as corrupted* where index=metrics earliest=-6h by host
```

Any pool/storage/path alarm across the whole crew at once:

```
index=main sourcetype=zfspool:status OR sourcetype=hwraid:status OR sourcetype=sanpath:status
  status!=absent status!=ok latest=now
| table host sourcetype status
| dedup host sourcetype
```

## Who this is for

Everyone with mirrors or parity between them and their data, especially the ones who have never looked at `/proc/mdstat` voluntarily.

## Next

[The Disk Failure Autopsy](disk-failure-autopsy.md) for per-drive forensics; [Backups You Can Testify Under](backup-warrant.md) for when integrity loses.
