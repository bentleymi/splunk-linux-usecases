# Kernel Crash Readiness

**The scenario:** When the box panics at 3 a.m., there are two kinds of operators: the one who gets a vmcore and a root cause, and the one who gets a reboot and a story. kdump is "configured" until /crash is full or nothing ever loaded the capture kernel — and you will not discover that in the crash itself. Taint flags, oops floods, and journald backlogs have been predicting the panic for weeks, if anything was reading them.

**The app recipe:**

| App | Role |
|---|---|
| [TA-kernel](https://splunkbase.splunk.com/app/10443) | kdump service state, kexec capture-kernel readiness, taint flag decode, new oops/bug/panic, uptime |
| [TA-psi](https://splunkbase.splunk.com/app/10448) | Memory-pressure stall — OOM-panic precursor |
| [TA-memwatch](https://splunkbase.splunk.com/app/10444) | Free memory trend, OOM kill counters (the crash that was *not* the kernel's fault) |
| [TA-journald](https://splunkbase.splunk.com/app/10457) | Journald health: disk usage, file growth — is the crash even going to be logged |

**Problems this kills:**
- "kdump was enabled" that was never actually kexec-loaded (the metric is `kexec_crash_loaded`, not a checkbox).
- Tainted-kernel tickets from out-of-tree drivers arriving as evidence, not anecdotes.
- OOM kills that were blamed on a kernel bug when memory was simply gone.
- journald having eaten the last 10 minutes of context because it was out of disk.

## Killer searches

Per host — readiness now, new panics/oops/bugs in the last 7 days:

```
| mstats latest(kernel.kexec_crash_loaded) as kdump_wired, latest(kernel.panic_new) as new_panics, latest(kernel.oops_new) as new_oops, latest(kernel.bug_new) as new_bugs where index=metrics earliest=-7d by host
```

Memory stall trend (an OOM panic in waiting looks exactly like this):

```
| mstats latest(psi.psi_mem_some_avg10) as mem_pressure where index=metrics earliest=-15m by host
```

Journald disk footprint — will it survive the crash it needs to record?

```
index=main sourcetype=journald:status latest=now
| table host status size_mb file_count service_active
| dedup host
```

## Who this is for

Kernel/driver developers, hardware-provisioning teams, and any ops lead who is tired of "it just rebooted" being the whole incident timeline.

## Next

Pair with [Blame It on the Network](prove-network-blame.md) — most "kernel panics" are memory pressure pretending.
