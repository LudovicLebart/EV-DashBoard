# TODO

Source : [`PLAN.md`](PLAN.md). Mettre à jour après chaque étape validée.

## Phase 0 — Essai routier : Spark EV puis Leaf (~30-50 $)

- [ ] Acheter le CANable 2.0 (~19-36 $) et un câble/breakout OBD-II mâle 16 broches à bornier (~10-15 $)
- [ ] Préparer l'ordinateur Linux (`can-utils`, noyau ≥ 5.10 pour `can-isotp`, heure synchronisée, veille désactivée) ; identifier le micrologiciel du CANable et le **reflasher en candleLight** (écoute seule) ; vérifier la résistance de terminaison de la carte
- [ ] Répéter à blanc à la maison le script `lire_isotp.py` (tutoriel, étape 4) et `candump`
- [ ] Choisir une appli de trace GPS Android (export GPX)
- [ ] **Premier essai sur la Spark EV** : confirmer l'**année** (2014 A123 ou 2015-2016 LG Chem, profils différents) et la durée d'accès
- [ ] Récupérer la liste communautaire de PID de la **Bolt EV** (Torque Pro) et tester sur la Spark, PID par PID (en-tête BECM 0x7E7 à confirmer)
- [ ] Obtenir ensuite l'accès à une Leaf (2016-2018) pour un essai
- [ ] Mesurer le brochage ; capturer en écoute seule ; trajet avec forte accélération
- [ ] Requêtes `21 01/02/04` via 6/14 ; noter si la passerelle les route
- [ ] Décoder hors ligne et vérifier (somme des 96 tensions, signe du courant)
- [ ] Second essai sur 12/13 en écoute seule confirmée (candleLight ; Leaf 2016-2017)

## Phase 1 — Socle logiciel

- [ ] Vérifier dans les réglages GitHub que le dépôt est **privé** ; créer l'arborescence (`firmware/`, `backend/`, `pwa/`, `profiles/`, `schema/`, `tools/`)
- [ ] Schéma canonique 1.0 (JSON Schema)
- [ ] Format de profil 0.1, profil `chevrolet/spark-ev` (banc), profil Leaf (brouillon), profil `generic-obd2`
- [ ] Décodeur Python et tests « golden »

## Questions ouvertes

- [ ] Leaf précise (année, kWh, chauffage) pour chaque essai
- [ ] Wi-Fi atteignant la place de stationnement
- [ ] Serveur Guitar Hunter : version Postgres, place disque, conteneur possible
- [ ] Année de la Spark ; accès aux Tesla Model 3/Y
- [ ] Heures par semaine (après la Phase 0)

## Planificateur (Phase 6b)

- [ ] Fixer x (durée d'arrêt), marge d'arrivée et régions de voyage (question ouverte 8)
- [ ] Choisir le moteur d'itinéraire (OSRM / Valhalla / GraphHopper) et la source de bornes ; vérifier couverture, licence, fraîcheur
- [ ] Trouver des courbes de charge publiques prudentes (Leaf, Spark) pour la v0

## Fonctionnalités additionnelles (priorité haute)

- [ ] Rapport de santé batterie v0 (PDF) à partir d'une capture de Phase 0
- [ ] Coût de recharge : choisir où saisir les tarifs (domicile, réseaux publics)
- [ ] Perte à l'arrêt : définir la méthode entre deux sessions
- [ ] Alertes ntfy : sujet non devinable ou serveur auto-hébergé ; aucun GPS/VIN dans les messages
- [ ] Catalogue complet : [`../reference/FONCTIONNALITES.md`](../reference/FONCTIONNALITES.md)

## Option commandes à distance (Phase 8, plus tard)

- [x] Voie décidée : API officielle uniquement, jamais le bus CAN (2026-11-23)
- [ ] Décider quelles commandes ; confirmer qu'on n'a aucune commande à distance sur Leaf/Spark tant qu'aucune API officielle n'existe
- [ ] Vérifier sur le tableau de bord développeur Tesla : coût, limites, signature des commandes (question ouverte 9)

## Plus tard

- Boîtier ESP32-C6 (Phase 2), requêtes embarquées (Phase 3), analyses batterie et sentinelle 12 V (Phase 4), affichage Android PWA + Web Bluetooth (Phase 5), modèle d'autonomie hivernale (Phase 6), **planificateur de trajets avec arrêts de recharge de x minutes (Phase 6b ; v0 à hypothèses explicites dès la Phase 1)**, preuve d'agnosticisme et ZE1 (Phase 7)
