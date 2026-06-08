# XOC‑1024 / 2× OPG‑512 — Rail‑Optimized Clos

At this scale, domain‑aligned placement is pivotal: jobs kept within first‑hop rail domains keep most collectives leaf‑local and reduce spine traffic across the composition.

Attributes
- Backend: rail‑optimized; CX7 1×400G per GPU (per OPG)
- Frontend: BF3 2×200G (L3MH); storage converged per OPG

Assets
- `connectivity-map.csv` — end‑to‑end cabling and port mapping (1856 cables)
- `netbox_inventory.json` — NetBox inventory export
- `bom.csv` — bill of materials (servers, switches, NICs, transceivers)
- `wiring/` — Hedgehog Wiring CRDs per fabric
  - `wiring-backend.yaml` — hhfab validate OK
  - `wiring-frontend.yaml` — hhfab validate OK
- `diagrams/hhfab/` — hhfab diagrams and validate logs per fabric
- `generated/` — pipeline provenance (inputs, run logs)

See also
- Tier overview: ../../README.md
- Compositions overview: ../../../README.md
