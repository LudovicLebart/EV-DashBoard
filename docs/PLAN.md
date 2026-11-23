# EV Dashboard — Plan de projet (révision 1)

> Télémétrie véhicule électrique, santé batterie et autonomie hivernale.
> Véhicule de référence : **Nissan Leaf AZE0 (2013-2017), 30 kWh en premier**, puis ZE1 (2018+).
> Architecture **véhicule-agnostique** : brancher une autre voiture = écrire un profil, pas modifier le firmware, le backend ou l'interface.
>
> Révision de l'ébauche « LeafSpy Maison — Télémétrie CAN & Autonomie hivernale » (9 pages). Date : 2026-11-23.

**Légende de fiabilité** utilisée dans tout le document :

- **[vérifié]** : confirmé par une source publique sérieuse (documentation OVMS, documentation constructeur de composant).
- **[communauté]** : rapporté de façon concordante par la communauté (forums, DBC communautaires), non confirmé officiellement.
- **[à valider]** : hypothèse plausible à confirmer **sur le véhicule ou dans le DBC** avant de s'y fier. Rien de marqué ainsi ne doit être codé en dur sans test sur capture réelle.

---

## 0. Résumé des changements majeurs par rapport à l'ébauche

| # | Ébauche | Révision | Pourquoi |
|---|---------|----------|----------|
| 1 | Un seul contrôleur CAN, TWAI en *Listen-Only*, **et** requêtes UDS émises sur le même bus | **Deux canaux CAN séparés** : EV-CAN (broches 12/13) **passif par construction matérielle** (émetteur du transceiver neutralisé), Car-CAN (6/14) **seul canal autorisé à émettre**, pour les requêtes de diagnostic via la passerelle, comme le fait un outil de diagnostic | Un contrôleur en Listen-Only ne peut ni émettre ni acquitter (ACK) : « zéro écriture » et « requêtes UDS » étaient contradictoires sur le même canal. On garde le bus moteur intouchable et on émet seulement là où le constructeur prévoit qu'on émette |
| 2 | Commutateur/cavalier 12/13 ↔ 6/14 | Les deux paires câblées en permanence ; le **profil** déclare quelles broches sont utilisées ; **par défaut seul 6/14 est activé** | Sur d'autres marques, 12/13 ou 3/11 portent autre chose (Ethernet de diagnostic BMW sur 3/11/12/13, MS-CAN Ford sur 3/11). Un cavalier mal placé sur une autre voiture = risque |
| 3 | Le module décode (driver véhicule dans l'ESP32) | **Le module enregistre les trames brutes**, le décodage se fait sur le serveur (et un sous-ensemble minimal sur le module pour l'affichage en direct) | Le firmware devient réellement agnostique (il ne connaît que débits, filtres et calendrier de requêtes). Un bug de décodage se corrige sans reflasher et **tout l'historique peut être redécodé** |
| 4 | « Aucun concept propriétaire ne sort du décodeur » (GIDs, Hx perdus) | Schéma canonique **+ champs d'extension par constructeur** (`x.nissan.gids`, `x.nissan.hx`) **+ trames brutes conservées** | Ne jamais jeter l'information ; le canonique sert à la comparaison, les extensions à la profondeur |
| 5 | « 80 % agnostique » | Liste explicite de ce qui est agnostique et de ce qui **ne l'est pas** (§2.4) | Le modèle d'autonomie, la gestion d'énergie du module, les seuils d'alerte cellulaires et même la définition du SOH dépendent du véhicule |
| 6 | Seuils 12 V : deep sleep sous 13,0 V (§3) mais coupure sous 12,4 V (§7) | Machine à états fondée sur **l'activité CAN** (critère principal) ; la tension sert à protéger la batterie 12 V, avec seuils à calibrer (§5.3) | 13,0 V n'est pas un indicateur fiable « voiture éteinte », et les deux seuils de l'ébauche se contredisaient |
| 7 | Pas de GPS, pas d'horloge, mais Phase 4 exige vitesse GPS et dénivelé | **Module GNSS ajouté** (position, altitude, **heure**) ; dénivelé tiré d'un modèle numérique de terrain, pas de l'altitude GNSS brute | Sans horloge temps réel, les journaux n'ont pas de date fiable ; le GNSS règle les deux problèmes |
| 8 | PWA connectée à l'ESP32 « via WebSocket ou BLE » | Constat : une PWA servie en HTTPS **ne peut pas** ouvrir un `ws://` local (contenu mixte), et **iOS ne supporte pas Web Bluetooth**. Décision dépendante du téléphone (question ouverte Q1) | Contrainte de plateforme bloquante non vue dans l'ébauche |
| 9 | 13 semaines | Horizon réaliste **6 à 9 mois à temps partiel** ; modèle d'autonomie v1 calibré **après un hiver complet de données** | Une personne, soirs et fins de semaine ; et l'hiver ne se rejoue pas |
| 10 | Phase 0 = développement logiciel | **Phase 0 = collecter des données dès cet hiver**, sans souder : LeafSpy Pro + dongle STN (journalisation), et captures brutes avec un adaptateur USB-CAN sur ordinateur portable | On est fin novembre : chaque semaine sans données est une semaine d'hiver perdue pour le modèle |
| 11 | Services « UDS 0x21 » | 0x21 = *ReadDataByLocalIdentifier* hérité de **KWP2000**, non défini par UDS (ISO 14229) ; transporté sur ISO-TP. La pile doit gérer 0x21 **et** 0x22, adressage normal **et** étendu, identifiants 11 **et** 29 bits | Prérequis pour Hyundai/Kia (anciens 0x21), BMW i3 (adressage étendu), etc. |
| 12 | Base Time-Series QuestDB ou InfluxDB | **PostgreSQL + TimescaleDB** pour les données décodées, **fichiers bruts immuables** comme source de vérité | Maturité, SQL standard, pas de rupture de version (InfluxDB v1→v2→v3), compression et agrégats continus |
| 13 | Matériel « fait maison » sans alternative | Option **acheter** (module OVMS v3, qui supporte déjà la Leaf) explicitement évaluée ; le système l'accepte comme source | La valeur du projet est dans les données et les modèles, pas forcément dans le boîtier (question ouverte Q3) |

### 0.1 Erreurs factuelles et imprécisions relevées dans l'ébauche

