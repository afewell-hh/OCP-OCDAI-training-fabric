# OPG-256 — Dual-Plane Backend (CX8 2x400G, BF3 2x200G, Converged Storage)

This building block provides a 256-xPU training cluster with two independent backend fabrics (Plane A and Plane B). Each GPU connects via a ConnectX-8 dual-port 400G NIC: port 0 to Plane A and port 1 to Plane B. Frontend uses BF3 2x200G with L3 multihoming. The dual-plane design improves bandwidth headroom for large collectives and allows independent plane maintenance without taking down the entire backend fabric. This OPG is suitable when steady-state utilization is high or when plane-level isolation is operationally required.

## Intended use

- Choose this OPG when higher failure isolation is required: each backend plane can be maintained independently without disrupting the other.
- Use when bandwidth headroom for large, bandwidth-intensive collectives exceeds what a single-plane Clos can provide.
- Use when the CX8 dual-port NIC is the target hardware (one port per plane per GPU).
- Use as a building block in XOC-512 (1x) or XOC-1024 (2x) compositions.
- Dual-plane adds parts and operational complexity; use the single-plane clos-ro variant when isolation is not a priority.

## Fabrics

- **backend-plane-a** (Hedgehog-managed): DS5000 backend Plane A leaves and spines; rail-optimized distribution
- **backend-plane-b** (Hedgehog-managed): DS5000 backend Plane B leaves and spines; rail-optimized distribution
- **frontend** (Hedgehog-managed): DS5000 frontend leaves and spines; L3MH with MCLAG

## Endpoint summary

| Role | Count | Notes |
|------|-------|-------|
| Compute servers (8 xPU each) | 32 | 256 xPUs total; each server has 8x CX8 dual-port NICs |

Note: Non-compute endpoint counts (storage, metadata, gateways, controllers) are intentionally excluded from the DIET `topology-map.yaml` translation pending source confirmation.

## Port budget / uplink reservation

**Backend Plane A/B leaf (DS5000, per plane):**

| Zone | Ports (per leaf) | Breakout | Speed | Purpose |
|------|-----------------|---------|-------|---------|
| Server downlinks | 1–32 | 2x400G | 400G | xPU compute backend connections (one port per plane per GPU NIC) |
| **XOC uplinks** | **33–64 (32 ports)** | **1x800G** | **800G** | **Reserved for XOC spine — must NOT be consumed by server connections** |

**Frontend leaf (DS5000):**

| Zone | Ports (per leaf) | Breakout | Speed | Purpose |
|------|-----------------|---------|-------|---------|
| Server downlinks (odd) | 1, 3, 5, … 63 (32 odd ports) | 4x200G | 200G | Frontend/storage endpoint connections |
| **XOC uplinks** | **2, 4, 6, … 64 (32 even ports)** | **1x800G** | **800G** | **Reserved for XOC spine — must NOT be consumed by server connections** |

## What a composer must supply

1. **XOC backend Plane A spine switches** (DS5000): cable to the 32x800G reserved uplinks on each Plane A leaf (ports 33–64)
2. **XOC backend Plane B spine switches** (DS5000): cable to the 32x800G reserved uplinks on each Plane B leaf (ports 33–64)
3. **XOC frontend spine switches** (DS5000): cable to the 32x800G reserved uplinks on each frontend leaf (even ports 2–64)
4. **External/border connectivity** (DS3000 or equivalent) for WAN/ISP uplinks if required

## Attributes

- Topology type: dual-plane backend Clos (2p); rail-optimized distribution per plane
- Scale-out NICs: ConnectX-8 dual-port 400G per GPU (8 NICs per server; port 0 to Plane A, port 1 to Plane B)
- Frontend/storage NICs: BlueField-3 2x200G per server (L3MH, MCLAG)
- Storage model: converged, 2x200G per server
- Optic class: OS2 SMF DR-class (OSFP-800G-DR8 for uplinks; OSFP-800G-2x400G-DR4 for backend)
- Multihoming: rail-optimized per plane on backend; L3MH/ECMP (MCLAG) on frontend

## Assets

- `connectivity-map.csv` — end-to-end cabling and port mapping
- `topology-map.yaml` — topology authoring plan
- `wiring/wiring-backend-plane-a.yaml` — backend Plane A fabric wiring (Hedgehog CRDs)
- `wiring/wiring-backend-plane-b.yaml` — backend Plane B fabric wiring (Hedgehog CRDs)
- `diagrams/hhfab/backend-plane-a.drawio` — backend Plane A topology diagram
- `diagrams/hhfab/backend-plane-b.drawio` — backend Plane B topology diagram
- `netbox_inventory.json` — NetBox inventory export

For validated end-to-end wiring in a complete cluster, see the **Implemented by** XOC compositions below. The files in this directory (`topology-map.yaml`, `wiring/`, `diagrams/`) are transitional scaffolding retained for reference and spot-checking.

## Implemented by

No XOC composition in this catalog currently implements this OPG.

## See also

- OPG family overview: `../README.md`
- Compositions root: `../../README.md`
