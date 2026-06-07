# OPG-64 — Collapsed Converged (CX7 1x400G, BF3 2x200G, Converged Storage)

A single converged leaf pair (2x DS5000) carries both the scale-out (RDMA) backend and the converged frontend (storage, in-band, management) for up to 8 compute servers (64 xPUs). Out-of-band management uses a DS1000 pair. No spine switch is active in this building block; the topology includes a placeholder spine pair whose uplink ports are reserved for future XOC connectivity or standalone growth. This is the simplest OPG topology: one managed fabric instead of two, no dedicated spine, and all endpoint types sharing the same leaf pair.

## Intended use

- Choose this OPG for the smallest practical cluster footprint at 64-xPU scale — one managed switch pair instead of separate backend and frontend fabrics.
- Use when a converged backend and frontend on the same leaf pair is acceptable and switch count minimization is the priority.
- Use as a standalone pod without spine connectivity, or as a leaf-only building block that a composer connects to XOC spine switches when scaling out.
- Not suitable when backend and frontend traffic isolation is required, or when rail-optimized placement is a scheduling constraint.

## Fabrics

- **converged** (Hedgehog-managed): single DS5000 leaf pair carrying backend (RDMA/RoCEv2), frontend (storage NVMe-oF, in-band, control-plane), and storage traffic
- **oob-mgmt** (unmanaged): DS1000 pair for BMC and PDU management

## Endpoint summary

| Role | Count | Notes |
|------|-------|-------|
| Compute servers (8 xPU each) | 8 | 64 xPUs total |
| Storage servers | 3 | Hyperconverged |
| Metadata servers | 3 | |
| HH gateways | 2 | |
| HH controller | 1 | |

## Port budget / uplink reservation

**Converged leaf (DS5000):**

| Zone | Ports (per leaf) | Breakout | Speed | Purpose |
|------|-----------------|---------|-------|---------|
| Backend server | 1–16 | 2x400G | 400G | xPU compute backend connections |
| Converged FE/storage | 27, 29, 31, 33, 35, 37 (6 odd ports) | 4x200G | 200G | Frontend, storage, and in-band connections |
| **XOC uplinks** | **26, 28, 30, 32, 34, 36, 38–63 (32 ports)** | **1x800G** | **800G** | **Reserved for XOC spine — must NOT be consumed by server connections** |

Note: 4x200G breakouts are only valid on odd ports (27, 29, 31, …). Violating this rule disrupts the uplink budget.

## What a composer must supply

1. **XOC spine switches** (DS5000 or equivalent) connected to the 32x800G reserved uplink ports per leaf (ports 26, 28, 30, 32, 34, 36, 38–63)
2. **Uplink cabling** — 32x OSFP-800G-DR8 cables per leaf from the reserved uplink zone to the XOC spine
3. **External/border connectivity** (DS3000 or equivalent) for WAN/ISP uplinks if required

The collapsed leaf pair can operate stand-alone without a spine when no external connectivity or cross-OPG traffic is required. The placeholder spine pair included in the topology plan is not wired; post-generation manual adjustment is required to activate it.

## Attributes

- Topology type: collapsed converged (single managed fabric, single plane)
- Scale-out NICs: CX7 1x400G per GPU
- Frontend/storage NICs: BF3 2x200G per server
- Storage model: converged, 2x200G per server
- Optic class: OS2 SMF DR-class (OSFP-800G-2x400G-DR4 for backend; OSFP-800G-4x200G-DR4 for FE/storage)
- Multihoming: L3MH/ECMP (HNP validation shim is eslag; RA intent is L3MH)

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
