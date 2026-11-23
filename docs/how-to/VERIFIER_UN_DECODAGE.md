# Comment vérifier qu'un décodage est juste

Une erreur de signe ou de facteur d'échelle ne produit aucune alerte : il faut des **contrôles croisés**.

## Contrôles à faire sur toute capture

| Contrôle | Attendu |
|----------|---------|
| Tension du pack vs somme des 96 tensions de cellules, **au repos** (courant en valeur absolue < 2 A) | Écart ≤ 1 %, constant d'une capture à l'autre |
| Signe du courant | Positif en décharge, négatif en recharge/régénération (convention du schéma canonique) |
| Puissance (V × I) pendant une accélération | Cohérente avec la vitesse et le couple |
| SOC / tension | Concordance avec le tableau de bord de la voiture sur les mêmes instants |
| Périodicité des trames périodiques | Trous = pertes de trames |
| Plage des valeurs | Dans les bornes physiques (température, tension de cellule) |

## Pas à pas

1. Choisir un instant au repos dans le journal.
2. Lire la tension du pack et la somme des tensions de cellules ; calculer l'écart.
3. Repérer une accélération : le courant doit monter dans le sens « décharge ».
4. Comparer à l'affichage de la voiture noté sur la fiche.
5. En cas d'écart : vérifier d'abord le **signe**, puis le **facteur d'échelle**, puis l'**indice de position** (index de cellule décalé).

## Quand ça ne colle pas

Noter la valeur, la trame brute et l'instant dans la fiche ; corriger le profil (pas le code) ; rejouer les captures de test (tests « golden »). Voir [`AJOUTER_UN_PROFIL_VEHICULE.md`](AJOUTER_UN_PROFIL_VEHICULE.md).
