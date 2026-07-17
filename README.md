# AD-Homelab-Portfolio

A self-directed Windows Server 2022 / Active Directory homelab built end-to-end in Oracle
VirtualBox — domain deployment, client onboarding, GPO administration, DNS/DHCP services,
NTFS/Share role-based access, simulated Tier 1 help desk tickets, and a ServiceNow ITSM
workflow. Documented phase-by-phase with screenshots and notes below.

## Overview

The lab simulates a small company domain (`almuina.local`) with one domain controller and
two department-segmented client workstations. Each phase below maps to a specific,
resume-relevant IT support or sysadmin skill — from standing up the domain itself through
diagnosing help desk tickets and working incidents in a real ITSM platform.

## Tech Stack

- **Server OS:** Windows Server 2022 (Desktop Experience) — AD DS, DNS, DHCP, Group Policy
- **Client OS:** Windows 10/11 Pro
- **Scripting:** PowerShell (New-ADUser, New-ADGroup, Add-ADGroupMember, CSV bulk import)
- **Virtualization:** Oracle VirtualBox (NAT Network)
- **ITSM:** ServiceNow (Personal Developer Instance)

## Repo Structure

```
AD-Homelab-Portfolio/
├── 01-domain-setup/
├── 02-client-join/
├── 03-user-management/
├── 04-helpdesk-tickets/
├── 05-permissions/
├── 06-network-services/
├── 07-gpo-backup/
├── 08-ticketing-system/
└── README.md
```

## Known Lab Shortcuts

A couple of deliberate simplifications, called out here rather than left for someone to
discover:

- Administered throughout as the built-in `Administrator` account, rather than a dedicated
  least-privilege admin account — a known lab shortcut, not a production practice.
- All lab user accounts share one password, for convenience while documenting screenshots —
  not a production practice.

## Connect

LinkedIn: www.linkedin.com/in/thiago-almuina  Resume: [add a link or "available on request"]

---

## Phase 1 — Domain Controller Deployment

Deployed Windows Server 2022 (Desktop Experience) as DC01 in VirtualBox with a static IP
(192.168.100.10) and promoted it to a domain controller for a new forest, almuina.local.
Built a 5-OU structure (IT, Sales, HR, Servers, Workstations with IT/Sales sub-OUs), created
3 security groups (GG-IT, GG-Sales, GG-HR), and provisioned 5 test users distributed across
their correct OUs and group memberships. Verified the domain deployment via Get-ADDomain and
Get-ADForest since the live promotion wizard screen couldn't be recaptured after the fact.

**Screenshots:** see `01-domain-setup/`

## Phase 2 — Client Domain Join

Joined WS01 and WS02 to the almuina.local domain, pointing each client's DNS to DC01
(192.168.100.10) beforehand. Signed in on each client with a distinct domain account —
ana.martinez on WS01, sofia.ramirez on WS02 — and verified group membership with
whoami /groups, confirming ALMUINA\GG-IT and ALMUINA\GG-Sales respectively. Learned that
group membership changes don't apply to an already-open session — a fresh logoff/logon is
required for Kerberos to pick up new group tokens. Confirmed both computers registered in
AD via ADUC on DC01.

**Screenshots:** see `02-client-join/`

## Phase 3 — ADUC & PowerShell Administration

Administered Active Directory using both the GUI and PowerShell: edited an existing user's
attributes in ADUC, then created a new user (elena.vargas) and a new security group
(GG-Helpdesk) via New-ADUser/New-ADGroup, verifying membership with Add-ADGroupMember and
Get-ADGroupMember. Built a CSV-driven bulk-creation script using Import-Csv + New-ADUser to
provision three additional users across the IT, Sales, and HR OUs in a single pass, and
confirmed correct OU placement for each with Get-ADUser.

**Screenshots:** see `03-user-management/`

## Phase 4 — Simulated Help Desk Tickets

Diagnosed and resolved 5 simulated help desk tickets split across WS01 and WS02, covering
account lockout, password reset, incorrect group membership, disabled account, and forced
password change scenarios. Used a mix of ADUC and PowerShell (Get-ADUser, Enable-ADAccount,
Remove/Add-ADGroupMember) to diagnose each issue and verify resolution, documenting each
ticket with a before → diagnosis → action → after workflow in a tracking spreadsheet.

**Screenshots:** see `04-helpdesk-tickets/`
**Ticket log:** see `04-helpdesk-tickets/helpdesk-tickets.xlsx`

## Phase 5 — NTFS/Share Permissions & Role-Based Access

Created a shared folder (\\DC01\Finance) and enforced role-based access by assigning the
GG-Finance-Share security group — never individual users — to both NTFS (Modify) and Share
(Change) permissions, removing the default Everyone entry. Mapped the network drive from
both clients: WS01 (ana.martinez, a GG-Finance-Share member) successfully accessed and read
the shared file, while WS02 (sofia.ramirez, not a member) received an explicit
"You do not have permission to access" error — confirming the access control worked exactly
as designed for both the correct and incorrect user.

**Screenshots:** see `05-permissions/`

## Phase 6 — Network Services: DNS & DHCP

Installed and authorized the DHCP role on DC01, then created a scope (Homelab-Clients,
192.168.100.100–150) with gateway and DNS options configured. Migrated WS01 and WS02 from
static to dynamic IPs, confirming two distinct leases in the DHCP console and DHCP Enabled:
Yes on both clients via ipconfig /all. Created a DNS A record (fileserver) and a CNAME alias
(files) in the almuina.local zone, and validated end-to-end name resolution with nslookup
and ping from WS02. Also observed dynamic DNS registration automatically creating host
records for both clients after their DHCP lease renewal.

**Screenshots:** see `06-network-services/`

## Phase 7 — GPOs, Remote Desktop & System State Backup

Segmented policy administration by placing WS01 in IT-Workstations and WS02 in
Sales-Workstations, then built and linked two OU-specific GPOs: a password policy
(10-character minimum) for IT, and a Control Panel restriction for Sales — the latter
requiring User Group Policy Loopback Processing (Merge mode) since the restricted user
account lives in a different OU than the computer. Verified correct policy segmentation
via gpupdate /force and gpresult /h on both clients, confirming each applied only its own
GPO. Configured Remote Desktop access via a dedicated security group, discovering along the
way that Domain Controllers don't honor local Remote Desktop Users membership — access is
instead governed by the "Allow log on through Remote Desktop Services" right in the Default
Domain Controllers Policy. Completed a System State backup using Windows Server Backup for
recovery readiness.

**Screenshots:** see `07-gpo-backup/`

## Phase 8 (Bonus) — Ticketing System: ServiceNow

Provisioned a ServiceNow Personal Developer Instance and configured Incident Management to
simulate a real IT service desk workflow. Customized the Category field with VPN and
Account & Permissions options, created user records reflecting the AD homelab's test users,
and logged 10 incidents spanning account lockouts, password resets, group membership errors,
hardware requests, and VPN issues. Worked incidents through their full lifecycle — resolving
several with documented resolution notes, escalating one to a Tier 2 assignment group, and
leaving others in progress — while observing how ServiceNow auto-calculates Priority from
Impact × Urgency rather than a manual field, mirroring real-world ITSM prioritization logic.

**Screenshots:** see `08-ticketing-system/`
