# The One-Query Incident Timeline

**The scenario:** 02:47, PagerDuty-ish noise starts. The usual response is seventeen tabs: dmesg here, SMART there, "did the network flap or did the service crash first?" Every add-on in this collection writes its observations as timestamped JSON events with a common `status` convention — which means the whole incident timeline is one search, not a scavenger hunt.

**The app recipe:** every add-on in the collection contributes one `:status` event stream sharing the same contract (`status=ok|alarm|absent|none`, host-qualified, minute-granular).

| Layer | Apps |
|---|---|
| Storage | [disk](https://splunkbase.splunk.com/app/10429) · [zfspool](https://splunkbase.splunk.com/app/10454) · [hwraid](https://splunkbase.splunk.com/app/10441) · [sanpath](https://splunkbase.splunk.com/app/10450) |
| Compute | [kernel](https://splunkbase.splunk.com/app/10443) · [memwatch](https://splunkbase.splunk.com/app/10444) · [psi](https://splunkbase.splunk.com/app/10448) · [throttle](https://splunkbase.splunk.com/app/10452) |
| Network | [netqual](https://splunkbase.splunk.com/app/10446) |
| Services | [service](https://splunkbase.splunk.com/app/10451) · [cronbeat](https://splunkbase.splunk.com/app/10437) · [backup](https://splunkbase.splunk.com/app/10435) |
| Environment | [ups](https://splunkbase.splunk.com/app/10453) · [rapl](https://splunkbase.splunk.com/app/10449) · [sensors](https://splunkbase.splunk.com/app/10385) · [amdgpu](https://splunkbase.splunk.com/app/10431) |
| Governance | [patch](https://splunkbase.splunk.com/app/10447) · [baseline](https://splunkbase.splunk.com/app/10436) · [authid](https://splunkbase.splunk.com/app/10433) · [devfleet](https://splunkbase.splunk.com/app/10439) · [journald](https://splunkbase.splunk.com/app/10457) |

**Problems this kills:**
- "Which subsystem broke first?" answerable in one graph.
- Arguing about causation between teams when a common timestamped record exists.
- Incident review meetings that spend 40 minutes just gathering facts.

## Killer searches

Everything that wasn't okay on any box, last 24 hours — the raw incident feed:

```
index=main sourcetype="*:status" status!=ok status!=absent status!=none earliest=-24h latest=now
| table _time host sourcetype status
| head 30
```

The incident timeline itself — alarms per subsystem per hour:

```
index=main sourcetype="*:status" status!=ok status!=absent status!=none earliest=-24h latest=now
| timechart span=1h count by sourcetype
```

The failed-services layer with unit names (the usual first domino):

```
index=main sourcetype=service:status failed_count>0 latest=now
| table host failed_count failed_units units_total
| dedup host
```

## Who this is for

On-call engineers, MSP NOCs, and homelabbers who want a real incident timeline without buying SOAR.

## Next

Drill into the culprit's own dashboard, or follow the thread to [OOM Autopsy](oom-autopsy.md), [Reboot Archaeologist](reboot-archaeologist.md), or [Blame It on the Network](prove-network-blame.md).
