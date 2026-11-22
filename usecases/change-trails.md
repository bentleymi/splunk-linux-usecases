# Who Broke the Box?

**The scenario:** The server is acting strange and nobody changed anything — which is true at the human level and false at the physical level. Someone plugged a Wi-Fi adapter into the rack PDU switch port's management VLAN. A cron entry appeared in root's crontab. `PermitRootLogin` quietly became `yes`. The audit log you needed did not exist because nothing was watching for change. These apps *are* the watchers, and the deltas they emit are timeline evidence on your side.

**The app recipe:**

| App | Role |
|---|---|
| [TA-devfleet](https://splunkbase.splunk.com/app/10439) | PCI/USB inventory signatures + device_delta events — physical config drift |
| [TA-baseline](https://splunkbase.splunk.com/app/10436) | Security settings drift: Secure Boot, sshd config, shadow policy, auditd |
| [TA-authid](https://splunkbase.splunk.com/app/10433) | Identity path drift: nsswitch order, SSSD state, Kerberos ticket caches |
| [TA-cronbeat](https://splunkbase.splunk.com/app/10437) | Scheduled-job inventory — the crontab you never approved |
| [TA-journald](https://splunkbase.splunk.com/app/10457) | Log-plane health: is the evidence being collected at all |

**Problems this kills:**
- "Nothing changed" investigations — device deltas put a timestamp on "something changed."
- Rogue USB mass storage appearing on a locked-down box.
- Security posture degrading through config drift rather than an incident.
- The discovery window for compromised identity configs collapsing from "months" to "next interval."

## Killer searches

Every physical device insertion/removal on any box, last 7 days (new vs removed per host):

```
index=main sourcetype=devfleet:status device_delta>0 earliest=-7d latest=now
| table _time host device_delta new_devices removed_devices
| head 20
```

All change-relevant alarms across every change app in one table:

```
index=main sourcetype=baseline:status OR sourcetype=authid:status OR sourcetype=cronbeat:status
  status=alarm latest=now
| table host sourcetype
| dedup host sourcetype
```

Timer/jobs inventory drift (the unexpected scheduled task):

```
index=main sourcetype=cronbeat:status latest=now
| table host status cron_jobs timers_total timers_stale
| dedup host
```

## Who this is for

Security-minded sysadmins, incident responders with no SIEM budget, and anyone who has ever asked "when did that USB drive appear?" to an empty room.

## Next

Turn the evidence into artifacts with [The Night Before the Audit](patch-audit-eve.md); correlate crashes with physical changes via [Kernel Crash Readiness](crash-ready.md).
