# OPG-64 — Rail-Optimized Clos (CX7 1x400G, BF3 2x200G, Converged Storage)

This building block applies a rail-optimized Clos to a 64-xPU training cluster. Scale-out traffic uses CX7 1x400G per GPU (RoCE); frontend and storage use BF3 2x200G per server with L3 multihoming across two leaves. Storage is converged into the frontend network for simplicity at this tier. Scheduling tenants within a common first-hop rail domain keeps most collectives leaf-local, dramatically reducing spine traffic.

## Intended use

- Choose this OPG when rail-aware placement is expected to dominate scheduling decisions at the 64-xPU scale.
- Use when a compact two-leaf backend is preferred over a full Clos with a dedicated spine tier within the OPG itself.
- Use when converged frontend (storage + in-band on the same leaf pair) is acceptable, without a separate in-band management fabric.
- Scales cleanly to larger OPG tiers without changing DS5000 zoning rules.

## Fabrics

- **backend** (Hedgehog-managed): DS5000 leaf pair carrying rail-optimized scale-out (RDMA/RoCEv2)
- **frontend** (Hedgehog-managed): DS5000 leaf pair carrying converged frontend, storage, and in-band traffic
- **oob-mgmt** (unmanaged): DS1000 pair for BMC and PDU management

## Endpoint summary

| Role | Count | Notes |
|------|-------|-------|
| Compute servers (8 xPU each) | 8 | 64 xPUs total |
| Storage servers | 3 | Converged storage |
| Metadata servers | 3 | |
| HH gateways | 2 | |
| HH controller | 1 | |

## Port budget / uplink reservation

**Backend rail leaf (DS5000):**

| Zone | Ports (per leaf) | Breakout | Speed | Purpose |
|------|-----------------|---------|-------|---------|
| Server downlinks | 1–32 | 2x400G | 400G | xPU compute backend connections |
| **XOC uplinks** | **33–64 (32 ports)** | **1x800G** | **800G** | **Reserved for XOC spine — must NOT be consumed by server connections** |

**Frontend leaf (DS5000):**

| Zone | Ports (per leaf) | Breakout | Speed | Purpose |
|------|-----------------|---------|-------|---------|
| Server downlinks (odd) | 1, 3, 5, … 63 | 4x200G | 200G | Frontend/storage endpoint connections |
| **XOC uplinks** | **2, 4, 6, … 64 (32 even ports)** | **1x800G** | **800G** | **Reserved for XOC spine — must NOT be consumed by server connections** |

## What a composer must supply

1. **XOC backend spine switches** (DS5000) connected to the 32x800G reserved uplink ports on each backend rail leaf (ports 33–64)
2. **XOC frontend spine switches** (DS5000) connected to the 32x800G reserved uplink ports on each frontend leaf (even ports 2–64)
3. **External/border connectivity** (DS3000 or equivalent) for WAN/ISP uplinks if required

## Attributes

- Topology type: rail-optimized Clos, single backend plane
- Scale-out NICs: CX7 1x400G per GPU
- Frontend/storage NICs: BF3 2x200G per server (L3MH)
- Storage model: converged, 2x200G per server
- Optic class: OS2 SMF DR-class (OSFP-800G-DR8 for uplinks; OSFP-800G-2x400G-DR4 backend; OSFP-800G-4x200G-DR4 frontend)
- Multihoming: L3MH/ECMP on frontend; rail-optimized on backend

## Assets

- `connectivity-map.csv` — end-to-end cabling and port mapping
- `wiring/wiring-backend.yaml` — backend fabric wiring (Hedgehog CRDs)
- `diagrams/hhfab/backend.drawio` — backend topology diagram
- `netbox_inventory.json` — NetBox inventory export

For validated end-to-end wiring in a complete cluster, see the **Implemented by** XOC compositions below. The files in this directory (`topology-map.yaml`, `wiring/`, `diagrams/`) are transitional scaffolding retained for reference and spot-checking.

## Implemented by

No XOC composition in this catalog currently implements this OPG.

## See also

- OPG family overview: `../README.md`
- Compositions root: `../../README.md`
