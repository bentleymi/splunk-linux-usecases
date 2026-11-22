# Splunk Linux Use Cases — App Recipes That Ship Together

The 20 [Linux monitoring add-ons in this collection](https://github.com/bentleymi/splunk-linux-usecases) do their best work in **combination**. Each add-on is a specialist; each use case below is a crew of specialists pointing at one real-world problem, with ready-to-paste searches on live data.

Pick a scenario, install the listed apps, paste the searches into a dashboard, done.

## The use cases

| | Use case | The problem it kills | Apps |
|---|---|---|---|
| 1 | [The Disk Failure Autopsy](usecases/disk-failure-autopsy.md) | "How did I not see that drive dying?" | [disk](https://splunkbase.splunk.com/app/10429) · [zfspool](https://splunkbase.splunk.com/app/10454) · [hwraid](https://splunkbase.splunk.com/app/10441) · [sanpath](https://splunkbase.splunk.com/app/10450) |
| 2 | [The 3-Boxes-in-a-Closet Health Board](usecases/fleet-health-board.md) | Homelab / small-fleet situational awareness | [memwatch](https://splunkbase.splunk.com/app/10444) · [netqual](https://splunkbase.splunk.com/app/10446) · [service](https://splunkbase.splunk.com/app/10451) · [cronbeat](https://splunkbase.splunk.com/app/10437) · [kernel](https://splunkbase.splunk.com/app/10443) · [patch](https://splunkbase.splunk.com/app/10447) · [throttle](https://splunkbase.splunk.com/app/10452) · [disk](https://splunkbase.splunk.com/app/10429) |
| 3 | [The Night Before the Audit](usecases/patch-audit-eve.md) | Patch gaps + config drift, with evidence | [patch](https://splunkbase.splunk.com/app/10447) · [baseline](https://splunkbase.splunk.com/app/10436) · [authid](https://splunkbase.splunk.com/app/10433) · [kernel](https://splunkbase.splunk.com/app/10443) |
| 4 | [Backups You Can Testify Under](usecases/backup-warrant.md) | Proving backups actually ran | [backup](https://splunkbase.splunk.com/app/10435) · [cronbeat](https://splunkbase.splunk.com/app/10437) · [zfspool](https://splunkbase.splunk.com/app/10454) · [disk](https://splunkbase.splunk.com/app/10429) |
| 5 | [Blame It on the Network (Prove It)](usecases/prove-network-blame.md) | 3am "the app is slow" — who's lying? | [netqual](https://splunkbase.splunk.com/app/10446) · [psi](https://splunkbase.splunk.com/app/10448) · [memwatch](https://splunkbase.splunk.com/app/10444) · [throttle](https://splunkbase.splunk.com/app/10452) |
| 6 | [Power, Thermal & the UPS That Lied](usecases/power-thermal-ups.md) | Watts, throttling, and battery truth | [ups](https://splunkbase.splunk.com/app/10453) · [rapl](https://splunkbase.splunk.com/app/10449) · [throttle](https://splunkbase.splunk.com/app/10452) · [sensors](https://splunkbase.splunk.com/app/10385) |
| 7 | [Kernel Crash Readiness](usecases/crash-ready.md) | Being ready *before* the panic | [kernel](https://splunkbase.splunk.com/app/10443) · [psi](https://splunkbase.splunk.com/app/10448) · [memwatch](https://splunkbase.splunk.com/app/10444) · [journald](https://splunkbase.splunk.com/app/10457) |
| 8 | [GPU Fleet Watch](usecases/gpu-fleet.md) | AI rigs: thermals, throttle, memory | [amdgpu](https://splunkbase.splunk.com/app/10431) · [throttle](https://splunkbase.splunk.com/app/10452) · [memwatch](https://splunkbase.splunk.com/app/10444) · [psi](https://splunkbase.splunk.com/app/10448) · [rapl](https://splunkbase.splunk.com/app/10449) |
| 9 | [Storage Pool Integrity Watch](usecases/pool-integrity.md) | Silent pool rot (ZFS / md / btrfs) | [zfspool](https://splunkbase.splunk.com/app/10454) · [sanpath](https://splunkbase.splunk.com/app/10450) · [disk](https://splunkbase.splunk.com/app/10429) · [hwraid](https://splunkbase.splunk.com/app/10441) |
| 10 | [Who Broke the Box?](usecases/change-trails.md) | Change & identity forensic trails | [devfleet](https://splunkbase.splunk.com/app/10439) · [baseline](https://splunkbase.splunk.com/app/10436) · [authid](https://splunkbase.splunk.com/app/10433) · [cronbeat](https://splunkbase.splunk.com/app/10437) · [journald](https://splunkbase.splunk.com/app/10457) |
| 11 | [The One-Query Incident Timeline](usecases/incident-timeline.md) | One search = the whole incident narrative | every app in the collection |
| 12 | [Find the Odd Box Out](usecases/odd-box-out.md) | Fleet divergence via fingerprints | [devfleet](https://splunkbase.splunk.com/app/10439) · [baseline](https://splunkbase.splunk.com/app/10436) · [cronbeat](https://splunkbase.splunk.com/app/10437) · [authid](https://splunkbase.splunk.com/app/10433) · [kernel](https://splunkbase.splunk.com/app/10443) |
| 13 | [The 3am OOM Autopsy](usecases/oom-autopsy.md) | What really killed that process | [memwatch](https://splunkbase.splunk.com/app/10444) · [psi](https://splunkbase.splunk.com/app/10448) · [journald](https://splunkbase.splunk.com/app/10457) |
| 14 | [NVMe End-of-Life Planner](usecases/nvme-eol-planner.md) | Replace drives before they fail | [disk](https://splunkbase.splunk.com/app/10429) · [zfspool](https://splunkbase.splunk.com/app/10454) · [sanpath](https://splunkbase.splunk.com/app/10450) |
| 15 | [The Reboot Archaeologist](usecases/reboot-archaeologist.md) | Planned vs. spontaneous reboots, proven | [kernel](https://splunkbase.splunk.com/app/10443) · [patch](https://splunkbase.splunk.com/app/10447) · [cronbeat](https://splunkbase.splunk.com/app/10437) · [service](https://splunkbase.splunk.com/app/10451) |
| 16 | [Node Fitness Check](usecases/node-fitness.md) | Go/no-go before the heavy workload | [psi](https://splunkbase.splunk.com/app/10448) · [memwatch](https://splunkbase.splunk.com/app/10444) · [throttle](https://splunkbase.splunk.com/app/10452) · [sensors](https://splunkbase.splunk.com/app/10385) · [rapl](https://splunkbase.splunk.com/app/10449) |
| 17 | [Before the Heatwave](usecases/before-heatwave.md) | Thermal headroom audit pre-summer | [sensors](https://splunkbase.splunk.com/app/10385) · [throttle](https://splunkbase.splunk.com/app/10452) · [rapl](https://splunkbase.splunk.com/app/10449) · [disk](https://splunkbase.splunk.com/app/10429) |

## How it works

Every add-on ships a lightweight Python collector (`bin/<name>_collector.py`) that reads straight from `/proc`, `/sys`, and the usual CLI tools — no agents, no daemons, no external dependencies. Splunk's `script://` input runs it; each app auto-builds an index-`metrics` series plus a `:status` JSON event, with its own dashboard out of the box.

All searches on these pages are tested on live multi-host data. Time range is deliberately specified per-search (`earliest=-15m` style) — run them anywhere, get answers.

## All 21 add-ons

| App | Splunkbase |
|---|---|
| Linux Disk SMART & NVMe Health | [TA-disk](https://splunkbase.splunk.com/app/10429) |
| AMD GPU Telemetry | [TA-amdgpu](https://splunkbase.splunk.com/app/10431) |
| Identity Client (nsswitch/SSSD/Kerberos) | [TA-authid](https://splunkbase.splunk.com/app/10433) |
| Backup Job Assurance (restic/borg/b2/rclone) | [TA-backup](https://splunkbase.splunk.com/app/10435) |
| Security Baseline (SecureBoot/sshd/auditd) | [TA-baseline](https://splunkbase.splunk.com/app/10436) |
| Scheduled Job Heartbeat (timers/cron) | [TA-cronbeat](https://splunkbase.splunk.com/app/10437) |
| Device Inventory Delta (PCI/USB) | [TA-devfleet](https://splunkbase.splunk.com/app/10439) |
| Hardware RAID Controller State | [TA-hwraid](https://splunkbase.splunk.com/app/10441) |
| Kernel Crash-Readiness | [TA-kernel](https://splunkbase.splunk.com/app/10443) |
| Memory Watchdog (leak/swap/OOM) | [TA-memwatch](https://splunkbase.splunk.com/app/10444) |
| Network Link Quality | [TA-netqual](https://splunkbase.splunk.com/app/10446) |
| Patch Compliance (apt/dnf) | [TA-patch](https://splunkbase.splunk.com/app/10447) |
| Pressure Stall Information | [TA-psi](https://splunkbase.splunk.com/app/10448) |
| Power & Energy (RAPL + hwmon) | [TA-rapl](https://splunkbase.splunk.com/app/10449) |
| Storage Path Health (multipath/iSCSI) | [TA-sanpath](https://splunkbase.splunk.com/app/10450) |
| Systemd Service Health | [TA-service](https://splunkbase.splunk.com/app/10451) |
| CPU Frequency & Throttle | [TA-throttle](https://splunkbase.splunk.com/app/10452) |
| UPS Health (NUT/apcupsd) | [TA-ups](https://splunkbase.splunk.com/app/10453) |
| Journald Log-Plane Meta | [TA-journald](https://splunkbase.splunk.com/app/10457) |
| Linux Storage Pool Integrity | [TA-zfspool](https://splunkbase.splunk.com/app/10454) |
| Hardware Sensors | [TA-sensors](https://splunkbase.splunk.com/app/10385) |

*Built by [michael@bentleypc.com](mailto:michael@bentleypc.com). Source for every app lives on GitHub under `bentleymi/TA-<slug>`.*
