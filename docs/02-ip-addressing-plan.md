# IP Addressing Plan (VLSM) — 172.30.48.0/23

Assigned block: **172.30.48.0/23** = 172.30.48.0 – 172.30.49.255 (512 addresses)

Allocated largest-to-smallest (standard VLSM practice) so every block falls
on a clean subnet boundary with no overlap.

| VLAN | Department | Subnet | Mask | Usable range | Gateway (router sub-int) | Broadcast |
|---|---|---|---|---|---|---|
| 20 | Engineering & Design | 172.30.48.0/26 | /26 (255.255.255.192) | .1 – .62 | 172.30.48.1 | 172.30.48.63 |
| 10 | Management & Administration | 172.30.48.64/26 | /26 | .65 – .126 | 172.30.48.65 | 172.30.48.127 |
| 60 | Future Department (reserved) | 172.30.48.128/26 | /26 | .129 – .190 | 172.30.48.129 | 172.30.48.191 |
| 30 | Site Operations & Projects | 172.30.48.192/27 | /27 (255.255.255.224) | .193 – .222 | 172.30.48.193 | 172.30.48.223 |
| 40 | Finance & Procurement | 172.30.48.224/27 | /27 | .225 – .254 | 172.30.48.225 | 172.30.48.255 |
| 50 | Servers (incl. CR4 file/app server) | 172.30.49.0/28 | /28 (255.255.255.240) | 49.1 – 49.14 | 172.30.49.1 | 172.30.49.15 |
| 99 | Network Management | 172.30.49.16/28 | /28 | 49.17 – 49.30 | 172.30.49.17 | 172.30.49.31 |
| — | **Reserved / unallocated** | 172.30.49.32 – 172.30.49.255 | — | — | — | — |

**224 addresses (172.30.49.32/27 upward) are left unallocated** — headroom
beyond VLAN 60 for further department growth, point-to-point links, or
additional server VLANs, without touching the block already in production.

## Allocation logic (for your video demo)

1. Sized each VLAN to current headcount **plus growth**, not the exact
   headcount — /26 and /27 blocks were chosen over tightly-fit blocks like
   /28 or /29 specifically so a department can add staff without a re-IP.
2. Largest subnets (Engineering, Admin, Future) allocated first from the
   start of the range, smallest (Servers, Management) last — this is why the
   plan lines up cleanly on /26 → /27 → /28 boundaries with zero wasted
   "padding" between blocks.
3. VLAN 60 is fully reserved (not merely a placeholder) — network engineers
   provision ahead of known future demand, and the Design Constraint states
   the new department is coming, not merely possible.
4. Router sub-interface address is always the **first usable host** in each
   subnet, by convention, so gateway addresses are predictable during
   configuration and troubleshooting.
