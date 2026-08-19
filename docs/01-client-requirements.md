# Client Requirements — Seshego Roadworks & Paving (Kimberley)

**Project ID:** CMPG325-2026-077 &nbsp;|&nbsp; **Client ID:** CLI-077 &nbsp;|&nbsp; **Industry:** Construction

## 1. Purpose

Design, simulate, and document a LAN for Seshego Roadworks & Paving that provides
department-level segmentation, inter-VLAN routing via Router-on-a-Stick, and
controlled access to a new application/file server (CR4), while leaving
capacity for a new department next year (Design Constraint).

## 2. Assumed Departments

The brief does not name specific departments, so the following are reasoned
assumptions based on a small-to-medium roadworks/paving contractor, documented
here as required.

| Dept | VLAN | Assumed headcount | Rationale |
|---|---|---|---|
| Management & Administration | 10 | ~20 | Executive management, HR, general admin/reception — every construction firm needs a central admin function |
| Engineering & Design | 20 | ~25 | Civil/road design engineers, draughtspersons, surveyors — the core technical function for a roadworks contractor, largest projected headcount |
| Site Operations & Projects | 30 | ~15 | Office-based project coordinators/supervisors who liaise with field crews and site trailers |
| Finance & Procurement | 40 | ~10 | Accounts, payroll, supplier and materials procurement |
| Servers | 50 | ~5 hosts | Houses the new application/file server (CR4) and future shared services (e.g. print server) |
| Future Department (reserved) | 60 | 0 now, ~20 reserved | Pre-provisioned to satisfy the stated Design Constraint ("a new department will be added next year") without renumbering the network later |
| Network Management | 99 | ~6 devices | Native/management VLAN for switch and router management interfaces — kept off the data VLANs for security |

## 3. CR4 — New Application/File Server

> "A new application/file server is installed and must be reachable by
> authorised departments only."

**Assumption:** the server holds project drawings, contracts and financial
records, so the departments that need day-to-day access are:

- Management & Administration (VLAN 10) — company records
- Engineering & Design (VLAN 20) — project drawings/documents
- Finance & Procurement (VLAN 40) — contracts, invoices

**Site Operations (VLAN 30) is treated as unauthorised** for direct server
access — site coordinators receive documents relayed by Engineering/Admin
rather than connecting to the server directly. This is a deliberate
assumption to keep the ACL demonstrable and defensible (fewer authorised
sources, clearer test cases for "permit" vs "deny").

This will be enforced with an extended ACL on the server's Router-on-a-Stick
sub-interface (VLAN 50), permitting VLANs 10/20/40 and denying all other
traffic to the server subnet.

## 4. Design Constraint — Future Department

VLAN 60 and a matching subnet are reserved and pre-configured on the router
and core switch (VLAN created, sub-interface configured, no access ports
assigned yet) so that onboarding the new department next year requires no
re-addressing — just activating access ports on an access/distribution
switch.

## 5. Client Requirements → Design Response

| Requirement | Design response |
|---|---|
| Segment departments | One VLAN per department |
| Assigned addressing block 172.30.48.0/23 | VLSM plan, see `02-ip-addressing-plan.md` |
| Inter-VLAN connectivity | Router-on-a-Stick — single router, dot1Q sub-interface per VLAN |
| CR4 — restrict server access | Extended ACL on server VLAN sub-interface |
| Design Constraint — future department | Reserved VLAN 60 + subnet, pre-configured |
| Working, testable Packet Tracer file | Built and verified per `04-physical-topology.md` |

## 6. Out of Scope (documented assumption)

No external WAN/Internet connectivity is assumed or required — the brief
specifies internal client requirements, addressing, and the Router-on-a-Stick
challenge only. The design is scoped to the internal LAN to keep the
demonstration focused on the assigned challenge (Router-on-a-Stick) and CR4
(ACL).
