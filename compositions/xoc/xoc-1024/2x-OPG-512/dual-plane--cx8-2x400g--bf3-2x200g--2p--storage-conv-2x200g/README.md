# XOC‑1024 / 2× OPG‑512 — Dual‑Plane

Two backend planes; CX8 2×400G per GPU.

Attributes
- Backend: dual plane (2p); CX8 2×400G per GPU
- Frontend: BF3 2×200G (L3MH); storage converged

Assets
- `connectivity-map.csv` — end‑to‑end cabling and port mapping (3392 cables)
- `netbox_inventory.json` — NetBox inventory export
- `bom.csv` — bill of materials (servers, switches, NICs, transceivers)
- `wiring/` — Hedgehog Wiring CRDs per fabric
  - `wiring-backend-plane-a.yaml` — hhfab validate OK
  - `wiring-backend-plane-b.yaml` — hhfab validate OK
  - `wiring-frontend.yaml` — hhfab validate OK
- `diagrams/hhfab/` — hhfab diagrams and validate logs per fabric
- `generated/` — pipeline provenance (inputs, run logs)

See also
- Tier overview: ../../README.md
- Compositions overview: ../../../README.md
