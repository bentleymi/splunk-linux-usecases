# The 3am OOM Autopsy

**The scenario:** The database disappeared at 3:12am. `grep oom` in the morning says the kernel shot it — the only question is whether it was murdered (leak, runaway) or executed (the box was simply out of memory for ten minutes and the kernel finally picked a victim). The OOM killer's choice is the last sentence of the story, not the first; what you need is the minute-by-minute pressure that preceded it, which the kernel emits as PSI before it ever draws blood.

**The app recipe:**

| App | Role |
|---|---|
| [TA-memwatch](https://splunkbase.splunk.com/app/10444) | Free-memory trend, swap-in/out counters, OOM-kill totals — the crime scene |
| [TA-psi](https://splunkbase.splunk.com/app/10448) | Memory full/some pressure: the suffering that precedes each kill |
| [TA-journald](https://splunkbase.splunk.com/app/10457) | The OOM report itself lives in journald — make sure it survives |

**Problems this kills:**
- "It crashed" vs. "what starved, for how long, and how many times has this happened."
- OOM victim selection blamed on the kernel; the trend shows the 10-minute memory collapse before the kill.
- The journald vacuum that ate the actual OOM report when the pressure spiked.

## Killer searches

Per host: OOM kill total, swap-out total, and how bad memory got:

```
| mstats latest(memwatch.oom_kill_total) as oom_kills, latest(memwatch.pswpout_total) as swap_out, latest(psi.psi_mem_full_avg10) as mem_stall where index=metrics earliest=-7d by host
```

The pre-mortem signature: watch `mem_stall` climb toward `oom_kills` in the same window — paste both panels side by side.

## Who this is for

Database and JVM people (Java's OOM reputation), anyone whose alerts currently arrive *after* the kill, and every team with an unidentified memory leak in production.

## Next

[Kernel Crash Readiness](crash-ready.md) for the panicing variety; [Node Fitness Check](node-fitness.md) to catch pressure before scheduling onto the box.
