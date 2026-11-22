# The Night Before the Audit

**The scenario:** Compliance lands tomorrow — SOC 2, CIS, cyber-insurance questionnaire, or the annual "prove you patch" review. The auditor wants the current patch state and config posture of every box, timestamped, not someone's recollection from the last maintenance window. Reconstructing that evidence from scattered `apt list --upgradable` screenshots is how audits become weekends.

**The app recipe:**

| App | Role |
|---|---|
| [TA-patch](https://splunkbase.splunk.com/app/10447) | apt/dnf upgradable counts, security-only counts, days-since-last-update |
| [TA-baseline](https://splunkbase.splunk.com/app/10436) | Secure Boot, kernel lockdown, sshd root login/password auth, shadow age policy, auditd |
| [TA-authid](https://splunkbase.splunk.com/app/10433) | Identity plumbing health: nsswitch chain, SSSD state + domains, Kerberos tickets, nscd |
| [TA-kernel](https://splunkbase.splunk.com/app/10443) | Running-kernel taint + uptime (kernel staleness evidence) |

**Problems this kills:**
- The "we patch monthly" claim that dies the moment the oldest host shows 180 days.
- Secure Boot off / root login yes / auditd dead on one forgotten box.
- LDAP/AD authentication quietly falling back to local cache (authid sees SSSD go stale before users do).
- Auditors asking "when did this change?" — every app timestamps its own history, forever queryable.

## Killer searches

Patch posture per host — this is the evidence table:

```
| mstats latest(patch.upgradable) as pkgs, latest(patch.security_upgradable) as security_pkgs, latest(patch.days_since_update) as days_since_update where index=metrics earliest=-24h by host
```

Every security-baseline alarm across the fleet (password auth, root login, shadow aging, auditd):

```
index=main sourcetype=baseline:status status=alarm latest=now
| table host auditd_active lockdown password_auth permit_root_login secure_boot shadow_max_age weak_ssh
| dedup host
```

Uptime as kernel-staleness evidence (a 300-day uptime is an unpatched kernel story):

```
index=main sourcetype=kernel:status latest=now
| stats avg(uptime_s) as avg_uptime_s by host
```

## Who this is for

The one sysadmin who is also the "compliance person," small teams selling into compliance-heavy verticals, and Managed Service Providers who need per-client evidence packets without a GRC platform.

## Next

Change-control trails with [Who Broke the Box?](change-trails.md); crash readiness for the "resilience" section of the questionnaire: [Kernel Crash Readiness](crash-ready.md).
