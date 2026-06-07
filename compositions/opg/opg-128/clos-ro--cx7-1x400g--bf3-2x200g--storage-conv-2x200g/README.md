# OPG-128 — Rail-Optimized Clos (CX7 1x400G, BF3 2x200G, Converged Storage)

This building block applies a rail-optimized Clos to a 128-xPU training pod (16 servers x 8 xPUs). Backend traffic uses CX7 1x400G per GPU with rail-optimized distribution; frontend and storage use BF3 2x200G per server with L3 multihoming across two frontend leaves. Rail-optimized wiring constrains each GPU's CX7 NIC to a dedicated backend rail leaf, giving predictable switch locality per rail domain. DS5000 zoning rules are consistent with larger OPG tiers, supporting clean scale.

## Intended use

- Choose this OPG when rail-aware placement is a scheduling priority at the 128-xPU tier.
- Use when per-rail failure isolation and deterministic RDMA locality are required.
- Use as a repeatable building block in XOC compositions of 128, 256, or 512 xPUs (1x, 2x, or 4x this OPG).
- Separate backend and frontend fabrics provide traffic isolation not available in the collapsed-conv variant.

## Fabrics

- **backend** (Hedgehog-managed): DS5000 backend rail leaf pair carrying scale-out (RDMA/RoCEv2); one NIC per rail
- **frontend** (Hedgehog-managed): DS5000 frontend leaf pair carrying converged frontend and storage traffic (L3MH)

## Endpoint summary

| Role | Count | Notes |
|------|-------|-------|
| Compute servers (8 xPU each) | 16 | 128 xPUs total |

Note: Non-compute endpoint counts (storage, metadata, gateways, controllers) are intentionally excluded from this DIET translation pending source confirmation. See the XOC compositions for full cluster endpoint counts.

## Port budget / uplink reservation

**Backend rail leaf (DS5000):**

| Zone | Ports (per leaf) | Breakout | Speed | Purpose |
|------|-----------------|---------|-------|---------|
| Server downlinks | 1–32 | 2x400G | 400G | xPU compute backend connections |
| **XOC uplinks** | **33–64 (32 ports)** | **1x800G** | **800G** | **Reserved for XOC spine — must NOT be consumed by server connections** |

**Frontend leaf (DS5000):**

| Zone | Ports (per leaf) | Breakout | Speed | Purpose |
|------|-----------------|---------|-------|---------|
| Server downlinks (odd) | 1, 3, 5, … 63 (32 odd ports) | 4x200G | 200G | Frontend/storage endpoint connections |
| **XOC uplinks** | **2, 4, 6, … 64 (32 even ports)** | **1x800G** | **800G** | **Reserved for XOC spine — must NOT be consumed by server connections** |

## What a composer must supply

1. **XOC backend spine switches** (DS5000) connected to the 32x800G reserved uplink ports on each backend rail leaf (ports 33–64)
2. **XOC frontend spine switches** (DS5000) connected to the 32x800G reserved uplink ports on each frontend leaf (even ports 2–64)
3. **External/border connectivity** (DS3000 or equivalent) for WAN/ISP uplinks if required

## Attributes

- Topology type: rail-optimized Clos, single backend plane
- Scale-out NICs: CX7 1x400G per GPU (one NIC per rail, one port per NIC)
- Frontend/storage NICs: BF3 2x200G per server (L3MH)
- Storage model: converged, 2x200G per server
- Optic class: OS2 SMF DR-class (OSFP-800G-DR8 for uplinks; OSFP-800G-2x400G-DR4 backend; OSFP-800G-4x200G-DR4 frontend)
- Multihoming: rail-optimized on backend; L3MH/ECMP (alternating) on frontend

## Assets

- `connectivity-map.csv` — end-to-end cabling and port mapping
- `topology-map.yaml` — topology authoring plan
- `wiring/wiring-backend.yaml` — backend fabric wiring (Hedgehog CRDs)
- `diagrams/hhfab/backend.drawio` — backend topology diagram
- `netbox_inventory.json` — NetBox inventory export

For validated end-to-end wiring in a complete cluster, see the **Implemented by** XOC compositions below. The files in this directory (`topology-map.yaml`, `wiring/`, `diagrams/`) are transitional scaffolding retained for reference and spot-checking.

## Implemented by

- [`xoc-128/1x-opg-128/clos-ro--cx7-1x400g--bf3-2x200g--storage-conv-2x200g`](../../../xoc/xoc-128/1x-opg-128/clos-ro--cx7-1x400g--bf3-2x200g--storage-conv-2x200g/)
- [`xoc-256/2x-OPG-128/clos-ro--cx7-1x400g--bf3-2x200g--storage-conv-2x200g`](../../../xoc/xoc-256/2x-OPG-128/clos-ro--cx7-1x400g--bf3-2x200g--storage-conv-2x200g/)
- [`xoc-512/4x-OPG-128/clos-ro--cx7-1x400g--bf3-2x200g--storage-conv-2x200g`](../../../xoc/xoc-512/4x-OPG-128/clos-ro--cx7-1x400g--bf3-2x200g--storage-conv-2x200g/)

## See also

- OPG family overview: `../README.md`
- Compositions root: `../../README.md`
