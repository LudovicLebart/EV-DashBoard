# Référence — Rapport d'inspection de la batterie (v0)

Spécification du PDF généré à partir d'**un seul essai** (Phase 0). **Statut : spécification, générateur non écrit.** Les valeurs de référence sont à sourcer ; aucune ne doit être affichée sans source **[à valider]**.

## Principes

- **Contexte d'abord** : un SOH sans température, niveau de charge et recharge récente est trompeur.
- **Dire ce qui manque** : chaque section indique ce qui a été lu, ce qui n'a pas répondu et ce qui est estimé.
- **Pas de verdict d'achat**, pas de garantie : le rapport informe.
- **Vie privée** : VIN masqué dans les copies partagées ; GPS jamais dans le rapport.

## Sections

| # | Section | Contenu | Source de données |
|---|---------|---------|-------------------|
| 1 | Identité et conditions | Modèle, année, kWh, odomètre, date, température extérieure, SOC début/fin, durée de l'essai, version du profil | Fiche d'essai + signaux |
| 2 | Santé | SOH, Hx, capacité (Ah), énergie restante (GIDs × Wh/GID), comparaison à la capacité neuve du profil | Requête `21 01` (Leaf) **[à valider]** |
| 3 | Équilibre des cellules | 96 tensions, écart max-min, groupes les plus bas, histogramme | Requête `21 02` |
| 4 | Sous charge | Tension et courant pendant l'accélération forte, chute de tension par groupe, groupes qui décrochent | `21 02` en rafales ; flux rapide 12/13 si enregistré **[à valider]** |
| 5 | Températures | Sondes du pack, écart, évolution pendant l'essai | `21 04` |
| 6 | Recharge (si observée) | Puissance, courbe pendant l'essai, type de borne | Journal |
| 7 | Fiabilité du rapport | Requêtes réussies/échouées, trames perdues, périodes sans donnée, plausibilité (somme des tensions vs tension pack au repos ≤ 1 %) | Journal |
| 8 | Lecture et limites | Ce que le rapport ne dit pas ; comparaison à une fourchette de référence **si la source est citée** | Texte fixe |

## Ce qui dépend de la voiture

- **Leaf AZE0 (2013-2017)** : toutes les sections possibles ; la section 4 est plus riche avec le flux rapide (second adaptateur).
- **Leaf ZE1 (2018)** : sections 2, 3, 5 via les requêtes sur 6/14 ; flux rapide indisponible (passerelle) ; section 4 limitée.
- **Spark EV** : sections selon les identifiants de données lus ; le rapport s'adapte aux capacités déclarées par le profil.
