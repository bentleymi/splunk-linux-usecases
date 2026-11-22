# Backups You Can Testify Under

**The scenario:** The backup job that "always works" failed silently eleven nights ago. restic finished but wrote nowhere; the systemd timer got disabled during that refactor; the borg archive is corrupt at chunk 40%. Insurance, management, and future-you all assume backups run — the only honest answer comes from a system that watches the watcher, on its own clock.

**The app recipe:**

| App | Role |
|---|---|
| [TA-backup](https://splunkbase.splunk.com/app/10435) | Detects restic/borg (and b2/rclone via timers), reports configured jobs + stale timers |
| [TA-cronbeat](https://splunkbase.splunk.com/app/10437) | Every systemd timer and cron entry: total, stale, absent — the heartbeat layer |
| [TA-zfspool](https://splunkbase.splunk.com/app/10454) | The target pool itself: md/btrfs/ZFS state, corruption counters |
| [TA-disk](https://splunkbase.splunk.com/app/10429) | SMART on the backup target disks |

**Problems this kills:**
- Backups that exist in cron but haven't actually fired (cronbeat's stale-timer detection).
- Backup-to-same-pool disasters: backup "success" onto a degrading md array.
- Quietly unconfigured backup tooling (`configured=0` is a verdict).
- Restore-day corruption discovered for the first time by the restore itself.

## Killer searches

Per host: is there actually backup tooling wired up, and is anything stale?

```
index=main sourcetype=backup:status status!=configured latest=now
| table host status backup_timers timers_stale
| dedup host
```

Scheduled jobs that have gone zombie on you:

```
index=main sourcetype=cronbeat:status status!=ok latest=now
| table host status timers_stale timers_total
| dedup host
```

Backup pool health trend (md RAID per-array failed-device counters):

```
| mstats latest(zfspool.md_*_failed_devices) as failed* where index=metrics earliest=-15m by host
```

## Who this is for

Anyone who has ever been responsible for data and slept through a backup failure window — which is everyone.

## Next

[The Disk Failure Autopsy](disk-failure-autopsy.md) extends target-side storage coverage; [Storage Pool Integrity Watch](pool-integrity.md) continuously scrubs.
