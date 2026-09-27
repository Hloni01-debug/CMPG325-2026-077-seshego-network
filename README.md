# CMPG 325 — Enterprise Network Infrastructure Project

**Student:** Mogola, Hloni
**Project ID:** CMPG325-2026-077
**Client:** Seshego Roadworks & Paving (Kimberley)
**Industry:** Construction

---

## Executive Summary

This repository contains the complete network design, VLSM addressing plan,
Cisco IOS configuration files, and simulation verification evidence for
**Seshego Roadworks & Paving**.

The network architecture is built on **Router-on-a-Stick (802.1Q
sub-interface inter-VLAN routing)** utilizing a Cisco ISR 4321 router and
Cisco Catalyst 2960 switches. The implementation enforces departmental VLAN
segmentation, an Extended ACL controlling access to the application/file
server (CR4), native VLAN trunk hardening, and pre-provisioned capacity
reserved for a new department next year (VLAN 60 Design Constraint)
requiring zero re-addressing when activated.

---

## Network Architecture & IP Addressing Plan

The allocation uses a base block of `172.30.48.0/23`, partitioned using
Variable Length Subnet Masking (VLSM):

| VLAN | Name | Subnet | Gateway (R1 Sub-interface) | Usable Range |
| :---: | :--- | :--- | :--- | :--- |
| **10** | Admin | `172.30.48.64/26` | `172.30.48.65` | `172.30.48.66 – 172.30.48.126` |
| **20** | Engineering | `172.30.48.0/26` | `172.30.48.1` | `172.30.48.2 – 172.30.48.62` |
| **30** | SiteOps | `172.30.48.192/27` | `172.30.48.193` | `172.30.48.194 – 172.30.48.222` |
| **40** | Finance | `172.30.48.224/27` | `172.30.48.225` | `172.30.48.226 – 172.30.48.254` |
| **50** | Servers (CR4) | `172.30.49.0/28` | `172.30.49.1` | `172.30.49.2 – 172.30.49.14` |
| **60** | Future Reserve | `172.30.48.128/26` | `172.30.48.129` | `172.30.48.130 – 172.30.48.190` |
| **99** | Management | `172.30.49.16/28` | `172.30.49.17` | `172.30.49.18 – 172.30.49.30` |
| **999** | Native / ParkingLot | Unrouted | N/A | Anti-VLAN-hopping (carries no data/hosts) |

---

## Key Technical Features

- **Router-on-a-Stick Inter-VLAN Routing:** Physical link `Gi0/0/0` on `R1`
  configured with 802.1Q encapsulation sub-interfaces `.10` through `.99`,
  each acting as the default gateway for its respective subnet.
- **Native VLAN Hardening:** All 802.1Q trunk links across `R1`, `SW-CORE`,
  and access switches (`SW-ADMIN`, `SW-ENG`, `SW-SITE`, `SW-FIN`) set their
  native VLAN to the unused, unrouted **VLAN 999** — kept separate from the
  live **VLAN 99** management subnet — to mitigate VLAN hopping and
  double-tagging attacks.
- **CR4 — Restricted Server Access:** Extended ACL applied outbound on the
  server sub-interface (`Gi0/0/0.50`) permitting Admin, Engineering, and
  Finance subnets while denying Site Operations.
- **Design Constraint (Future Expansion):** VLAN 60 (`172.30.48.128/26`) and
  sub-interface `Gi0/0/0.60` are fully provisioned on `R1` and `SW-CORE`;
  access-switch ports for the upcoming department remain unconfigured/disabled
  until deployment.
- **Switch Security Hardening:** Unused access ports across all switches are
  assigned to blackhole VLAN 999 and placed in an `administratively down`
  state.

---

## Repository Structure

