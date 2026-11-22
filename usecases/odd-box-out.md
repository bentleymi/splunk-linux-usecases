# Find the Odd Box Out

**The scenario:** Three "identical" app servers, and one of them misbehaves. Every config-management story promises boxes are cattle; in reality drift accumulates in the gaps that your CM doesn't cover — plugged-in devices, auditd state, timer inventories, sssd domains, taint flags. The fix for config drift is comparison, and comparison needs comparable facts: the apps hash their domain of interest, and identical boxes should *agree*.

**The app recipe:**

| App | The fingerprint it contributes |
|---|---|
| [TA-devfleet](https://splunkbase.splunk.com/app/10439) | PCI + USB inventory signatures (md5 of `lspci`/`lsusb`) |
| [TA-baseline](https://splunkbase.splunk.com/app/10436) | Secure Boot / lockdown / sshd / shadow / auditd states |
| [TA-cronbeat](https://splunkbase.splunk.com/app/10437) | Timer + cron job counts and stale marks |
| [TA-authid](https://splunkbase.splunk.com/app/10433) | nsswitch chain, SSSD/nscd/Kerberos states |
| [TA-kernel](https://splunkbase.splunk.com/app/10443) | Taint flags (the "what did you load into this kernel" fingerprint) |

**Problems this kills:**
- The "clone" that isn't a clone.
- Security settings that drifted on exactly production's busiest node.
- The host that "shouldn't need it" and turns out to be the only one with auditd alive… or dead.

## Killer searches

Cluster the fleet by hardware fingerprint — hosts sharing one PCI signature are clones; lone signatures are the odd ones:

```
index=main sourcetype=devfleet:status earliest=-2h latest=now
| stats count as runs values(host) as hosts by pci_signature
| sort -runs
```

USB fingerprints tell the "what's plugged into each box" story:

```
index=main sourcetype=devfleet:status earliest=-2h latest=now
| stats values(host) as hosts by usb_signature
```

Security-posture comparison table — eyeball the disagreeing cells:

```
index=main sourcetype=baseline:status latest=now
| table host secure_boot lockdown weak_ssh password_auth auditd_active
| dedup host
```

## Who this is for

MSPs with "standard builds," anyone running three copies of anything, and auditors who suspect the golden image drifted.

## Next

Once you find the odd box, [The Night Before the Audit](patch-audit-eve.md) documents it and [Who Broke the Box?](change-trails.md) explains how it happened.
