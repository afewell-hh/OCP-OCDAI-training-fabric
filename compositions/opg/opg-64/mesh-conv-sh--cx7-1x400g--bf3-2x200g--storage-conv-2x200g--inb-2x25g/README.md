# OPG-64 — Mesh-Converged Single-Homed (CX7 1x400G, BF3 2x200G, Converged Storage, In-Band 2x25G)

This building block provides a 64-xPU training cluster using a shared DS5000 mesh pair that carries scale-out, SoC/storage, and in-band management on a single converged fabric. Scale-out traffic uses CX7 1x400G per GPU with same-switch distribution; storage and SoC traffic use BF3 2x200G per server; in-band management uses a dedicated DS2000 pair with a generic 2x25G NIC on every server class. Out-of-band management uses a DS1000 pair. This variant is structurally identical to the rail-optimized sibling (`mesh-conv-ro`) except that scale-out uses `same-switch` distribution rather than `rail-optimized`.

## Intended use

- Choose this OPG when a simpler scale-out attachment model is preferred over rail-optimized distribution at the 64-xPU tier.
- Use when job scheduling does not exploit first-hop rail locality, making rail-optimized distribution unnecessary.
- Use when a separate in-band management fabric (DS2000) is required alongside the converged scale-out/SoC leaf pair.
- Note: grouped-per-leaf `same-switch` semantics depend on `hh-netbox-plugin` issue `#322`; until resolved, scale-out connections may be distributed alternating rather than grouped by leaf.

## Fabrics

- **soc-storage-scale-out** (Hedgehog-managed): DS5000 mesh leaf pair carrying scale-out (RDMA/RoCEv2) and SoC/storage traffic
- **inb-mgmt** (Hedgehog-managed): DS2000 leaf pair for in-band management (25G)
- **oob-mgmt** (unmanaged): DS1000 pair for BMC and PDU management

## Endpoint summary

| Role | Count | Notes |
|------|-------|-------|
| Compute servers (8 xPU each) | 8 | 64 xPUs total |
| Storage servers | 3 | Hyperconverged |
| Metadata servers | 3 | |
| HH gateways | 2 | No scale-out or SoC NIC |
| HH controller | 1 | No scale-out or SoC NIC |

## Port budget / uplink reservation

**SoC/storage/scale-out leaf (DS5000):**

| Zone | Ports (per leaf) | Breakout | Speed | Purpose |
|------|-----------------|---------|-------|---------|
| Scale-out server | 1–16 | 2x400G | 400G | xPU compute scale-out connections |
| SoC/storage server | 27, 29, 31, 33, 35, 37 (6 odd ports) | 4x200G | 200G | SoC and storage connections |
| Mesh inter-switch | 26, 28 | 1x800G | 800G | DS5000 mesh cross-connect (consumed within OPG) |
| **XOC uplinks** | **30, 32, 34, 36, 38–63 (30 ports)** | **1x800G** | **800G** | **Reserved for XOC spine — must NOT be consumed by server connections** |

Note: Ports 26 and 28 are consumed by the mesh cross-connect within this OPG. The uplink zone (`port_spec: 26, 28, 30, 32, 34, 36, 38–63`) defines 32 ports total; 2 are used for mesh, leaving 30 available for XOC spine uplinks.

## What a composer must supply

1. **XOC spine switches** (DS5000 or equivalent) connected to the available uplink ports on each DS5000 mesh leaf (30x800G per leaf after mesh ports are consumed)
2. **Uplink cabling** — OSFP-800G-DR8 cables from available uplink ports to XOC spine
3. **External/border connectivity** (DS3000 or equivalent) for WAN/ISP uplinks if required

## Attributes

- Topology type: mesh-converged, single-homed scale-out
- Scale-out NICs: CX7 1x400G per GPU (8 ports per compute server, all to same switch; see `#322` note above)
- SoC/storage NICs: BF3 2x200G per compute server; generic dual-200G for storage/metadata
- In-band management NIC: generic 2x25G on every server class (one port connected)
- Storage model: converged on the SoC/storage mesh fabric
- Optic class: OS2 SMF DR-class
- Multihoming: same-switch on scale-out; same-switch on SoC/storage

## Assets

- `connectivity-map.csv` — end-to-end cabling and port mapping
- `topology-map.yaml` — topology authoring plan
- `wiring/` — Hedgehog Wiring CRDs
- `diagrams/` — visual diagrams
- `netbox_inventory.json` — NetBox inventory export

For validated end-to-end wiring in a complete cluster, see the **Implemented by** XOC compositions below. The files in this directory (`topology-map.yaml`, `wiring/`, `diagrams/`) are transitional scaffolding retained for reference and spot-checking.

## Implemented by

- [`xoc-64/1x-OPG-64/mesh-conv-sh--cx7-1x400g--bf3-2x200g--storage-conv-2x200g--inb-2x25g`](../../../xoc/xoc-64/1x-OPG-64/mesh-conv-sh--cx7-1x400g--bf3-2x200g--storage-conv-2x200g--inb-2x25g/)
- [`xoc-128/2x-opg-64/mesh-conv-sh--cx7-1x400g--bf3-2x200g--storage-conv-2x200g--inb-2x25g`](../../../xoc/xoc-128/2x-opg-64/mesh-conv-sh--cx7-1x400g--bf3-2x200g--storage-conv-2x200g--inb-2x25g/)

## See also

- OPG family overview: `../README.md`
- Compositions root: `../../README.md`
