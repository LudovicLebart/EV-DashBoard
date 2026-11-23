# Architecture et choix : pourquoi ce découpage

Explication des décisions. Les détails d'exécution sont dans [`../management/PLAN.md`](../management/PLAN.md).

## 1. Une voiture = un profil

Principe directeur : **le code ne connaît aucune voiture ; un profil la décrit.**

```
Véhicule ──► Module/adaptateur ──► Journal brut ──► Décodeur (profil) ──► Base ──► Interface
              (enregistre)          (vérité)         (DBC + YAML)        (canonique)   (PWA)
```

- Le **module** ne fait qu'enregistrer des trames et envoyer des requêtes autorisées.
- Le **décodage** se fait sur le serveur, à partir d'un **profil** : trames à lire, requêtes à envoyer, facteurs d'échelle, capacités.
- Le **schéma canonique** (tension, courant, SOC…) est la seule chose que l'interface connaît.

Intérêt : brancher une autre voiture = écrire un profil ; corriger une erreur de décodage = refaire tourner le décodeur sur les journaux bruts, sans reflasher le module.

## 2. L'agnosticisme est honnête, pas « 80 % »

Trois niveaux de support :

| Niveau | Ce que l'on obtient |
|--------|---------------------|
| **N0 générique** | Enregistrement brut + PID OBD-II standard si la voiture répond |
| **N1 communautaire** | Tension, courant, SOC, éventuellement cellules, depuis des listes publiques |
| **N2 référence** | Pipeline complet, modèle d'autonomie calibré |

Le **code** peut être agnostique ; la **valeur analytique** ne l'est pas (SOH, réserve, modèle d'autonomie, chauffage…). Un nouveau véhicule arrive vite en N1, en N2 après une saison de données.

## 3. Pourquoi deux « canaux » CAN séparés

Un contrôleur en mode « écoute seule » ne peut ni émettre ni acquitter : « zéro écriture » et « requêtes UDS » étaient incompatibles sur le même contrôleur. D'où :

- un canal **passif** (bus rapide de la Leaf, broches 12/13), dont le matériel ne peut physiquement pas émettre ;
- un canal **diagnostic** (broches 6/14), seul autorisé à envoyer des requêtes, via une **liste blanche** compilée dans le firmware.

## 4. Pourquoi enregistrer en brut

Les trames brutes sont **immuables** et servent de vérité. La base décodée se reconstruit. Cela protège contre : un facteur d'échelle faux, un nouveau signal découvert plus tard, une mise à jour du véhicule qui change les trames.

## 5. Choix technologiques en bref

| Domaine | Choix | Raison courte |
|---------|-------|---------------|
| Phase 0 | CANable + ordinateur Linux + GPS du téléphone | Aucun développement d'appli, matériel réutilisé ensuite |
| Boîtier (plus tard) | ESP32-C6 | Deux contrôleurs CAN natifs, Wi-Fi + BLE |
| Firmware | ESP-IDF (C) | Maîtrise des interruptions, multi-contrôleur |
| Décodage | Python (`cantools`, `python-can`) | Écosystème DBC |
| Base | PostgreSQL (+ TimescaleDB si possible) | Maturité, SQL, conservation du brut à part |
| Interface | PWA React + Web Bluetooth (Android) | Pas d'appli native nécessaire |
| Modèle d'autonomie | Physique, paramétrique | Peu de trajets, extrapolation au froid |

Alternatives écartées et raisons : voir [`../management/PLAN.md`](../management/PLAN.md) §6.

## 6. Construire plutôt qu'acheter

Un module OVMS v3 ferait une grande partie du travail, mais son prix actuel est introuvable (150 $ à son lancement en 2018), et LeafSpy ne sert que pour la Leaf. L'utilisateur a choisi de **construire**, sans acheter de matériel qui ne servirait qu'à la Phase 0.

## 7. Matériel par étapes (règle : rien de jetable)

1. **Phase 0** : CANable + câble OBD-II (~30-50 $), ordinateur et téléphone déjà possédés.
2. **Plus tard** : boîtier ESP32 (~80-110 $ le banc ; ~150-250 $ la version voiture), seulement si l'on veut un enregistreur permanent.

Prix et sources : [`../management/PLAN.md`](../management/PLAN.md) §5.5.
