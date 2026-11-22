# Blame It on the Network (Prove It)

**The scenario:** "The app is slow." The developer blames the network; the network person blames the app; the database blames everyone. Meanwhile the truth is one of: NIC link retraining to half-duplex at midnight, an interface dropping 3 % of packets, IO stall from a noisy neighbor, or OOM swap-thrash. Every one of those has telemetry — you just have to have the four apps that read it.

**The app recipe:**

| App | Role |
|---|---|
| [TA-netqual](https://splunkbase.splunk.com/app/10446) | Per-interface operstate, carrier/speed, rx/tx errors + drops, link-flap events |
| [TA-psi](https://splunkbase.splunk.com/app/10448) | Pressure Stall Information: is CPU/memory/IO actually stalling tasks |
| [TA-memwatch](https://splunkbase.splunk.com/app/10444) | Free memory, swap-in activity (the signature of thrash, not slowness) |
| [TA-throttle](https://splunkbase.splunk.com/app/10452) | CPU frequency collapse / thermal throttle (the other "slow" story) |

**Problems this kills:**
- "Network's fine, it's your app" arguments, settled in one screenshot.
- Link flap history — the flap that aligns perfectly with the outage window.
- "Slow application" that is actually IO-stalled on a saturated pool (PSI `full` metrics don't lie).
- The NIC that "works" while incrementing rx_errors at 40/s.

## Killer searches

Is anything actually stalling? PSI + memory headroom per host:

```
| mstats latest(psi.psi_io_full_avg10) as io_stall, latest(psi.psi_mem_some_avg10) as mem_pressure where index=metrics earliest=-15m by host
```

Hosts with real link flaps since the app was started (netqual's flap ring buffer):

```
index=main sourcetype=netqual:status latest=now
| spath input=_raw output=flaps path=link_flaps{}
| where mvcount(flaps)>0
| table host flaps
| dedup host
```

Interfaces that are down-but-were-expected (carrier=0):

```
index=main sourcetype=netqual:status latest=now
| spath input=_raw output=carr path=ifaces{}.carrier
| search carr=0
| table host
| dedup host
```

## Who this is for

SREs, homelab network engineers, and anyone who has had to defend a switch against an application developer with confidence.

## Next

When PSI says memory, hand off to [the Memory Watchdog page in fleet health](fleet-health-board.md); when it says IO, [Storage Pool Integrity Watch](pool-integrity.md).