1. **Brochage Leaf** : EV-CAN sur 12/13 avec **13 = CAN-H, 12 = CAN-L** et Car-CAN sur 6 (H) / 14 (L) — cohérent avec plusieurs sources **[communauté]**, mais au moins une source OVMS inverse les libellés → **mesurer** avant de brancher (§5.1).
2. **ZE1 (2018+)** : ce n'est pas « vérifier la présence éventuelle d'une passerelle ». C'est un fait documenté : sur ZE1, le port OBD-II est **isolé par une passerelle CAN qui s'éteint au contact coupé** ; l'écoute passive de l'EV-CAN exige un **câble de dérivation** inséré sur le connecteur de la passerelle (24 broches, derrière le combiné d'instruments) **[vérifié, doc OVMS]**. Des propriétaires de 2018 signalent même des broches 12/13 non équipées au port OBD **[communauté]**. L'interrogation du LBC via le port OBD fonctionne en revanche contact mis (LeafSpy fonctionne sur ZE1) **[communauté]**.
3. **Énergie par GID** : OVMS utilise **80 Wh/GID** par défaut, LeafSpy affiche aussi ses kWh sur cette base ; 77,5 Wh est une valeur « utile mesurée » d'origine communautaire. Le GID **n'est pas** une mesure d'énergie utilisable jusqu'à 0 : il y a une réserve. GIDs à neuf : ≈ 356 (30 kWh), ≈ 502 (40 kWh), ≈ 775 (62 kWh) ; Ah à neuf ≈ 79 / 115 / 176 **[vérifié, paramètres OVMS]**. → le facteur Wh/GID est un **paramètre du profil**, pas une constante.
4. **Nombre de sondes thermiques** : « 4 sondes » est vrai pour les packs 24 kWh d'origine ; **le pack 30 kWh n'en expose vraisemblablement que 3** **[communauté, à valider]**. → le profil déclare le nombre, le schéma accepte un tableau de taille variable et des valeurs nulles.
5. **96 « paires »** : correct pour 24/30/40 kWh (192 cellules, 96S2P) ; le 62 kWh est en 96S3P **[communauté]**. On parle donc de **96 groupes parallèles**, pas de 96 cellules. La tension mesurée par le LBC est celle du groupe ; une cellule faible dans un groupe est masquée en partie par ses voisines.
6. **Trames passives** : 0x1DB (courant/tension pack, ~10 ms), 0x5BC (GIDs, ~100 ms), 0x5C0 (températures, multiplexée, ~500 ms), 0x1DA (onduleur : régime, couple, ~10 ms) sont des identifiants connus de la communauté **[communauté]**, mais **facteurs d'échelle, signe du courant, multiplexage et fréquences sont [à valider]** sur capture. L'ébauche oublie des trames utiles : SOC fin du LBC (0x55B), limites de puissance charge/décharge (0x1DC), climatisation (0x54B / 0x54F ou voisines) **[à valider]**.
7. **La puissance du chauffage (`hvac_power_w`)** est listée comme champ canonique fourni par le driver : sur Leaf, sa source CAN n'est **pas identifiée** dans l'ébauche. **[à valider]** : peut n'être disponible qu'estimée (bilan pack − traction − auxiliaires).
8. **Tesla Model 3/Y** : il n'y a **pas de prise OBD-II** ; l'accès passe par un connecteur de diagnostic interne dont la référence varie selon l'année. « Facile, matériel identique, seul le câble change » est optimiste : le DBC est rétro-ingénié et **change avec les mises à jour logicielles**.
9. **CAN-FD** : le TWAI de l'ESP32 est CAN 2.0 uniquement. Point non mentionné : un contrôleur CAN classique **en mode normal** sur un bus qui transporte des trames FD les interprète comme des erreurs et **émet des trames d'erreur qui perturbent le bus**. En Listen-Only il ne perturbe pas, mais ne peut rien lire des trames FD. → interdiction absolue du mode normal sur un bus FD avec un contrôleur classique.
10. **ESP32-C3** : un seul contrôleur TWAI, un seul cœur, ~400 Ko de RAM → insuffisant pour deux bus + carte SD + Wi-Fi à ~2000 trames/s.
11. **Transceiver** : le TJA1051T/3 a un mode *silent* mais **pas de mode veille avec réveil par le bus** ; le SN65HVD230 a un mode veille à environ 370 µA. Pour un réveil sur activité CAN à faible consommation, il faut un transceiver avec veille et réveil par le bus (famille TJA1042/TJA1043, TCAN1042, MCP2562 selon variante) **[à valider sur fiche technique]**.
12. **« Deep sleep < 1 mA »** n'est pas atteignable avec une carte de développement « Super Mini » (régulateur, LED, pont USB) ni avec un convertisseur Buck ordinaire (courant de repos de plusieurs mA). Le budget de veille se conçoit au niveau de la carte, pas du microcontrôleur.
13. **Critère de validation ISO-TP « 96 tensions cohérentes avec la tension pack (±0,5 V) »** : la somme des 96 tensions et la tension pack ne sont pas mesurées au même instant ni par la même chaîne ; ±0,5 V sur ~360 V est irréaliste sous charge. Critère révisé au §7.
14. **Critère « erreur < 5 % sur les trajets hivernaux »** : non mesurable tel quel (erreur de quoi, mesurée quand, sur combien de trajets ?). Critère révisé au §7.

---

## 1. Vision et objectifs

EV Dashboard est une plateforme personnelle qui **enregistre en continu** les données d'un véhicule électrique, les **archive à la maison**, et en tire des indicateurs que l'ordinateur de bord ne donne pas.

Capacités visées (inchangées sur le fond, reformulées pour être mesurables) :

1. **Télémétrie autonome** : chaque trajet et chaque recharge enregistrés sans action du conducteur.
2. **Santé cellulaire** : détecter les groupes de cellules qui décrochent sous charge ou par grand froid (écart et résistance interne relative), avec un niveau de confiance statistique, pas sur une seule mesure.
3. **Autonomie hivernale** : prédire l'énergie nécessaire pour un trajet donné (température, chauffage, vitesse, dénivelé) et la marge restante.
4. **Vieillissement long terme** : SOH, Hx, Ah, dispersion cellulaire, sessions de recharge sur plusieurs années.
5. **Sentinelle 12 V** : suivi de la batterie auxiliaire et alerte avant la panne.

**Non-objectifs** (pour ne pas dériver) : aucune écriture de configuration, aucun effacement de codes défaut, aucun pilotage du véhicule, pas de service cloud public, pas de compatibilité « toutes voitures » sans profil.

### 1.1 Ce que « plugger n'importe quelle voiture » veut dire concrètement

On définit trois **niveaux de support**, pour que la promesse soit vérifiable :

| Niveau | Ce que l'on obtient | Ce qu'il faut |
|--------|---------------------|---------------|
| **N0 — Générique** | Enregistrement brut de tout ce qui circule sur 6/14 (passif) + PID OBD-II standard (mode 01) **si** le véhicule y répond (vitesse, température extérieure, tension module 0x42…). Aucune donnée batterie garantie : **les VÉ ne sont pas tenus de répondre aux PID d'émissions** | Rien : profil `generic-obd2` livré |
| **N1 — Communautaire** | Signaux batterie de base (tension, courant, SOC, éventuellement cellules) depuis un DBC/liste de PID publics | Un profil (YAML + DBC) + captures de test |
| **N2 — Référence** | Tout le pipeline : cellules, santé, modèle d'autonomie calibré | Profil complet + calibration du modèle sur les trajets de ce véhicule |

La Leaf AZE0 vise N2. La preuve d'agnosticisme est un **critère de sortie** (Phase 6) : un deuxième véhicule d'une autre famille amené en N1 **sans modifier le firmware, le cœur du backend ni la PWA**.

---

## 2. Architecture

### 2.1 Vue d'ensemble

```
 Véhicule                    Module embarqué                       Maison                         Conducteur
┌──────────────┐  passif   ┌───────────────────────────┐  Wi-Fi  ┌──────────────────────────┐   ┌──────────────┐
│ EV-CAN 12/13 ├──(RX)────►│ Canal B : écoute seule     │ (HTTPS) │ API d'ingestion          │   │ PWA          │
│              │           │                            │────────►│  ├ fichiers bruts (vérité)│──►│  historique  │
│ Car-CAN 6/14 ├◄─(TX/RX)─►│ Canal A : diag autorisé    │         │  ├ décodeur (profils)     │   │  santé       │
│              │           │ Ordonnanceur de requêtes   │         │  ├ TimescaleDB (canonique)│   │  autonomie   │
│ +12 V broche 16 ────────►│ Mesure 12 V, alimentation  │         │  └ analyses / modèles     │   └──────────────┘
└──────────────┘           │ GNSS (heure, position)     │         └──────────────────────────┘          ▲
                           │ Journal brut sur microSD   │                                               │
                           │ Décodage minimal (direct)  │──── BLE / écran dédié (en conduite) ──────────┘
                           └───────────────────────────┘
 Autres sources : LeafSpy Pro (CSV), dongle ELM/STN, module OVMS, API cloud constructeur → même ingestion
```

Principe directeur : **le module est un enregistreur générique** ; toute connaissance du véhicule vit dans le **profil** ; le serveur décode et analyse ; la PWA ne connaît que le **schéma canonique**.

### 2.2 Transports (sources de données)

Chaque source est un « transport » qui produit soit des **trames brutes horodatées**, soit directement des **échantillons canoniques**. Le profil déclare les transports qu'il supporte et, pour chacun, les signaux disponibles.

| Transport | Passif plein débit | Requêtes | Usage dans ce projet |
|-----------|--------------------|----------|----------------------|
| `can_raw` (module EV Dashboard, USB-CAN type candleLight) | Oui | Oui (canal A seulement) | Cible principale |
| `elm_stn` (OBDLink CX/MX+, vLinker) | **Non** : le mode moniteur ELM sature la liaison série/BLE bien avant 2000 trames/s ; les clones ELM327 bon marché échouent souvent sur les réponses multi-trames longues | Oui, quelques requêtes/s | Phase 0 (validation des requêtes LBC), dépannage |
| `file_import` (CSV LeafSpy Pro, journaux candump/SavvyCAN) | Selon fichier | — | Collecte de cet hiver ; tests de rejeu |
| `ovms` (module OVMS v3 via MQTT/serveur v3) | Non (métriques déjà décodées) | — | Alternative « acheter » (Q3) |
| `cloud_api` (Tesla Fleet API, Smartcar…) | Non, faible fréquence | — | Hors périmètre initial ; prévu dans le schéma (données clairsemées). Sans objet pour la Leaf AZE0 en Amérique du Nord (télématique 3G arrêtée) **[à valider]** |

### 2.3 Responsabilités par couche

| Couche | Fait | Ne fait pas |
|--------|------|-------------|
| **Firmware** (ESP-IDF, C) | Capture CAN horodatée, journal brut avec CRC, ordonnanceur de requêtes déclaratif, ISO-TP, mesure 12 V, GNSS, gestion d'énergie, synchronisation Wi-Fi, décodage **minimal** (tableau compilé depuis le profil) pour l'affichage direct | Interpréter la sémantique (SOH, kWh utiles), stocker l'historique, modèles |
| **Backend** (Python, FastAPI) | Ingestion idempotente, archivage brut, décodage (cantools + ISO-TP + décodeurs de réponses), conversion canonique, dérivés (puissance, énergie intégrée), segmentation trajets/recharges, analyses, modèles, API | Connaître un véhicule autrement que par son profil (+ greffon optionnel) |
| **PWA** (React + Vite) | Afficher le canonique, l'historique, les capacités du véhicule (« non disponible » ≠ 0) | Décoder des trames, contenir des identifiants CAN |

### 2.4 Ce qui est agnostique, ce qui ne l'est pas

**Réellement agnostique (aucun changement par véhicule)** : format de journal brut, firmware (capture, ordonnanceur, ISO-TP, énergie, synchronisation), ingestion, stockage, moteur de décodage, schéma canonique, segmentation trajets/recharges (sur signaux canoniques), algorithmes statistiques (dispersion cellulaire, régression de résistance interne), cadre du modèle de consommation, PWA.

**Propre à chaque véhicule (dans le profil ou sa calibration)** :

- couche physique : broches, débits, présence d'une passerelle, CAN-FD, adressage 11/29 bits, adressage ISO-TP normal/étendu ;
- dictionnaire de décodage (DBC, requêtes, mise en page des réponses) ;
- **comportement de veille du véhicule** (quand le bus se tait, réveils nocturnes de la télématique, du chargeur) — il conditionne directement la gestion d'énergie du module ;
- **sémantique** : le SOH d'un constructeur n'est pas celui d'un autre ; le SOC « BMS » n'est pas le SOC affiché ; réserve sous 0 % ;
- topologie batterie (96S2P, 98S…), nombre et position des sondes, chimie → **seuils d'alerte cellulaires** ;
- **modèle d'autonomie** : masse, SCx, roulement, rendement chaîne de traction, régénération, type de chauffage (résistif PTC ou thermopompe — les deux existent sur AZE0 selon la finition **[à valider pour ta voiture]**), préconditionnement ; tout cela se **calibre par véhicule** sur ses propres trajets ;
- comportement 12 V (stratégie du convertisseur DC/DC, recharge 12 V à l'arrêt ou non).

Conclusion honnête : le **code** peut être agnostique à ~100 % ; la **valeur analytique** ne l'est pas — un nouveau véhicule arrive en N1 rapidement, en N2 seulement après une saison de données.

---

## 3. Contrat de données canonique

### 3.1 Règles générales

- **Unités SI explicites dans le nom** (`_v`, `_a`, `_w`, `_wh`, `_c`, `_pct`, `_kmh`, `_m`). Pas de kWh/kW dans le stockage (Wh, W), conversion à l'affichage.
- **Conventions de signe** : `pack_current_a` **positif en décharge** (énergie sortant de la batterie), négatif en recharge/régénération ; `pack_power_w` suit le même signe. Le profil déclare la convention de sa source et la transformation.
- **Nullabilité** : un signal absent est `null` / absent, **jamais 0**. Une capacité non déclarée par le profil n'est jamais affichée comme une valeur.
- **Provenance** de chaque série : `measured` (lu du véhicule), `derived` (calculé par le backend, ex. V×I), `model` (sortie d'un modèle, ex. énergie utilisable estimée). La PWA l'indique.
- **Horodatage** : `ts` UTC en µs + `ts_quality` (`gnss`, `ntp`, `interpolated`, `unknown`). Voir §3.5.
- **Versionnage** : schéma en SemVer (`canonical_schema: 1.0.0`). Ajout de champ = mineure ; renommage/changement d'unité = majeure avec migration. Chaque profil déclare la version visée ; l'ingestion refuse un profil d'une version majeure inconnue.
- **Extensions constructeur** : espace de noms `x.<marque>.<nom>` (`x.nissan.gids`, `x.nissan.hx`, `x.nissan.lbc_soh_pct`), même règles d'unités et de nullabilité, déclarées dans le profil. Jamais utilisées par la PWA générique, sauf vue « avancée » qui les liste dynamiquement.

### 3.2 Signaux scalaires (v1)

| Signal | Unité | Type | Notes |
|--------|-------|------|-------|
| `pack_voltage_v` | V | float | |
| `pack_current_a` | A | float | + = décharge |
| `pack_power_w` | W | float | `derived` si non fourni |
| `soc_bms_pct` | % | float | SOC rapporté par le BMS (définition propre au constructeur) |
| `soc_display_pct` | % | float | SOC affiché au conducteur, si disponible |
| `energy_remaining_bms_wh` | Wh | float | Énergie restante selon le BMS (Leaf : GIDs × Wh/GID du profil) |
| `energy_usable_est_wh` | Wh | float | **`model`** uniquement : énergie réellement utilisable avant réserve |
| `capacity_ah` | Ah | float | Capacité estimée par le BMS |
| `soh_pct` | % | float | **Définition constructeur** ; `soh_definition` déclaré dans le profil (`capacity`, `bms_internal`…) |
| `cell_v_min_mv`, `cell_v_max_mv`, `cell_v_spread_mv` | mV | float | `derived` depuis le tableau, ou fourni |
| `pack_temp_min_c`, `pack_temp_max_c` | °C | float | |
| `charge_power_limit_w`, `discharge_power_limit_w` | W | float | |
| `vehicle_speed_kmh` | km/h | float | Source CAN de préférence (GNSS en secours) |
| `motor_speed_rpm`, `motor_torque_nm` | tr/min, N·m | float | |
| `hvac_power_w` | W | float | Peut n'être que `derived`/`model` (Leaf : **[à valider]**) |
| `ambient_temp_c`, `cabin_temp_c` | °C | float | |
| `aux_12v_v` | V | float | Mesure du module (§5.3) et/ou valeur véhicule (`source` distincte) |
| `odometer_km` | km | float | |
| `vehicle_state` | enum | `off`, `awake`, `ready`, `charging_ac`, `charging_dc` | Déduit par le profil |
| `gnss_lat`, `gnss_lon` | ° | float | Sensible (vie privée) |
| `gnss_alt_m`, `elevation_dem_m` | m | float | `elevation_dem_m` = `derived` depuis un modèle de terrain |

### 3.3 Signaux tableaux

| Signal | Forme | Notes |
|--------|-------|-------|
| `cell_voltages_mv[]` | `float[n]` | `n` déclaré par le profil (96 pour la Leaf). Index = position BMS, documentée dans le profil. Instantané **non simultané** : le BMS les échantillonne en séquence |
| `pack_temps_c[]` | `float?[k]` | `k` déclaré ; une sonde absente = `null` |
| `cell_balancing[]` | `bool[n]` | Shunts actifs, si disponible |

Chaque instantané tableau porte `ts_request` et `ts_response` (latence connue) et le courant pack échantillonné le plus proche, pour les calculs de résistance.

### 3.4 Capacités déclarées

Pour chaque signal, le profil déclare : transport, `rate_hz` attendu, résolution, `provenance`, condition de disponibilité (ex. `vehicle_state in [ready, charging_*]`). L'API expose `GET /vehicles/{id}/capabilities` ; la PWA construit ses vues à partir de cette liste.

### 3.5 Contrat firmware → backend : journal brut `EVLOG v1`

- Un fichier par **session** (réveil → veille), pré-alloué, écrit en **blocs de 4 à 16 Ko**, chaque bloc avec en-tête (numéro, longueur, CRC32), compressé (heatshrink ou LZ4) **[choix à benchmarker]**.
- En-tête de fichier : magie, version du format, identifiant du module, version firmware, **identifiant + empreinte du profil**, numéro de démarrage, horodatage d'ancrage s'il est connu.
- Enregistrements :
  - trame CAN : canal, Δt en µs (horloge monotone), identifiant, drapeaux (étendu, RTR, **TX propre**), DLC, données ;
  - événement : réveil (cause), passage en veille, erreurs bus (compteurs TEC/REC, bus-off), **trames perdues** (compteur de débordement du FIFO), échec de requête ;
  - mesure analogique : tension 12 V, température du module ;
  - GNSS : position, vitesse, **heure UTC** (ancrage temporel) ;
  - ancrage temporel : (monotone µs ↔ UTC, source NTP/GNSS).
- Le module enregistre aussi **ses propres requêtes émises** : le backend réassemble l'ISO-TP à partir du journal, sans faire confiance au réassemblage embarqué.
- **Gestion de l'heure sans RTC** : l'horloge monotone survit au deep sleep (temporisateur RTC de l'ESP32), mais l'oscillateur RC interne dérive de plusieurs % → prévoir un **quartz 32,768 kHz** sur la carte, et s'appuyer sur les ancrages GNSS (chaque trajet) et NTP (chaque synchronisation). Le backend interpole entre ancrages et attribue `ts_quality`.

### 3.6 Contrat de synchronisation (module → serveur)

- Déclenchement : véhicule à l'arrêt (bus silencieux) **et** SSID domestique visible **et** tension 12 V au-dessus du seuil de préservation.
- `PUT /v1/devices/{device_id}/sessions/{session_id}?offset=N` par morceaux ; le serveur répond l'offset validé (reprise après coupure) ; `POST …/commit` avec SHA-256 du fichier. **Idempotent**.
- HTTPS (mbedTLS) avec jeton par module ; nom d'hôte fixe sur le réseau local.
- Le module **n'efface un fichier qu'après accusé de validation** et seulement s'il a besoin de place (stockage circulaire).
- Durée de synchronisation bornée (ex. 10 min) pour limiter la consommation 12 V.

### 3.7 Contrat backend → PWA

- REST + JSON, version du schéma dans chaque réponse (`schema_version`), `GET /schema` (liste des signaux, unités, provenances).
- Séries temporelles : `GET /vehicles/{id}/series?signals=…&from=…&to=…&resolution=…` (agrégats continus côté base).
- En direct (module → téléphone) : trame compacte versionnée à 1-5 Hz (sous-ensemble canonique), indépendante du transport (BLE GATT ou autre, §6.4).

---

## 4. Profil véhicule

### 4.1 Format

Un profil = un dossier versionné dans `profiles/<marque>/<modèle>/` :

- `profile.yaml` : métadonnées, bus, requêtes, correspondances vers le canonique, capacités, extensions, paramètres de calibration, liste blanche d'émission ;
- `*.dbc` : trames passives (décodées par `cantools` côté serveur ; un générateur compile les quelques signaux nécessaires à l'affichage direct en tableau binaire pour le firmware) ;
- `decoders.py` (optionnel, **greffon backend uniquement**) : pour ce qu'un DBC ne sait pas exprimer (réponse ISO-TP de 196 octets découpée en 96 valeurs, trames multiplexées exotiques). Le firmware n'exécute jamais de code de profil ;
- `captures/` : journaux de référence **anonymisés** (VIN masqué, GNSS retiré) + valeurs attendues (tests « golden »).

Les transformations autorisées dans le YAML sont volontairement limitées : facteur, décalage, signe, table de correspondance, bornes de validité. Tout le reste va dans `decoders.py` avec tests.

### 4.2 Exemple : Nissan Leaf AZE0 30 kWh (extrait)

Toutes les valeurs marquées `# A VALIDER` doivent être confirmées sur capture avant usage.

```yaml
profile: nissan/leaf-aze0-30kwh
profile_version: 0.1.0
canonical_schema: 1.0.0
vehicle:
  make: Nissan
  model: Leaf
  platform: AZE0
  years: [2016, 2017]          # 30 kWh ; un profil 24 kWh dérivera de celui-ci
  battery:
    topology: 96S2P
    cell_groups: 96
    temp_sensors: 3            # A VALIDER (4 sur 24 kWh)
    nominal_gids_new: 356      # source : paramètres OVMS
    nominal_ah_new: 79
  calibration:
    wh_per_gid: 80.0           # convention OVMS/LeafSpy ; ~77,5 "utile" selon communauté -> à calibrer
    reserve_wh: null           # à établir (LBW/VLBW), ne pas rouler jusqu'au mode tortue pour le mesurer
    hvac_type: unknown         # ptc | heat_pump  (selon finition S/SV/SL) A VALIDER

buses:
  ev_can:
    pins: {h: 13, l: 12}       # A VALIDER au multimètre/oscilloscope
    bitrate: 500000
    mode: listen_only          # imposé ; le matériel du canal B ne peut pas émettre
    dbc: leaf_ev_can.dbc
  car_can:
    pins: {h: 6, l: 14}
    bitrate: 500000
    mode: diagnostic           # émission autorisée UNIQUEMENT selon tx_whitelist
    dbc: leaf_car_can.dbc

vehicle_state:                 # comment le profil déduit l'état
  awake_if: {bus: ev_can, any_frame_within_ms: 500}
  charging_if: {signal: x.nissan.charge_status, in: [ac, dc]}   # signal A IDENTIFIER

passive:                       # correspondances DBC -> canonique
  - {msg: 0x1DB, signal: LB_Current, to: pack_current_a, sign: -1}   # signe A VALIDER
  - {msg: 0x1DB, signal: LB_Voltage, to: pack_voltage_v}
  - {msg: 0x5BC, signal: LB_Remain_GIDs, to: x.nissan.gids}
  - {msg: 0x55B, signal: LB_SOC, to: soc_bms_pct}                    # A VALIDER
  - {msg: 0x1DC, signal: LB_Discharge_Power_Limit, to: discharge_power_limit_w, factor: 1000}  # A VALIDER
  - {msg: 0x1DA, signal: Motor_Speed, to: motor_speed_rpm}
derived:
  - {to: energy_remaining_bms_wh, expr: "x.nissan.gids * calibration.wh_per_gid"}

tx_whitelist:                  # vérifiée par le firmware, en plus de l'ordonnanceur
  - {bus: car_can, id: 0x79B, services: [0x21], data_prefixes: ["21 01", "21 02", "21 04", "21 06"]}
  - {bus: car_can, id: 0x79B, flow_control: true}

polls:
  lbc_g1:
    bus: car_can
    request: {tx: 0x79B, rx: 0x7BB, addressing: normal, data: "21 01"}   # KWP2000 ReadDataByLocalIdentifier
    isotp: {fc_block_size: 0, fc_st_min_ms: 0, timeout_ms: 1000}
    schedule: {on: [session_start, session_end], every_s: 60, when: vehicle_state in [ready, charging_ac, charging_dc]}
    decode: decoders.lbc_group1        # -> capacity_ah, soh_pct, x.nissan.hx, x.nissan.lbc_soh_pct
  lbc_cells:
    bus: car_can
    request: {tx: 0x79B, rx: 0x7BB, data: "21 02"}  # ~196 octets : 1 First Frame + ~28 Consecutive Frames
    schedule:
      every_s: 30
      burst: {when: "abs(pack_current_a) > 80", min_interval_ms: 500, max_per_event: 10}
    decode: decoders.lbc_cell_voltages # -> cell_voltages_mv[96]
  lbc_temps:
    bus: car_can
    request: {tx: 0x79B, rx: 0x7BB, data: "21 04"}
    schedule: {every_s: 60}
    decode: decoders.lbc_temps         # -> pack_temps_c[3] (null si sonde absente)
  lbc_shunts:
    bus: car_can
    request: {tx: 0x79B, rx: 0x7BB, data: "21 06"}   # A VALIDER
    schedule: {every_s: 300}
    decode: decoders.lbc_shunts        # -> cell_balancing[96]

power_management:              # spécifique véhicule, mesuré en Phase 0
  bus_silence_before_sleep_s: 30     # A MESURER : les calculateurs Leaf bavardent-ils après l'arrêt ?
  expected_night_wakeups: unknown    # télématique, minuteries de charge/climatisation
```

Hypothèse importante **[communauté, à valider en Phase 0]** : LeafSpy interroge le LBC avec des dongles ELM qui ne câblent que 6/14, donc la passerelle route les requêtes 0x79B/0x7BB entre Car-CAN et EV-CAN. Si ce n'est pas le cas sur ta voiture, le plan de repli est d'émettre sur l'EV-CAN **uniquement** contact mis, avec les mêmes garde-fous (§5.2), ce qui exige un canal B capable d'émettre — décision à prendre après mesure, pas avant.

### 4.3 Profil ZE1 (plus tard)

Même LBC, extensions différentes (GIDs, groupes et éventuellement facteurs différents **[à valider]**), et surtout **couche physique différente** : EV-CAN accessible seulement par un câble de dérivation sur le connecteur de la passerelle ; requêtes via le port OBD contact mis seulement. Le firmware ne change pas ; le câble et le profil oui.

---

## 5. Matériel et sécurité

### 5.1 Décisions matérielles

| Élément | Décision | Justification / alternative |
|---------|----------|-----------------------------|
| Microcontrôleur | **ESP32-C6** (module -40/+85 °C) | **Deux contrôleurs TWAI natifs** (supportés à partir d'ESP-IDF 5.2, API TWAI v2) **[vérifié, doc Espressif]** : un par bus, sans MCP2515 en SPI. Wi-Fi 6 + BLE. Risque : un seul cœur à 160 MHz → **benchmark obligatoire** (2 bus pleins + SD + Wi-Fi) en Phase 2a. Repli : ESP32-S3 + MCP2515 pour le canal de diagnostic (faible débit). **Éviter les modules S3 à PSRAM octale (N8R8/N16R8), limités à 65 °C** sans ECC **[vérifié, fiche Espressif]** : un habitacle d'été dépasse cela. ESP32-C3 écarté (§0.1 n°10) |
| Canal B (EV-CAN, passif) | Transceiver avec veille/réveil par le bus, **broche TXD non reliée au MCU et forcée à l'état récessif, mode silencieux câblé** | Passivité garantie par le matériel, pas seulement par le logiciel. Le contrôleur TWAI est en plus configuré en Listen-Only |
| Canal A (Car-CAN 6/14, diagnostic) | Transceiver 3,3 V avec veille/réveil (famille TCAN1042/TJA1042, **[à choisir sur fiche technique]**) | Sa broche de réveil sert aussi au réveil du module sur activité CAN |
| Alimentation | Protection d'entrée (fusible 1 A, TVS dimensionnée pour la décharge de charge ISO 7637/16750, protection inversion) + **régulateur à faible courant de repos (< 30 µA), entrée ≥ 40 V** | Le courant de repos du régulateur domine le budget de veille |
| Mesure 12 V | Diviseur **avant** la diode de protection + **ADC externe** (type INA226/ADS1115) ou étalonnage deux points | L'ADC ESP32 (±50-100 mV) ne distingue pas 12,4 V de 12,5 V ; or ces 100 mV séparent ~75 % et ~85 % de charge d'une batterie au plomb |
| Stockage | microSD industrielle (pSLC, -40/+85 °C), SPI | Fichiers pré-alloués, écriture par blocs, `fsync` toutes les 1-2 s, CRC par bloc → au pire la dernière seconde est perdue en cas de coupure |
| GNSS | Module u-blox M10 (ou équivalent) + antenne active, alimentation coupée en veille | Position, vitesse, **heure UTC** ; dénivelé via modèle de terrain |
| Horloge | Quartz 32,768 kHz sur le domaine RTC | Dérive de l'horloge en deep sleep |
| Connectique | Câble OBD-II mâle court + boîtier ASA (pas de PETG : ramollit vers 75-80 °C), soulagement de traction | Ne gêne ni les pédales ni les genoux ; aucun fil d'origine coupé |
| Batterie de secours | **Aucune** (conservé de l'ébauche, justifié) | |

**Consommation (budget à valider sur banc)** : veille totale **≤ 0,5 mA** visée (soit ~0,36 Ah/mois, négligeable devant une batterie de ~40 Ah) ; trajet ~150 mA (GNSS + 2 bus + SD) ; synchronisation Wi-Fi ~150-250 mA pendant quelques minutes.

**Volume de données (à mesurer en Phase 0)** : si l'EV-CAN porte ~1500 trames/s, à ~16 octets/trame brute → ~85 Mo/h avant compression. Une carte de 32 Go tient > 300 h ; le Wi-Fi domestique transfère une heure de conduite en quelques minutes. On peut donc **tout enregistrer** au début ; la décimation par identifiant (déclarée dans le profil) viendra si la mesure l'exige.

### 5.2 Règles de sécurité bus (non négociables)

1. **Aucune émission sur l'EV-CAN** (matériel + logiciel).
2. **Liste blanche d'émission compilée dans le firmware** : identifiants et préfixes de service autorisés (0x21, 0x22 en lecture, trames de *Flow Control*). Un profil ne peut **pas** élargir la liste sans recompilation avec un drapeau explicite. Interdits : 0x10 (sauf session par défaut si un profil l'exige et après revue), 0x11, 0x14, 0x27, 0x28, 0x2E, 0x2F, 0x31, 0x34-0x37, 0x3D, 0x85.
3. **Jamais d'émission si le véhicule dort** : émettre seulement si du trafic a été vu sur le bus dans les 500 dernières ms et que l'état est `ready` ou `charging_*`. Émettre sur un bus endormi réveille les calculateurs, vide la batterie 12 V et peut laisser des codes défaut.
4. **Débit vérifié avant le mode normal** : démarrage en Listen-Only sur le canal A ; passage en mode normal seulement après N trames valides sans erreur au débit attendu. Un débit faux en mode normal génère des trames d'erreur.
5. **Bus-off ou compteurs d'erreurs en hausse** → arrêt des requêtes pour la session, événement journalisé, pas de récupération automatique agressive.
6. **Limitation de débit** des requêtes (ex. ≤ 10 requêtes/s au total) et une seule requête ISO-TP en vol par calculateur.
7. **CAN-FD** : si le profil déclare un bus FD, le contrôleur TWAI n'y est utilisable **qu'en Listen-Only** (et ne verra pas les trames FD) ; tout le reste exige un contrôleur FD.
8. **Broches non standard (12/13, 3/11…)** désactivées par défaut ; activées seulement par un profil qui les déclare.

### 5.3 Gestion d'énergie et 12 V (remplace les seuils contradictoires)

Machine à états (le critère principal est l'**activité CAN**, la tension sert à **protéger**) :

| État | Entrée | Sortie |
|------|--------|--------|
| `LOGGING` | Réveil sur activité CAN | Bus silencieux depuis `bus_silence_before_sleep_s` (profil, mesuré) → `SYNC` ou `SLEEP` |
| `SYNC` | Silence + SSID domestique + `V12 > V_sync` | Fin de transfert, durée max atteinte, ou `V12 < V_sync` → `SLEEP` |
| `SLEEP` | — | Réveil CAN → `LOGGING` ; minuterie (ex. toutes les 6 h) → `SENTINEL` |
| `SENTINEL` | Minuterie | Mesure 12 V (au repos depuis ≥ 4 h : seule une tension de repos a un sens), alerte via Wi-Fi si disponible → `SLEEP` |
| `PRESERVE` | `V12 < V_preserve` | Plus de Wi-Fi ni de GNSS ; seul le réveil CAN reste actif → `LOGGING` quand le véhicule se réveille (le DC/DC remontera la tension) |

Seuils de départ (**à calibrer** selon la technologie de la batterie : plomb ouvert ou AGM) : `V_sync = 12,3 V`, alerte à 12,4 V au repos, `V_preserve = 12,0 V`. Une tension > ~13,2 V indique que le DC/DC est actif — information utile, **pas** critère de veille.

Points à mesurer en Phase 0/2 : combien de fois par nuit le bus se réveille (télématique, minuteries) ; les premières centaines de ms après un réveil sont perdues (démarrage du module) — accepté et documenté ; consommation pendant une recharge de 8 h (le DC/DC alimente, sans risque).

### 5.4 Autres risques de sécurité et de conformité

- **Sécurité fonctionnelle** : le projet n'est pas développé selon ISO 26262 ; d'où la doctrine « lecture seule, liste blanche, passivité matérielle sur le bus moteur ». LeafSpy interroge le LBC en roulant depuis des années sans incident connu, ce qui est un indice, pas une preuve.
- **Garantie** : un accessoire OBD n'annule pas la garantie en soi, mais un dommage qui lui est attribuable peut être refusé. Une AZE0 2016-2017 peut encore être sous garantie de capacité batterie (8 ans **[à valider selon pays et date]**) : ne rien faire qui puisse être interprété comme une intervention sur le LBC.
- **Distraction** : affichage en conduite fixé sur support, aucune interaction en roulant (Code de la route local).
- **Vie privée** : le GNSS est un journal de déplacements ; le VIN apparaît dans certaines réponses de diagnostic. **Aucune capture brute, VIN ou trace GNSS dans un dépôt public** ; les captures de test sont anonymisées.

---

## 6. Choix technologiques (tranchés)

| Domaine | Choix | Écartés et pourquoi |
|---------|-------|---------------------|
| Firmware | **ESP-IDF (C) 5.5.x** via CMake ; PlatformIO toléré comme simple lanceur | Arduino : couche TWAI multi-contrôleur en retard, moins de contrôle sur IRAM/interruptions (débordements FIFO TWAI pendant les accès flash/Wi-Fi : ISR en IRAM obligatoire) |
| Décodage serveur | **Python : `cantools` (DBC), `python-can` (formats de journaux), ISO-TP maison testé** | Réutilise l'écosystème DBC ; OVMS (code C++ du module Leaf) et le dépôt DBC communautaire Leaf servent de **référence de lecture** — licence à vérifier avant toute copie |
| Stockage | **Fichiers bruts immuables (vérité) + PostgreSQL/TimescaleDB (canonique)** ; DuckDB/Parquet en export pour l'analyse exploratoire | QuestDB : performant mais niche, ingestion plutôt que relationnel (sessions, profils, événements). InfluxDB : ruptures v1→v2→v3 et changement de langage de requête, risque sur un horizon de plusieurs années. SQLite : suffisant en volume, mais pas d'agrégats continus ni de compression temporelle |
| API | **FastAPI** | |
| Synchronisation | **HTTPS par morceaux, reprise par offset** (§3.6) | MQTT : inadapté aux transferts de gros fichiers ; FTP/SMB : pas d'intégrité ni de reprise propres |
| Interface historique | **PWA React + Vite**, graphiques uPlot (séries longues) | Grafana possible en parallèle pour l'exploration, pas comme produit |
| Affichage en conduite | **Dépend du téléphone (Q1)** : Android → PWA + **Web Bluetooth** (GATT) depuis l'origine HTTPS ; iPhone → coquille native (Capacitor) avec BLE, ou petit **écran dédié** sur le module | Wi-Fi point d'accès du module : téléphone privé d'Internet et blocage `ws://` depuis une page HTTPS (contenu mixte) ; PWA servie par le module en HTTP : pas de *service worker* (pas d'installation ni de hors-ligne) |
| Modèle d'autonomie | Modèle **physique paramétrique** (aéro + roulement + pente + inertie + auxiliaires/chauffage en fonction de T) calibré par **moindres carrés régularisés** sur les trajets ; résidus éventuellement corrigés par un modèle statistique simple | Apprentissage « boîte noire » : trop peu de trajets (dizaines à centaines), non extrapolable à des températures jamais vues |
| Dénivelé | Modèle numérique de terrain (SRTM ou données LiDAR provinciales) interrogé par position | Altitude GNSS : bruit de plusieurs mètres, dérivée inutilisable |

**Construire ou acheter le module** : le module OVMS v3 (ESP32, 3 bus CAN, GNSS, support Leaf existant, y compris câble ZE1 vendu par la communauté) couvre une grande partie des Phases 2-3. Recommandation par défaut : **construire un enregistreur minimal** (moins cher, format brut maîtrisé, passivité matérielle sur l'EV-CAN), **sauf** si l'objectif est le résultat plutôt que l'apprentissage matériel (Q3) — auquel cas acheter OVMS, l'ajouter comme transport `ovms`, et réinvestir le temps dans les Phases 4-5. Le reste du plan ne change pas.

---

## 7. Feuille de route

Hypothèse de charge : **~8-10 h/semaine** (à corriger avec Q6). Durées en semaines calendaires. Les pistes **A (données/logiciel)** et **B (matériel)** avancent en parallèle.

### Phase 0 — Collecter dès cet hiver et vérifier le terrain (semaines 1-2, sans soudure)

Matériel : dongle **OBDLink CX ou MX+** (STN, fiable sur les réponses longues), application **LeafSpy Pro** (journalisation CSV), adaptateur **USB-CAN candleLight/CANable** + séparateur OBD donnant accès à 6/14 **et** 12/13, ordinateur portable.

1. Activer la journalisation LeafSpy Pro sur **tous les trajets** de cet hiver (contenu exact des CSV **[à valider]** : GIDs, SOH, Hx, cellules, températures, GNSS du téléphone). C'est le jeu de données du modèle hivernal v0.
2. Mesurer le brochage (au repos : ~60 Ω entre H et L d'une même paire ; contact mis : H ≈ 2,5-3,5 V, L ≈ 1,5-2,5 V ; oscilloscope si possible).
3. Captures passives (Listen-Only) sur les deux bus : contact mis à l'arrêt, conduite (dont une accélération forte), recharge L2, extinction et 10 min après, et si possible une nuit complète (réveils).
4. Requêtes 21 01/02/04/06 via le STN **sur 6/14** : confirmer que la passerelle route vers le LBC, mesurer la latence et la taille des réponses.
5. Archiver toutes les captures avec un descriptif (scénario, température, SOC).

**Critères de sortie** : brochage confirmé par mesure ; table des identifiants observés avec période mesurée et charge de bus (%) par bus ; 21 01/02/04 obtenus via 6/14 (ou constat contraire documenté → décision de repli §4.2) ; temps de silence du bus après extinction mesuré ; ≥ 5 trajets LeafSpy journalisés.

### Phase 1 — Socle logiciel et première valeur sans matériel (semaines 2-6, piste A)

1. Schéma canonique 1.0 (JSON Schema), format de profil 0.1, profil `nissan/leaf-aze0-30kwh` et profil `generic-obd2`.
2. Décodeur Python : DBC + réassemblage ISO-TP + décodeurs de réponses ; importeurs candump/SavvyCAN **et** CSV LeafSpy (deux sources de nature différente → même sortie canonique : première preuve d'abstraction).
3. Tests « golden » sur les captures de la Phase 0 ; intégration continue.
4. TimescaleDB + ingestion + première vue PWA « historique » (trajets, SOH/Hx/Ah, dispersion cellulaire, températures).

**Critères de sortie** : le rejeu de chaque capture produit des séries canoniques stables (tests verts) ; écart GIDs/SOH/Hx entre notre décodage et LeafSpy = 0 sur les mêmes instants ; tension pack (0x1DB) vs somme des 96 cellules : écart **au repos (|I| < 2 A)** ≤ 1 % et constant d'une capture à l'autre ; ajout d'un signal = modification du profil seul (vérifié par une revue de diff).

**Valeur livrée** : historique consultable de la batterie dès décembre, sans avoir soudé.

### Phase 2 — Enregistreur embarqué passif (semaines 3-12, piste B)

- **2a (banc)** : carte ESP32-C6 + 2 transceivers + SD + GNSS + mesure 12 V ; firmware de capture et journal `EVLOG v1` ; **rejeu des captures de Phase 0 sur banc** (adaptateur USB-CAN en émetteur vers le module) au débit réel ×1,5 ; mesure de veille 48 h sur alimentation de laboratoire à 12,6 V.
- **2b (véhicule)** : installation, synchronisation Wi-Fi, ancrages temporels.

**Critères de sortie** : perte de trames < 0,1 % au banc à 1,5× le débit mesuré (compteur de débordement + trous dans la périodicité de 0x1DB) ; courant de veille ≤ 0,5 mA mesuré sur 48 h ; **2 semaines sans intervention** dans la voiture avec ≥ 95 % des trajets capturés (comparaison avec l'odomètre/LeafSpy) ; 100 % des sessions synchronisées sans action ; **aucun code défaut nouveau** (relevé LeafSpy avant/après) ; horodatage `gnss`/`ntp` sur ≥ 95 % des échantillons.

### Phase 3 — Requêtes de diagnostic (semaines 10-14)

1. ISO-TP (SF/FF/FC/CF, numéros de séquence, délais N_Bs/N_Cr), ordonnanceur déclaratif, liste blanche, règles §5.2.
2. Calendrier du profil Leaf (21 01 au démarrage/arrêt et périodique, 21 02 périodique + rafales sous fort courant, 21 04, 21 06).

**Critères de sortie** : taux de réussite des requêtes ≥ 99 % véhicule `ready` ; latence 21 02 mesurée et journalisée ; aucune émission hors liste blanche (test unitaire + vérification du journal TX) ; aucune émission bus endormi (vérifié sur une nuit) ; **aucun code défaut après 500 km**.

### Phase 4 — Analyses batterie et sentinelle 12 V (semaines 12-18, piste A)

1. Segmentation trajets/recharges ; sessions de recharge (L2 vs CHAdeMO, courbes de puissance).
2. Résistance interne relative par groupe : régression ΔV/ΔI sur **de nombreux événements** (instantanés non simultanés → bruit), normalisée en température ; classement avec intervalle de confiance ; alerte seulement si un groupe sort de la distribution de façon persistante.
3. Sentinelle 12 V : tendance de la tension de repos, alerte ntfy ou équivalent.

**Critères de sortie** : pour chaque groupe, estimation avec IC95 publiée après ≥ 30 événements de forte charge ; reproductibilité : classement des 5 groupes les plus faibles stable entre deux moitiés de l'échantillon ; alerte 12 V testée par simulation (alimentation de labo).

### Phase 5 — Affichage en conduite (semaines 16-20)

Selon Q1 (Web Bluetooth Android, coquille native iOS, ou écran dédié). Quatre indicateurs : énergie restante (BMS + estimation `model` quand disponible, provenance affichée), consommation instantanée traction vs auxiliaires (si `hvac_power_w` disponible, sinon masqué), écart cellulaire sous charge, marge à destination (à partir de la Phase 6).

**Critères de sortie** : rafraîchissement ≥ 2 Hz, reconnexion automatique < 5 s après démarrage, lisible en un coup d'œil (test sur 10 trajets), aucune interaction requise en roulant.

### Phase 6 — Modèle d'autonomie hivernale (v0 cet hiver, v1 hiver suivant)

1. **v0 (dès janvier)** sur les données LeafSpy + enregistreur : modèle physique paramétrique, chauffage en fonction de la température extérieure et de la durée (le préchauffage de l'habitacle coûte surtout en début de trajet).
2. **v1 (automne suivant)** : recalibration sur un hiver complet, calculateur de marge et vitesse conseillée.
3. Énergie utilisable : établir la réserve à partir des données communautaires et d'arrêts volontaires bien au-dessus du mode tortue — **ne pas rouler en hiver jusqu'à la réserve pour calibrer**.

**Critères de sortie (v1)** : sur ≥ 20 trajets d'hiver non vus pendant la calibration, erreur absolue médiane de l'énergie consommée prédite au départ ≤ 8 %, P90 ≤ 15 %, et meilleure que la base de référence « consommation moyenne des 30 derniers jours » ; erreur ≤ 5 % sur l'énergie restante à l'arrivée quand la prédiction est refaite à mi-trajet.

### Phase 7 — Preuve d'agnosticisme et ZE1

1. Deuxième famille de véhicule en **N1** (idéalement Hyundai/Kia ou GM : requêtes 0x22, autre topologie) — avec une vraie voiture si possible (Q5), sinon depuis des captures publiques.
2. Profil ZE1 (câble de dérivation passerelle + requêtes via OBD).

**Critères de sortie** : le deuxième véhicule produit tension, courant, SOC et (si disponible) cellules dans la PWA **sans aucune modification** du firmware, du cœur backend ni de la PWA (diff limité à `profiles/`) ; les tests golden des deux profils passent dans la même CI.

### Calendrier indicatif

| Période | Jalons |
|---------|--------|
| Déc. 2026 | Phase 0 terminée, collecte LeafSpy en cours ; Phase 1 avancée (historique consultable) |
| Janv.-fév. 2027 | Enregistreur embarqué dans la voiture (Phase 2b), modèle v0 |
| Mars-mai 2027 | Requêtes diagnostic, analyses cellulaires, sentinelle 12 V |
| Été 2027 | Affichage en conduite, deuxième véhicule, ZE1 |
| Automne 2027 | Modèle hivernal v1 calibré, prêt pour l'hiver 2027-2028 |

---

## 8. Risques

| Risque | Impact | Mitigation | Critère de validation |
|--------|--------|------------|-----------------------|
| Émission involontaire sur un bus (bug, mauvais débit, bus endormi) | Codes défaut, comportement imprévu du véhicule, batterie 12 V vidée | Passivité matérielle EV-CAN, liste blanche compilée, vérification du débit, pas d'émission bus endormi (§5.2) | Zéro code défaut après 500 km ; journal TX conforme à la liste blanche |
| Décharge de la batterie 12 V | Panne au démarrage par grand froid | Régulateur à faible courant de repos, machine à états §5.3, `PRESERVE` | Veille ≤ 0,5 mA sur 48 h ; nombre de réveils nocturnes mesuré et budgété |
| Passerelle ne routant pas le LBC sur 6/14 | Pas de cellules sans émettre sur l'EV-CAN | Vérifié en Phase 0 avant tout choix matériel | Réponse 21 02 obtenue via 6/14 |
| ZE1 : port OBD isolé | Pas d'écoute passive au port | Câble de dérivation, profil distinct | Phase 7 |
| Perte de trames (FIFO TWAI, accès flash, Wi-Fi) | Trous dans les données, calculs faussés | ISR en IRAM, Wi-Fi coupé en roulant, compteur de pertes journalisé | < 0,1 % au banc à 1,5× |
| Corruption SD / usure | Perte de trajets | Industrielle pSLC, pré-allocation, CRC par bloc, effacement seulement après accusé serveur | Coupures d'alimentation répétées au banc (100 cycles) sans perte au-delà du dernier bloc |
| Températures extrêmes | Plantage, SD hors spécification | Composants -40/+85 °C, pas de PSRAM octale sans ECC, boîtier ASA | Fonctionnement après nuits ≤ -20 °C et habitacle chaud |
| Horodatage faux | Corrélations faussées (météo, recharge) | Quartz 32 kHz, ancrages GNSS/NTP, `ts_quality` | ≥ 95 % d'échantillons ancrés |
| Facteurs d'échelle/signes DBC faux | Analyses fausses en silence | Contrôles croisés (pack vs Σ cellules, GIDs vs LeafSpy, V×I vs couple×vitesse) dans les tests | Tests golden |
| Mise à jour logicielle du véhicule change les trames | Profil cassé | Empreinte de profil dans chaque journal, alertes de plausibilité au décodage, brut conservé → redécodage | — |
| Sur-ambition / planning | Abandon | Valeur dès la Phase 1, pistes parallèles, critères de sortie | Revue à chaque fin de phase |
| Validation sans véhicule (autres marques) | Profils non testés | Niveaux N0/N1/N2 honnêtes ; « non validé » affiché dans la PWA pour un profil sans capture réelle | — |
| Fuite de données personnelles | Déplacements, VIN | Pas de brut dans un dépôt public, anonymisation des captures | Revue avant commit |
| Licences (DBC communautaires, code OVMS) | Impossible de publier | Vérifier les licences avant d'intégrer, sinon réécrire depuis captures | — |

---

## 9. Arborescence cible du dépôt

```
firmware/          ESP-IDF : capture, EVLOG, ISO-TP, ordonnanceur, énergie, sync
backend/           FastAPI, ingestion, décodeur, analyses, modèles
  evdash/decoder/  moteur générique (profils, DBC, ISO-TP)
pwa/               React + Vite
profiles/
  generic/obd2/
  nissan/leaf-aze0-30kwh/   profile.yaml, *.dbc, decoders.py, captures/ (anonymisées)
schema/            canonical-1.0.0.json, evlog-v1.md
tools/             rejeu de captures sur banc, anonymisation, générateur de tableau firmware
docs/              ce plan, brochages mesurés, résultats de Phase 0
```

---

## 10. Questions ouvertes (seulement celles qui changent le plan)

1. **Téléphone en voiture : iPhone ou Android** (ou tablette/écran dédié acceptable) ? → choix du transport d'affichage en direct (Web Bluetooth impossible sur iOS).
2. **Ta Leaf exacte** : année, finition (S/SV/SL), 24 ou 30 kWh, thermopompe ou chauffage résistif ? → profil, nombre de sondes, modèle de chauffage.
3. **Objectif : apprendre à construire le matériel, ou obtenir le résultat ?** → construire l'enregistreur (Phase 2) ou acheter un OVMS v3 et sauter une bonne partie des Phases 2-3.
4. **Le Wi-Fi domestique atteint-il l'emplacement de stationnement, et quel serveur tourne en permanence à la maison ?** → synchronisation automatique et hébergement de TimescaleDB.
5. **As-tu accès à un deuxième véhicule électrique (lequel) et/ou à une ZE1 ?** → contenu et date de la Phase 7.
6. **Combien d'heures par semaine** peux-tu y consacrer ? → calendrier.
7. **Le dépôt GitHub est-il public ?** → politique de captures, VIN et traces GNSS.
8. **OK pour acheter LeafSpy Pro + un dongle OBDLink et un adaptateur USB-CAN maintenant** (~150-200 $) pour ne pas perdre cet hiver ?

---

## Sources consultées

- Documentation OVMS, module Nissan Leaf/e-NV200 (bus utilisés, passerelle ZE1, paramètres GIDs/Ah, 80 Wh/GID) : https://docs.openvehicles.com/en/latest/components/vehicle_nissanleaf/docs/index.html
- Documentation « leaf-obd », connecteur de diagnostic (EV-CAN 12/13, Car-CAN 6/14, AV-CAN 3/11) : https://leaf-obd.readthedocs.io/en/latest/diagnostic-connector.html
- Fil OVMS ZE1 (câble de dérivation) : https://community.openenergymonitor.org/t/ovms-ze1-40kwh-64kwh-2018-nissan-leaf-can-tap-cable-available/29798?page=2
- Forums sur les sondes thermiques 30 kWh : https://www.boards.ie/discussion/2057594661/battery-heat-meter-and-30kw-leaf , https://mynissanleaf.com/threads/where-are-the-temperature-sensors-located-within-the-battery-pack.25451/
- ESP-IDF TWAI sur ESP32-C6 (deux contrôleurs, IDF ≥ 5.2) : https://docs.espressif.com/projects/esp-idf/en/stable/esp32c6/api-reference/peripherals/twai.html
- Fiche ESP32-S3-WROOM-1 (N16R8 limité à 65 °C sans ECC) : https://www.espressif.com/sites/default/files/documentation/esp32-s3-wroom-1_wroom-1u_datasheet_en.pdf
