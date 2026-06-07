# OPG-128 — Single-Homed Clos (CX7 1x400G, BF3 2x200G, Converged Storage)

This building block applies a single-homed Clos backend to a 128-xPU training pod (16 servers x 8 xPUs). Each server's backend NICs terminate on a single leaf (8 servers per leaf), giving clear per-leaf failure domains. Frontend remains L3 multihomed via BF3 2x200G. This matches the OPG-M exemplar topology for the backend, and DS5000 zoning is consistent with rail-optimized variants at this tier.

## Intended use

- Choose this OPG when per-leaf backend failure domains are more important than rail-aware traffic locality.
- Use when the workload does not benefit from rail-optimized placement, or when single-homed backend cabling is simpler to deploy and operate.
- Use as a repeatable building block in XOC compositions of 128, 256, or 512 xPUs (1x, 2x, or 4x this OPG).
- Aligns with OPG-M exemplar specifications for the backend wiring pattern.

## Fabrics

- **backend** (Hedgehog-managed): DS5000 backend leaf pair carrying scale-out (RDMA/RoCEv2); each server homed to one leaf
- **frontend** (Hedgehog-managed): DS5000 frontend leaf pair carrying converged frontend and storage traffic (L3MH)

## Endpoint summary

| Role | Count | Notes |
|------|-------|-------|
| Compute servers (8 xPU each) | 16 | 128 xPUs total |

Note: Non-compute endpoint counts (storage, metadata, gateways, controllers) are intentionally excluded from this DIET translation pending source confirmation. See the XOC compositions for full cluster endpoint counts.

## Port budget / uplink reservation

**Backend leaf (DS5000):**

| Zone | Ports (per leaf) | Breakout | Speed | Purpose |
|------|-----------------|---------|-------|---------|
| Server downlinks | 1–32 | 2x400G | 400G | xPU compute backend connections (8 servers per leaf) |
| **XOC uplinks** | **33–64 (32 ports)** | **1x800G** | **800G** | **Reserved for XOC spine — must NOT be consumed by server connections** |

**Frontend leaf (DS5000):**

| Zone | Ports (per leaf) | Breakout | Speed | Purpose |
|------|-----------------|---------|-------|---------|
| Server downlinks (odd) | 1, 3, 5, … 63 (32 odd ports) | 4x200G | 200G | Frontend/storage endpoint connections |
| **XOC uplinks** | **2, 4, 6, … 64 (32 even ports)** | **1x800G** | **800G** | **Reserved for XOC spine — must NOT be consumed by server connections** |

## What a composer must supply

1. **XOC backend spine switches** (DS5000) connected to the 32x800G reserved uplink ports on each backend leaf (ports 33–64)
2. **XOC frontend spine switches** (DS5000) connected to the 32x800G reserved uplink ports on each frontend leaf (even ports 2–64)
3. **External/border connectivity** (DS3000 or equivalent) for WAN/ISP uplinks if required

## Attributes

- Topology type: single-homed Clos (per-server backend on one leaf), single backend plane
- Scale-out NICs: CX7 1x400G per GPU (8 NICs per server, all homed to same leaf)
- Frontend/storage NICs: BF3 2x200G per server (L3MH)
- Storage model: converged, 2x200G per server
- Optic class: OS2 SMF DR-class (OSFP-800G-DR8 for uplinks; OSFP-800G-2x400G-DR4 backend; OSFP-800G-4x200G-DR4 frontend)
- Multihoming: single-homed on backend; L3MH/ECMP (alternating) on frontend

## Assets

- `connectivity-map.csv` — end-to-end cabling and port mapping
- `topology-map.yaml` — topology authoring plan
- `wiring/wiring-backend.yaml` — backend fabric wiring (Hedgehog CRDs)
- `diagrams/hhfab/backend.drawio` — backend topology diagram
- `netbox_inventory.json` — NetBox inventory export

For validated end-to-end wiring in a complete cluster, see the **Implemented by** XOC compositions below. The files in this directory (`topology-map.yaml`, `wiring/`, `diagrams/`) are transitional scaffolding retained for reference and spot-checking.

## Implemented by

- [`xoc-128/1x-opg-128/clos-sh--cx7-1x400g--bf3-2x200g--storage-conv-2x200g`](../../../xoc/xoc-128/1x-opg-128/clos-sh--cx7-1x400g--bf3-2x200g--storage-conv-2x200g/)
- [`xoc-256/2x-OPG-128/clos-sh--cx7-1x400g--bf3-2x200g--storage-conv-2x200g`](../../../xoc/xoc-256/2x-OPG-128/clos-sh--cx7-1x400g--bf3-2x200g--storage-conv-2x200g/)
- [`xoc-512/4x-OPG-128/clos-sh--cx7-1x400g--bf3-2x200g--storage-conv-2x200g`](../../../xoc/xoc-512/4x-OPG-128/clos-sh--cx7-1x400g--bf3-2x200g--storage-conv-2x200g/)

## See also

- OPG family overview: `../README.md`
- Compositions root: `../../README.md`
