# Experimental XOC-3712 — 6× OPG-512 + 1× OPG-640

Fork-only preliminary composition for issue #606. It combines six 64-server
B300 OPG-512 domains (3,072 GPUs) with one 80-server SN40/SN50 OPG-640 domain
(640 XPUs). Storage and converged-management remain OPG-local; SN40/SN50
management is intentionally omitted.

Each of the three shared DS5000 fabrics has 32 spines. Frontend has 14 leaves;
backend plane A and B each have 51. Every leaf uses reserved E1/33–64 once,
yielding 448 + 1,632 + 1,632 = 3,712 leaf-spine links.

`wiring/` contains the composed HNP artifacts. The frontend artifact passed
`hhfab validate`; backend validation and derived inventory/BOM/map exports are
the remaining follow-up before promotion.
