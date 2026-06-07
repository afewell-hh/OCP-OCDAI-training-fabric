# OPG-512 — Rail-Optimized Clos (CX7 1x400G, BF3 2x200G, Converged Storage)

This building block provides a 512-xPU rail-optimized Clos network for 64 eight-GPU servers. Each GPU connects to the backend via a single ConnectX-7 400G NIC (8 NICs per server, one per rail); each server attaches to the frontend via a BlueField-3 DPU with two 200G ports (L3MH across two leaves). With tenancy aligned to first-hop rail domains, most collectives remain leaf-local, dramatically reducing spine traffic and uplink congestion. Consistent DS5000 zoning supports clean scale from smaller OPG tiers.

## Intended use

- Choose this OPG when rail-aware placement is a scheduling priority at the 512-xPU tier.
- Use when keeping collective traffic leaf-local (via rail domain alignment) is critical to meeting bandwidth targets.
- Use as a building block in XOC-512 (1x) or XOC-1024 (2x) compositions.
- Separate backend and frontend fabrics provide traffic isolation not available in collapsed-conv variants.

## Fabrics

- **backend** (Hedgehog-managed): DS5000 backend rail leaves and spines; rail-optimized distribution
- **frontend** (Hedgehog-managed): DS5000 frontend leaves and spines; L3MH with MCLAG

## Endpoint summary

| Role | Count | Notes |
|------|-------|-------|
| Compute servers (8 xPU each) | 64 | 512 xPUs total |

Note: Non-compute endpoint counts (storage, metadata, gateways, controllers) are intentionally excluded from the DIET `topology-map.yaml` translation pending source confirmation. See the XOC compositions for full cluster endpoint counts.

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
- Scale-out NICs: ConnectX-7 single-port 400G per GPU (8 NICs per server, one per rail)
- Frontend/storage NICs: BlueField-3 2x200G per server (L3MH, MCLAG)
- Storage model: converged, 2x200G per server
- Optic class: OS2 SMF DR-class (OSFP-800G-DR8 for uplinks; OSFP-800G-2x400G-DR4 backend; OSFP-800G-4x200G-DR4 frontend)
- Multihoming: rail-optimized on backend; L3MH/ECMP (MCLAG) on frontend

## Assets

- `connectivity-map.csv` — end-to-end cabling and port mapping
- `topology-map.yaml` — topology authoring plan
- `wiring/wiring-backend.yaml` — backend fabric wiring (Hedgehog CRDs)
- `diagrams/hhfab/backend.drawio` — backend topology diagram
- `netbox_inventory.json` — NetBox inventory export

For validated end-to-end wiring in a complete cluster, see the **Implemented by** XOC compositions below. The files in this directory (`topology-map.yaml`, `wiring/`, `diagrams/`) are transitional scaffolding retained for reference and spot-checking.

## Implemented by

- [`xoc-512/1x-OPG-512/clos-ro--cx7-1x400g--bf3-2x200g--storage-conv-2x200g`](../../../xoc/xoc-512/1x-OPG-512/clos-ro--cx7-1x400g--bf3-2x200g--storage-conv-2x200g/)
- [`xoc-1024/2x-OPG-512/clos-ro--cx7-1x400g--bf3-2x200g--storage-conv-2x200g`](../../../xoc/xoc-1024/2x-OPG-512/clos-ro--cx7-1x400g--bf3-2x200g--storage-conv-2x200g/)

## See also

- OPG family overview: `../README.md`
- Compositions root: `../../README.md`
