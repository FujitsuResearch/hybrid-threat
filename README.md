# Hybrid Threat Knowledge Graph

Supplementary data for the accompanying paper on a **cyber × cognitive
integrated threat knowledge graph**. This dataset expresses the operational
relationships between cyber attacker behaviours (MITRE ATT&CK) and cognitive /
information-influence behaviours (DISARM) as a single, standards-based
[STIX 2.1](https://docs.oasis-open.org/cti/stix/v2.1/stix-v2.1.html) graph.

The core idea is a **bridging layer**: instead of re-implementing or merging the
two frameworks, we keep the official ATT&CK and DISARM bundles intact and add a
small set of *role-category* objects and *relationship* objects that connect
`attack-pattern`s across the two frameworks by their STIX IDs.

## Contents

```
public/
├── README.md                     ← this file
├── LICENSE                       ← CC BY-SA 4.0 (applies to the original contribution)
├── THIRD_PARTY_LICENSES/
│   ├── NOTICE.md                 ← required attributions (ATT&CK, DISARM, JRC)
│   ├── MITRE-ATTACK-LICENSE.txt  ← MITRE ATT&CK Terms of Use
│   └── DISARM-LICENSE.md         ← CC BY-SA 4.0 legal code
└── data/
    ├── attack.bundle.json                ← MITRE ATT&CK (Enterprise) STIX 2.1 bundle
    ├── disarm.bundle.json                ← DISARM STIX 2.1 bundle
    └── attack_pattern_relations.json     ← original cross-framework bridge bundle
```

### `data/attack.bundle.json`

A STIX 2.1 bundle of MITRE ATT&CK (Enterprise). Redistributed unmodified as the
node source for cyber techniques.

| Object type | Count |
|---|---:|
| `attack-pattern` | 835 |
| `x-mitre-analytic` | 1,739 |
| `malware` | 696 |
| `x-mitre-detection-strategy` | 691 |
| `course-of-action` | 268 |
| `intrusion-set` | 187 |
| `x-mitre-data-component` | 109 |
| `tool` | 91 |
| `campaign` | 52 |
| `x-mitre-data-source` | 38 |
| `x-mitre-tactic` | 14 |
| `relationship` | 20,048 |
| *other* (`identity`, `marking-definition`, `x-mitre-collection`, `x-mitre-matrix`) | 4 |
| **Total** | **24,772** |

### `data/disarm.bundle.json`

A STIX 2.1 bundle of the DISARM Framework. Redistributed as the node source for
cognitive / information-influence techniques.

| Object type | Count |
|---|---:|
| `relationship` | 1,542 |
| `attack-pattern` | 294 |
| `location` | 153 |
| `intrusion-set` | 151 |
| `threat-actor` | 153 |
| `x-mitre-tactic` | 16 |
| *other* (`identity`, `marking-definition`, `x-mitre-matrix`) | 3 |
| **Total** | **2,312** |

### `data/attack_pattern_relations.json`

The **original contribution** of this project: a STIX 2.1 bundle that bridges
the two frameworks above. It does not duplicate any `attack-pattern`; instead it
references the `attack-pattern` objects in the two official bundles by their
STIX IDs (`source_ref` / `target_ref`).

| Object type | Count | Purpose |
|---|---:|---|
| `x-role-category` | 4 | The role taxonomy (see below) |
| `relationship` | 47 | Cross-framework operational links, typed by role |
| `report` | 16 | Source references / evidence for each case |
| `grouping` | 8 | Per-case operational context (8 case studies) |
| `note` | 4 | Analytical disclaimers (e.g. speculative links) |
| **Total** | **79** | |

**Role categories** (`x-role-category`) are derived from a systematic
categorisation of the Hybrid CoE / JRC hybrid-threat conceptual model. Each
category is identified by an `x_role_shortname`:

| `x_role_shortname` | Name | Meaning |
|---|---|---|
| `conditioning` | Conditioning | Activity A fulfils a necessary precondition for activity B. |
| `amplification` | Amplification | Activity A increases the magnitude of effect of activity B. |
| `redirection` | Redirection | The direct target of an activity differs from the target its effect propagates to. |
| `obfuscation` | Obfuscation | Activity A reduces detectability / attributability / intent-traceability of activity B. |

**Relationships** are directed edges between two `attack-pattern`s. The
`relationship_type` carries the role (`x-conditioning`, `x-amplification`,
`x-redirection`, `x-obfuscation`) and matches an `x_role_shortname` via string
equality — the same linking pattern ATT&CK uses between `kill_chain_phases`
and `x-mitre-tactic`. Case-specific nuance is captured in `description`.

Relationship breakdown: `x-conditioning` 24, `x-obfuscation` 11,
`x-amplification` 6, `x-redirection` 6.

The 8 case studies documented via `grouping` / `report` / `note` objects are:
Ghostwriter / UNC1151, Star Blizzard, Macron Leaks, Sandworm × Brexit,
Volt Typhoon × Taiwan Municipal Election 2022, Volt Typhoon × Taiwan
Presidential Election 2024, Russia 2016 US Election Interference, and
Doppelgänger / RRN.

## Usage

### Any STIX 2.1 tooling

All three files are standard STIX 2.1 bundles and can be loaded with any STIX
library (e.g. the [`stix2`](https://stix2.readthedocs.io/) Python library) or
inspected as plain JSON. To resolve a relationship, look up its `source_ref`
and `target_ref` STIX IDs in `attack.bundle.json` or `disarm.bundle.json`.

### Loading into Neo4j (reproducible)

The graph can be reproduced in Neo4j from the three bundles. The mapping below
is described at the conceptual level so it can be re-implemented with any driver
(e.g. the official `neo4j` Python driver, or `APOC`'s JSON import). No specific
loader code is required.

**Node / edge mapping**

| STIX object | Neo4j element | Key properties |
|---|---|---|
| `attack-pattern` (ATT&CK & DISARM) | `(:AttackPattern)` node | `stix_id` (unique), `name`, `external_id` (T-code), `x_domain` (`cyber` for ATT&CK `Txxxx`, `cognitive` for DISARM `T0xxxx`), `description` |
| `x-role-category` | `(:RoleCategory)` node | `x_role_shortname` (unique), `name`, `description` |
| `relationship` (`relationship_type` = `x-<role>`) | directed edge `(:AttackPattern)-[:<ROLE>]->(:AttackPattern)` | resolves `source_ref` → `target_ref` against `AttackPattern.stix_id`; `description` |

**Procedure**

1. Create a uniqueness constraint on `AttackPattern.stix_id` (and on
   `RoleCategory.x_role_shortname`).
2. Parse the two official bundles and `MERGE` an `(:AttackPattern)` node for each
   `attack-pattern`. These bundles supply all nodes.
3. Parse `attack_pattern_relations.json` and `MERGE` a `(:RoleCategory)` node for
   each `x-role-category`.
4. For each `relationship`, `MERGE` a directed edge between the `AttackPattern`
   nodes whose `stix_id` equals `source_ref` and `target_ref`; type the edge by
   `relationship_type` and link it to the matching `(:RoleCategory)` via the
   `x_role_shortname` ↔ `relationship_type` (minus the `x-` prefix) string match.

Using `MERGE` throughout keeps the import idempotent (safe to re-run). The two
official bundles provide the nodes; the relations bundle provides the
cross-framework edges.

A working reference implementation of this projection (for a related demo,
importing a selected subset) is available in the project repository at
`coordinated-campaign-demo/data/scripts/import_stix_to_neo4j.py`, and the
projection design is documented in `docs/attack_pattern_relations_design.md`.

## Licensing & Attribution

This dataset combines material under three different licensing regimes. See
[`THIRD_PARTY_LICENSES/NOTICE.md`](./THIRD_PARTY_LICENSES/NOTICE.md) for the full
required notices.

| File | License | Attribution |
|---|---|---|
| `data/attack.bundle.json` | MITRE ATT&CK Terms of Use | © 2025 The MITRE Corporation. Reproduced and distributed with permission. |
| `data/disarm.bundle.json` | CC BY-SA 4.0 | DISARM Foundation |
| `data/attack_pattern_relations.json` | CC BY-SA 4.0 | This project (role taxonomy derived from JRC123305) |

- **MITRE ATT&CK®** is a registered trademark of The MITRE Corporation.
  Terms of Use: [`THIRD_PARTY_LICENSES/MITRE-ATTACK-LICENSE.txt`](./THIRD_PARTY_LICENSES/MITRE-ATTACK-LICENSE.txt).
- **DISARM** is © the DISARM Foundation, licensed under
  [CC BY-SA 4.0](./THIRD_PARTY_LICENSES/DISARM-LICENSE.md).
- The original contribution (`attack_pattern_relations.json`) is released under
  **CC BY-SA 4.0** (see [`LICENSE`](./LICENSE)) to satisfy DISARM's ShareAlike
  condition.
- The role-category taxonomy is derived from Giannopoulos et al. (2021),
  *The Landscape of Hybrid Threats*, JRC123305.

## References

Data sources and prior work:

```bibtex
@misc{mitre-attack-stix-data,
  author       = {{MITRE}},
  title        = {{ATT\&CK} {STIX} Data},
  howpublished = {\url{https://github.com/mitre-attack/attack-stix-data}},
  year         = {2026}
}

@inproceedings{terp2022disarm,
  author    = {Sara-Jayne Terp and Pablo C. Breuer},
  title     = {{DISARM}: A Framework for Analysis of Disinformation Campaigns},
  booktitle = {2022 {IEEE} Conference on Cognitive and Computational Aspects
               of Situation Management ({CogSIMA})},
  pages     = {1--8},
  year      = {2022},
  doi       = {10.1109/CogSIMA54611.2022.9830669}
}

@misc{cyberdatalab-disinfox,
  author       = {{CyberDataLab}},
  title        = {{DISINFOX}: Disinformation Incident Records},
  howpublished = {\url{https://github.com/CyberDataLab/disinfox}},
  year         = {2026}
}

@misc{gonzalez2025disinfox,
  author        = {Felipe S{\'a}nchez Gonz{\'a}lez and
                   Javier Pastor-Galindo and
                   Jos{\'e} A. Ruip{\'e}rez-Valiente},
  title         = {Toward Interoperable Representation and Sharing of
                   Disinformation Incidents in Cyber Threat Intelligence},
  year          = {2025},
  eprint        = {2502.20997},
  archiveprefix = {arXiv},
  primaryclass  = {cs.CR},
  url           = {https://arxiv.org/abs/2502.20997}
}
```
