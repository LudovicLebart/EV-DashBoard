# Référence — Nissan Leaf : trames et requêtes connues

**Tout ce qui suit est d'origine communautaire et marqué [à valider] sur capture réelle.** Facteurs d'échelle, signes, multiplexage et fréquences ne sont pas confirmés. Sources en bas de page.

## Plateformes

| Plateforme | Années | Batterie | Accès OBD |
|------------|--------|----------|-----------|
| AZE0 | 2013-2017 | 24 kWh, 30 kWh | 6/14 ; 12/13 (EV-CAN) **[communauté]** |
| ZE1 | 2018+ | 40 kWh et plus | Port isolé par passerelle ; 6/14 contact mis |

## Trames passives (EV-CAN, broches 12/13)

| ID | Fréquence | Contenu supposé | Statut |
|----|-----------|-----------------|--------|
| `0x1DB` | ~100 Hz | Courant et tension du pack | [communauté, à valider] |
| `0x1DA` | ~100 Hz | Régime et couple du moteur | [communauté, à valider] |
| `0x5BC` | ~10 Hz | GIDs restants | [communauté, à valider] |
| `0x5C0` | ~2 Hz | Températures (multiplexée) | [communauté, à valider] |
| `0x55B` | ? | SOC fin du LBC | [à valider] |
| `0x1DC` | ? | Limites de puissance charge/décharge | [à valider] |

## Requêtes de diagnostic (Car-CAN, broches 6/14)

Émetteur `0x79B` → batterie (LBC) `0x7BB`. Service `0x21` : *ReadDataByLocalIdentifier* hérité de **KWP2000** (non défini par UDS), transporté sur ISO-TP.

| Requête | Contenu supposé | Taille | Statut |
|---------|-----------------|--------|--------|
| `21 01` | Capacité (Ah), SOH, Hx | court | [communauté, à valider] |
| `21 02` | Tensions des 96 groupes de cellules | ~196 octets, plusieurs trames | [communauté, à valider] |
| `21 04` | Températures du pack | | [communauté, à valider] |
| `21 06` | Shunts d'équilibrage | | [à valider] |

## Constantes utiles

- GID : OVMS utilise **80 Wh/GID** ; ~77,5 Wh est une valeur « utile » communautaire. Le facteur est un **paramètre du profil**.
- Valeurs à neuf (paramètres OVMS) : ~356 GIDs / ~79 Ah (30 kWh), ~502 / ~115 (40 kWh), ~775 / ~176 (62 kWh) **[vérifié, paramètres OVMS]**.
- Topologie : 96 **groupes parallèles** (96S2P pour 24/30/40 kWh), pas 96 cellules.
- Sondes de température : 24 kWh = 4 ; 30 kWh = 3 vraisemblablement **[communauté, à valider]**.

## Points ouverts

- La passerelle route-t-elle les requêtes `0x79B/0x7BB` entre 6/14 et la batterie ? (LeafSpy fonctionne avec des dongles qui ne câblent que 6/14 : indice, pas preuve.)
- Source CAN de la puissance du chauffage : non identifiée.

## Sources

- Documentation OVMS, module Nissan Leaf : https://docs.openvehicles.com/en/latest/components/vehicle_nissanleaf/docs/index.html
- Documentation « leaf-obd », connecteur de diagnostic : https://leaf-obd.readthedocs.io/en/latest/diagnostic-connector.html
- Câble de dérivation ZE1 : https://community.openenergymonitor.org/t/ovms-ze1-40kwh-64kwh-2018-nissan-leaf-can-tap-cable-available/29798?page=2
