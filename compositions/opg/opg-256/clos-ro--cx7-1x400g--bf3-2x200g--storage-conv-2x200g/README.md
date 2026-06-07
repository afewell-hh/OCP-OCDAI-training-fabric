# OPG-256 — Rail-Optimized Clos (CX7 1x400G, BF3 2x200G, Converged Storage)

This building block provides a 256-xPU rail-optimized Clos network for 32 eight-GPU servers. Each GPU connects to the backend via a single ConnectX-7 400G NIC (8 NICs per server, one per rail), while each server attaches to the frontend via a BlueField-3 DPU with two 200G ports (L3MH across two leaves). The rail-optimized wiring maps each pair of rails to a dedicated backend leaf switch, so every backend leaf serves all 32 servers but only two of their eight GPUs. This produces the same switch count and bandwidth as a standard Clos but constrains the cabling pattern to simplify deployment and troubleshooting. This OPG does not include an XOC spine; uplink ports are reserved on each leaf for XOC connectivity.

## Intended use

- Choose this OPG for the 256-xPU tier when rail-aware placement is a scheduling priority.
- Use when backend and frontend fabric isolation is required (separate fabrics, separate spine tiers within this OPG).
- Use as a building block in XOC-256 (1x) or XOC-512 (2x) compositions.
- The Hedgehog backend controller (hh-ctrl-be) uses port E1/33 on be-leaf-01 and be-leaf-02, reducing those leaves' effective XOC uplink capacity from 32 to 31.

## Fabrics

- **backend** (Hedgehog-managed): 4x DS5000 backend rail leaves + 4x DS5000 backend spines (within-OPG); rail-optimized distribution
- **frontend** (Hedgehog-managed): 2x DS5000 frontend leaves + 2x DS5000 frontend spines (within-OPG); L3MH with MCLAG
- **oob** (unmanaged): DS1000-48 switches for BMC and PDU management

## Endpoint summary

| Role | Count | Notes |
|------|-------|-------|
| Compute servers (8 xPU each) | 32 | 256 xPUs total |
| Storage servers | 10 | See OPG-256 RA source |
| Metadata servers | 10 | See OPG-256 RA source |
| HH gateways | 2 | |
| HH controller | 1 | |

Note: Non-compute endpoint counts are based on the OPG-256 RA reference; they are excluded from the DIET `topology-map.yaml` translation pending final source confirmation.

## Port budget / uplink reservation

**Backend rail leaf (DS5000):**

| Zone | Ports (per leaf) | Breakout | Speed | Purpose |
|------|-----------------|---------|-------|---------|
| Server downlinks | 1–32 | 2x400G | 400G | xPU compute backend connections |
| Within-OPG spine uplinks | 33–64 (32 ports) | 1x800G | 800G | Connected to within-OPG backend spine |

Note: The within-OPG backend spine is part of this building block. There are no free uplink ports reserved for an XOC spine above this OPG at the backend rail leaf level — the XOC backend spine connectivity is provided by the within-OPG backend spine's own uplink capacity. See Composer requirements below.

**Frontend leaf (DS5000):**

| Zone | Ports (per leaf) | Breakout | Speed | Purpose |
|------|-----------------|---------|-------|---------|
| Server downlinks (odd) | 1, 3, 5, … 63 (32 odd ports) | 4x200G | 200G | Frontend/storage endpoint connections (56 logical ports per leaf) |
| **XOC uplinks** | **2, 4, 6, … 64 (32 even ports)** | **1x800G** | **800G** | **Reserved for XOC spine — must NOT be consumed by server connections** |

## What a composer must supply

1. **XOC backend spine switches** (DS5000): cable to the 32x800G reserved uplinks on each within-OPG backend spine switch (ports available at the backend spine tier)
2. **XOC frontend spine switches** (DS5000): cable to the 32x800G reserved uplinks on each frontend leaf (even ports 2–64 per switch)
3. **External/border connectivity** (DS3000 or equivalent) for WAN/ISP uplinks if required

Rail mapping for composer reference: be-leaf-01 = rails 0,1; be-leaf-02 = rails 2,3; be-leaf-03 = rails 4,5; be-leaf-04 = rails 6,7.

## Attributes

- Topology type: rail-optimized Clos, single backend plane, within-OPG spine tier
- Scale-out NICs: ConnectX-7 single-port 400G per GPU (8 NICs per server, one per rail)
- Frontend/storage NICs: BlueField-3 2x200G per server (L3MH, MCLAG)
- Storage model: converged, 2x200G per server
- Optic class: OS2 SMF DR-class (OSFP-800G-DR8 uplinks; OSFP-800G-2x400G-DR4 backend; OSFP-800G-4x200G-DR4 frontend; SFP+-10G-LR OOB)
- Multihoming: rail-optimized on backend; L3MH/ECMP (MCLAG) on frontend
- Backend controller note: hh-ctrl-be uses port E1/33 on be-leaf-01 and be-leaf-02, reducing those leaves' spine uplinks from 32 to 31 (negligible bandwidth impact, less than 4%)

## Assets

- `connectivity-map.csv` — end-to-end cabling and port mapping
- `topology-map.yaml` — topology authoring plan
- `wiring/wiring-backend.yaml` — backend fabric wiring (Hedgehog CRDs)
- `diagrams/hhfab/backend.drawio` — backend topology diagram
- `netbox_inventory.json` — NetBox inventory export

For validated end-to-end wiring in a complete cluster, see the **Implemented by** XOC compositions below. The files in this directory (`topology-map.yaml`, `wiring/`, `diagrams/`) are transitional scaffolding retained for reference and spot-checking.

## Implemented by

No XOC composition in this catalog currently implements this OPG.

## See also

- OPG family overview: `../README.md`
- Compositions root: `../../README.md`
- OPG-M System Architecture (2026-01-14): https://www.opencompute.org/documents/opg-m-system-architecture-final-14-january-2026-pdf
- XOC-N System Architecture (2026-01-14): https://www.opencompute.org/documents/xoc-n-system-architecture-final-14-january-2026-pdf
