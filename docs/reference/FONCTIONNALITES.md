# Référence — Catalogue des fonctionnalités

Liste de toutes les fonctions retenues, avec origine et phase. Source de vérité : [`../management/PLAN.md`](../management/PLAN.md). Origine : **concurrence** = existe dans des outils connus (LeafSpy, TeslaMate, A Better Routeplanner, EVNotify, OVMS) **[non revérifié fonction par fonction]** ; **idée** = proposition de ce projet.

## Socle (déjà au plan)

| Fonction | Phase |
|----------|-------|
| Télémétrie autonome des trajets et recharges | 0-2 |
| Santé des cellules (écart, résistance interne relative) | 4 |
| Autonomie hivernale | 6 |
| Vieillissement long terme (SOH, Hx, Ah) | 4 |
| Sentinelle 12 V | 4 |
| Planificateur de trajets avec arrêts de recharge de x minutes | 6b |

## Ajoutées le 2026-11-23

| # | Fonction | Origine | Phase | Priorité |
|---|----------|---------|-------|----------|
| 1 | Rapport de santé batterie (PDF) : inspection avant achat, dossier de garantie | Concurrence (LeafSpy, rapports de revente) | v0 en 1, v1 en 4 | Haute |
| 2 | Coût de recharge (domicile, public, au km, tarifs saisis) | Concurrence (TeslaMate) | 4 | Haute |
| 3 | Perte à l'arrêt (énergie perdue voiture garée) | Concurrence (TeslaMate) | 2-4 | Haute |
| 4 | Alertes ntfy (12 V, recharge terminée, dérive de cellule, charge lente) | Concurrence (EVNotify) | 4 | Haute |
| 5 | Décomposition de l'énergie (traction, chauffage, auxiliaires, régénération) | Concurrence (TeslaMate, ABRP) | 6 | Moyenne |
| 6 | Courbes de charge par température et type de borne | Idée | 4, 6b | Moyenne |
| 7 | Détection d'anomalies (cellule, charge lente, écart pack/cellules) | Idée | 4 | Moyenne |
| 8 | Projection du vieillissement avec fourchette | Idée | 4+ | Moyenne |
| 9 | Météo le long de la route (température, vent) | Concurrence (ABRP) | 6b | Moyenne |
| 10 | Conseil de préchauffage branché | Idée | 6 | Moyenne |
| 11 | Fiabilité des bornes (journal personnel) | Idée | 6b | Moyenne |
| 12 | Apprentissage des trajets habituels | Idée | 6b+ | Moyenne |
| 13 | Note de conduite après trajet | Idée | 6 | Moyenne |
| 14 | Comparaison entre véhicules sur un même trajet | Idée | 7 | Moyenne |

## Options soumises à décision explicite

| # | Fonction | Statut |
|---|----------|--------|
| 15 | Lecture des codes défaut | Lecture seule mais **hors de la liste blanche** : décision écrite requise pour élargir (voir [`SECURITE_BUS.md`](SECURITE_BUS.md)) |
| 16 | **Commandes à distance** (préchauffage, recharge…) | **Phase 8, option à débloquer**, désactivée par défaut ; **API officielle uniquement, jamais le bus CAN** (décision du 2026-11-23) ; **Tesla** seule voie officielle trouvée ; **Leaf et Spark : aucune API officielle trouvée, donc non prises en charge** |
| 17 | Partage de données entre utilisateurs | Retenue, non planifiée ; cadre de vie privée requis d'abord |

## Notes de conception

- **Rapport de santé** : y indiquer les conditions de mesure (température, SOC, recharge récente) ; un SOH sans contexte est trompeur. Ne jamais présenter comme une garantie.
- **Perte à l'arrêt** : calculée entre deux sessions, sans enregistrement continu ; la résolution du SOC limite la précision **[à valider]**.
- **Alertes ntfy** : sujet non devinable ou serveur auto-hébergé ; aucune donnée personnelle (GPS, VIN) dans les messages **[à valider]**.
- **Commandes à distance** : voir plan, Phase 8 ; règles de sécurité dédiées dans [`SECURITE_BUS.md`](SECURITE_BUS.md).
