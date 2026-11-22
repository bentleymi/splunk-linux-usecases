# GPU Fleet Watch

**The scenario:** Six AMD boxes are training something and the run is "20 % slower today." Is a card thermal-throttling at 92 °C? Is the PCIe slot negotiation at x4 instead of x16? Did system RAM swap so hard the GPU starved? Is the package cooking at 180 W while the workload asks for 80? On consumer boards with seven GPUs there's no BMC, no datacenter telemetry — the truth is in sysfs, and AMD exposes all of it.

**The app recipe:**

| App | Role |
|---|---|
| [TA-amdgpu](https://splunkbase.splunk.com/app/10431) | AMD GPU telemetry: per-card state from `/sys/class/drm` + KFD topology |
| [TA-throttle](https://splunkbase.splunk.com/app/10452) | CPU package frequency/thermal throttle counters |
| [TA-memwatch](https://splunkbase.splunk.com/app/10444) | RAM headroom, swap thrash (GPU memory pressure is almost always host RAM) |
| [TA-psi](https://splunkbase.splunk.com/app/10448) | IO stall — the dataloader-starved-GPU signature |
| [TA-rapl](https://splunkbase.splunk.com/app/10449) | Chip power, so heat and perf can be correlated |

**Problems this kills:**
- "Run 4 is slow" root-caused to a card hot enough to clock down.
- Dead KFD / no-render-node boxes discovered before the training job dies.
- Host RAM exhaustion that gets misdiagnosed as GPU failure, every single time.
- Silent PCIe degradation (counters trend; topology tells you).

## Killer searches

GPU presence + system pressure in one row per host:

```
| mstats latest(amdgpu.gpu_count) as gpus, latest(amdgpu.kfd) as kfd_nodes, latest(memwatch.mem_available_pct) as free_mem where index=metrics earliest=-15m by host
```

IO stall + throttle + power for a slow "GPU" run (the GPU is rarely the culprit — prove it):

```
| mstats latest(psi.psi_io_full_avg10) as io_stall, latest(throttle.pkg_0_throttle_count) as throttle_count, latest(rapl.powercap_count) as powercaps where index=metrics earliest=-60m by host
```

Card inventory events per host (topology drift across reboots):

```
index=main sourcetype=amdgpu:status latest=now
| table host gpu_count kfd status
| dedup host
```

## Who this is for

AI/ML operators running AMD rigs, homelab inference farms, and anyone who has ever had to tell a researcher "it's not your code."

## Next

Heat stories lead to [Power, Thermal & the UPS](power-thermal-ups.md); memory stories to [Blame It on the Network](prove-network-blame.md).
