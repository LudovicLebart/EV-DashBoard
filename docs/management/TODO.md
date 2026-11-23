# TODO

Source : [`PLAN.md`](PLAN.md). Mettre à jour après chaque étape validée.

## Phase 0 — Essai routier Leaf (~30-50 $)

- [ ] Acheter le CANable 2.0 (~20-36 $) et un câble/breakout OBD-II mâle 16 broches (~10-15 $)
- [ ] Préparer l'ordinateur Linux (`can-utils`, noyau `can-isotp`) ; vérifier le micrologiciel du CANable (candleLight ou slcan)
- [ ] Choisir une appli de trace GPS Android
- [ ] Obtenir l'accès à une Leaf (2016-2018) pour un essai
- [ ] Mesurer le brochage ; capturer en écoute seule ; trajet avec forte accélération
- [ ] Requêtes `21 01/02/04` via 6/14 ; noter si la passerelle les route
- [ ] Décoder hors ligne et vérifier (somme des 96 tensions, signe du courant)
- [ ] Second essai sur 12/13 en écoute seule (Leaf 2016-2017)

## Phase 1 — Socle logiciel

- [ ] Créer le dépôt de code **privé** (déjà privé) et l'arborescence (`firmware/`, `backend/`, `pwa/`, `profiles/`, `schema/`, `tools/`)
- [ ] Schéma canonique 1.0 (JSON Schema)
- [ ] Format de profil 0.1, profil Leaf (brouillon), profil `generic-obd2`
- [ ] Décodeur Python et tests « golden »

## Questions ouvertes

- [ ] Leaf précise (année, kWh, chauffage) pour chaque essai
- [ ] Wi-Fi atteignant la place de stationnement
- [ ] Serveur Guitar Hunter : version Postgres, place disque, conteneur possible
- [ ] Année de la Spark ; accès aux Tesla Model 3/Y
- [ ] Heures par semaine (après la Phase 0)

## Plus tard

- Boîtier ESP32-C6 (Phase 2), requêtes embarquées (Phase 3), analyses batterie et sentinelle 12 V (Phase 4), affichage Android PWA + Web Bluetooth (Phase 5), modèle d'autonomie hivernale (Phase 6), preuve d'agnosticisme et ZE1 (Phase 7)
