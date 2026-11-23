# Protocole d'essai avant achat (batterie)

Une page à suivre chez un vendeur. **Statut : non encore exécuté** ; à répéter d'abord à la maison sur la Spark. Contexte : [`../management/PLAN.md`](../management/PLAN.md), Phase 0. Commandes détaillées : [`../tutorials/PREMIER_ESSAI_ROUTIER.md`](../tutorials/PREMIER_ESSAI_ROUTIER.md).

## Avant d'y aller

- [ ] Ordinateur chargé, Linux démarré, CANable reflashé en candleLight **[à valider]**, câble OBD-II, script de requêtes prêt.
- [ ] Appli de trace GPS installée sur le téléphone.
- [ ] Avoir répété le protocole à la maison (branchement en moins de 10-15 minutes).
- [ ] Fiche d'essai imprimée ou ouverte (voir ci-dessous).

## Demander la permission

- [ ] Expliquer au vendeur : branchement **en lecture seule** sur la prise OBD-II, aucune modification, aucun effacement de codes défaut, retrait à la fin.
- [ ] Accord du vendeur **avant** de brancher. Sans accord : ne pas brancher.
- [ ] Relever l'état du tableau de bord : **voyants, codes défaut visibles, SOC, odomètre, barres de capacité**, avant le branchement (photo).

## Branchement et écoute (voiture à l'arrêt)

1. Câblage : 6 → CANH, 14 → CANL, 5 → masse. Terminaison de l'adaptateur désactivée.
2. Contact mis **sans démarrer**, écoute seule : vérifier que des trames passent, sans erreur.
3. Requêtes de lecture **à l'arrêt** (21 01, 21 02, 21 04 sur une Leaf) : une à la fois, ≤ 5 par seconde. **Aucune réponse** : noter, arrêter, ne pas insister.

## Essai (~15 minutes)

1. Lancer la trace GPS et le journal brut ; noter l'heure.
2. Conduite normale, puis **une forte accélération** en sécurité, puis stabilisé.
3. Arrêt : **couper les requêtes avant** de couper le contact, laisser enregistrer 2 minutes, arrêter le journal.

## Remise en état

- [ ] Débrancher, remettre l'habitacle comme trouvé.
- [ ] **Comparer avec l'état d'avant** : aucun voyant ni code défaut nouveau.
- [ ] Noter température extérieure, SOC final, odomètre.

## Fiche d'essai (à remplir)

| Champ | Valeur |
|-------|--------|
| Date, lieu | |
| Modèle, année, kWh, VIN (masqué dans les copies) | |
| Odomètre | |
| Température extérieure | |
| SOC au départ / à l'arrivée | |
| Voyants avant / après | |
| Réponses obtenues (21 01, 21 02, 21 04) | |
| Anomalies | |

## Ce qu'on ne fait pas

- Pas d'émission sur 12/13, pas d'écriture, pas d'effacement de défauts, pas de service de session ou de sécurité (voir [`../reference/SECURITE_BUS.md`](../reference/SECURITE_BUS.md)).
- Pas de conclusion d'achat tirée du seul rapport : il ne remplace ni l'historique du véhicule ni une inspection mécanique.
