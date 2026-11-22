# Node Fitness Check

**The scenario:** Before you pin the big training job, the backup restore, or the video transcode onto "that big box," the honest question is not "is it idle?" but "does it have *headroom right now*?" Idle boxes hide full stall queues, dying swap, and packages pinned at thermal limits. PSI plus a few trend metrics answers it in one screen and stops the "whose fault is the slowdown" game before it starts.

**The app recipe:**

| App | Question it answers |
|---|---|
| [TA-psi](https://splunkbase.splunk.com/app/10448) | Are tasks actually stalling on CPU, memory, or IO right now? |
| [TA-memwatch](https://splunkbase.splunk.com/app/10444) | Actual free memory and swap trajectory |
| [TA-throttle](https://splunkbase.splunk.com/app/10452) | Is the CPU already heat-stressed? |
| [TA-sensors](https://splunkbase.splunk.com/app/10385) | Package/VRM temperatures — the headroom behind the throttle |
| [TA-rapl](https://splunkbase.splunk.com/app/10449) | How much of the power budget is already consumed |

**Problems this kills:**
- "Schedule it on the idle node" that was 100 % IO-stalled.
- The two-heavy-jobs-at-once incident — a one-table go/no-go before you start.
- "The machine slowed down for no reason" — it never is.

## Killer searches

The go/no-go table — all three stall dimensions plus free memory, per host:

```
| mstats latest(psi.psi_io_full_avg10) as io_stall, latest(psi.psi_mem_full_avg10) as mem_stall, latest(psi.psi_cpu_full_avg10) as cpu_stall, latest(memwatch.mem_available_pct) as free_pct where index=metrics earliest=-15m by host
```

Heat check before you pile on:

```
| mstats latest(throttle.pkg_0_throttle_count) as throttle_events, latest(sensors.coretemp_package_id_0) as pkg_temp where index=metrics earliest=-60m by host
```

## Who this is for

ML engineers picking a training box, render-farm wranglers, and anyone who has ever said "it should have capacity" and been wrong.

## Next

If the table says heat: [Before the Heatwave](before-heatwave.md). If it says memory: [OOM Autopsy](oom-autopsy.md).
