# Third-Party Notices and Attributions

This dataset redistributes and builds upon material from third parties.
The notices below are required by the respective upstream licenses and are
provided for attribution. Full license texts are included in this directory.

---

## 1. MITRE ATT&CK®

`data/attack.bundle.json` is derived from the MITRE ATT&CK® STIX 2.1 data.

> © 2025 The MITRE Corporation. This work is reproduced and distributed with
> the permission of The MITRE Corporation.

ATT&CK® is a registered trademark of The MITRE Corporation. Use of the ATT&CK
data is governed by the MITRE ATT&CK Terms of Use, reproduced in
[`MITRE-ATTACK-LICENSE.txt`](./MITRE-ATTACK-LICENSE.txt).

- Source: https://github.com/mitre-attack/attack-stix-data

---

## 2. DISARM Framework

`data/disarm.bundle.json` is derived from the DISARM Framework, which is
licensed under the Creative Commons Attribution-ShareAlike 4.0 International
License (CC BY-SA 4.0). The full license text is reproduced in
[`DISARM-LICENSE.md`](./DISARM-LICENSE.md).

- DISARM Framework: https://github.com/DISARMFoundation/DISARMframeworks
- The DISARM Foundation, licensed under CC BY-SA 4.0.

The DISARM-derived STIX objects were produced with reference to DISINFOX
(disinformation incident records) by the CyberDataLab.

- DISINFOX: https://github.com/CyberDataLab/disinfox

---

## 3. Hybrid CoE / JRC role-category taxonomy

The four `x-role-category` objects in `data/attack_pattern_relations.json`
(Conditioning, Amplification, Redirection, Obfuscation) were systematically
derived from:

> Giannopoulos, G., Smith, H., Theocharidou, M. (eds.) (2021).
> *The Landscape of Hybrid Threats: A Conceptual Model.* JRC123305.
> Publications Office of the European Union.
> https://publications.jrc.ec.europa.eu/repository/handle/JRC123305

---

## 4. Original contribution

`data/attack_pattern_relations.json` (the role-category taxonomy objects, the
cross-framework relationship SROs, and the accompanying grouping/note/report
objects) is an original contribution of this project and is licensed under
CC BY-SA 4.0. See the top-level [`../LICENSE`](../LICENSE).

Because this contribution incorporates and adapts CC BY-SA 4.0 material
(DISARM), it is released under the same CC BY-SA 4.0 license to satisfy the
ShareAlike condition.
