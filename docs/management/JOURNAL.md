# Journal des décisions

Entrées récentes en haut. Format : `[DATE] Décision → Conséquence`.

- [2026-11-23] Fonctionnalités additionnelles retenues : rapport de santé batterie, coût de recharge, perte à l'arrêt, alertes ntfy (priorité haute) + 13 autres conservées ; **commandes à distance** = option à débloquer (Phase 8), désactivée par défaut, exception encadrée à la doctrine lecture seule (plan §7.1, `reference/FONCTIONNALITES.md`).
- [2026-11-23] Fonction importante ajoutée : **planification de trajets avec arrêts de recharge de x minutes**, courts et sûrs, comme une Tesla → Phase 6b (conseil seulement, marges P90, plan B, v0 à hypothèses explicites).
- [2026-11-23] Revue de la documentation → corrections : tutoriel réécrit Spark d'abord (écoute passive → requêtes à l'arrêt → trajet ; dépannage ajouté) ; CANable : candleLight recommandé (vraie écoute seule), slcan = acquittements donc 6/14 seulement ; `candump -H` retiré (horodatage matériel, pas « absolu ») ; requêtes par une seule socket ISO-TP (script Python) au lieu de `isotpsend` seul, interface repassée hors listen-only ; `can-isotp` confirmé (Linux ≥ 5.10) ; tensions CAN et masse/terminaison précisées ; faits Spark EV sourcés (2014 A123 ~21 kWh, 2015-2016 LG 19 kWh / 192 cellules, pas de module OVMS, PID de la Bolt, en-tête 7E7 non vérifié) ; contradictions Spark/Leaf, Phase 6/7, LeafSpy/STN, prix CANable et calendrier corrigées ; PDF régénéré.
- [2026-11-23] Première voiture d'essai : **Chevrolet Spark EV** → Phase 0 d'abord sur la Spark, puis Leaf ; PDF de synthèse (plan, Phase 0, matériel) ajouté.
- [2026-11-23] Ordinateur portable en double démarrage Windows + Linux → **Linux** pour la Phase 0 (requêtes batterie gérées par le noyau), Windows en secours.
- [2026-11-23] Téléphone **Android** → PWA + Web Bluetooth ; variantes d'usage du téléphone décrites (plan §6.2).
- [2026-11-23] Premier besoin : remplacer LeafSpy par un essai routier ; GPS du téléphone accepté ; pas de matériel qui ne servirait qu'à la Phase 0 → CANable + câble OBD + ordinateur (~30-50 $) ; ESP32, SD, GNSS reportés.
- [2026-11-23] Leaf visées : toutes les Leaf 2016-2018 (essais) ; pas de Leaf en propre.
- [2026-11-23] Achat de LeafSpy, OBDLink ou OVMS refusé (coût, spécifique Leaf) → on construit.
- [2026-11-23] Serveur de Guitar Hunter retenu pour l'hébergement, avec isolation (plan §6.1).
- [2026-11-23] Dépôt **privé** de préférence.
- [2026-11-23] Révision du plan (révision 2) : deux canaux CAN, enregistrement brut, profils, schéma canonique versionné ; critique de l'ébauche initiale.