```text
.
├── README.md
├── docs/
│   ├── 01-client-requirements.md   # Assumed departments, headcounts, CR4 interpretation
│   ├── 02-ip-addressing-plan.md    # VLSM breakdown of 172.30.48.0/23
│   ├── 03-logical-topology.md      # VLANs, sub-interfaces, ACL placement
│   └── 04-physical-topology.md     # Devices, cabling, ports
├── diagrams/
│   ├── logical-topology.png
│   ├── physical-topology.png
│   └── screenshots/                # Captured verification evidence
│       ├── 01-pc-admin-pings-part1.png
│       ├── 02-pc-admin-pings-complete.png
│       ├── 03-r1-subinterfaces-routing.png
│       ├── 04-sw-core-trunking.png
│       ├── 05-sw-core-vlan-database.png
│       └── 06-sw-admin-svi-port-security.png
└── packet-tracer/
    └── Seshego_Roadworks_Enterprise_Network.pkt
```

---

## Milestone Progress

| Milestone | Due Date | Status | Details |
| --- | --- | --- | --- |
| **Milestone 1** — Client Design Review | 28 Aug 2026 | **Completed** | Requirements, physical & logical topology, IP addressing plan, initial repo. |
| **Milestone 2** — Implementation & Routing Verification | 2 Oct 2026 | **Completed** | `.pkt` build, sub-interfaces, trunking, management SVI, and ping tests verified. |
| **Final Submission** | 16 Oct 2026 | **In Progress** | Final `.pkt` delivery, repository documentation audit, and demonstration video. |

---

## Verification & Testing Evidence

All tests were executed and validated inside Cisco Packet Tracer:

1. **Default Gateway Reachability**
   Ping from `PC-Admin2` (`172.30.48.66`) → Gateway (`172.30.48.65`):
   **4/4 Received (0% Loss)**.
   *Reference:* `diagrams/screenshots/01-pc-admin-pings-part1.png`

2. **Inter-VLAN Departmental Routing**
   Ping from `PC-Admin2` (VLAN 10) → `PC-Eng1` (`172.30.48.2`, VLAN 20):
   **4/4 Received (0% Loss, TTL=127)**.
   *Reference:* `diagrams/screenshots/01-pc-admin-pings-part1.png`

3. **Server Subnet Access (Authorized VLAN)**
   Ping from `PC-Admin2` (VLAN 10) → `SRV-FILE` (`172.30.49.2`, VLAN 50):
   **4/4 Received (0% Loss, TTL=127)**.
   *Reference:* `diagrams/screenshots/01-pc-admin-pings-part1.png`

4. **Cross-Subnet Management Reachability**
   Ping from `PC-Admin2` (VLAN 10) → `SW-ADMIN` SVI (`172.30.49.19`, VLAN 99):
   **4/4 Received (0% Loss, TTL=254)**.
   *Reference:* `diagrams/screenshots/02-pc-admin-pings-complete.png`

5. **Router Sub-interface Status & Routing Table**
   Executed `show ip interface brief` and `show ip route` on **R1**.
   Confirmed sub-interfaces `Gi0/0/0.10` through `Gi0/0/0.99` in `up/up`
   operational status and directly connected in the routing table.
   *Reference:* `diagrams/screenshots/03-r1-subinterfaces-routing.png`

6. **Core Switch Trunking & VLAN Database**
   Executed `show interfaces trunk` on **SW-CORE**: confirmed 802.1Q
   trunking active across `Fa0/2–5` and `Gi0/1`, with VLANs
   `10, 20, 30, 40, 50, 60, 99` in Spanning Tree forwarding state.
   Executed `show vlan brief` on **SW-CORE**: verified active VLANs and
   assignment of unused ports to VLAN 999.
   *Reference:* `diagrams/screenshots/04-sw-core-trunking.png` &
   `diagrams/screenshots/05-sw-core-vlan-database.png`

7. **Access Switch Port Security & SVI Verification**
   Executed `show ip interface brief` on **SW-ADMIN**: confirmed `Vlan99`
   (`172.30.49.19`) is `up/up` and all unused switchports are
   `administratively down`.
   *Reference:* `diagrams/screenshots/06-sw-admin-svi-port-security.png`

---

## Assigned Networking Challenge

**Challenge:** Router-on-a-Stick (sub-interface inter-VLAN routing)
**Difficulty:** Intermediate

---

## Academic Integrity Statement

AI assistance was used for planning, drafting documentation structures, and
troubleshooting Cisco IOS syntax in alignment with the **North-West
University (NWU) AI Policy**. All network design decisions, topology
construction, command execution, and verification tests remain the
student's own work.
