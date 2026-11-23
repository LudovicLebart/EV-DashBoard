# Comment ajouter un profil véhicule

Objectif : brancher une nouvelle voiture **sans modifier le firmware, le cœur du backend ni l'interface**. Le diff d'un ajout de véhicule doit se limiter à `profiles/`.

Contexte : [`../explanation/ARCHITECTURE_ET_CHOIX.md`](../explanation/ARCHITECTURE_ET_CHOIX.md). Format détaillé : [`../management/PLAN.md`](../management/PLAN.md) §4.

## 1. Créer le dossier

```
profiles/<marque>/<modèle>/
  profile.yaml
  *.dbc               (trames passives)
  decoders.py         (optionnel, greffon serveur seulement)
  captures/           (journaux de référence ANONYMISÉS + valeurs attendues)
```

## 2. Remplir `profile.yaml`

Sections minimales : métadonnées du véhicule (marque, plateforme, années, topologie de la batterie, nombre de sondes), **bus** utilisés (broches, débit, mode), correspondances **DBC → schéma canonique** (facteur, décalage, signe), **requêtes** (adresses, données, calendrier, décodeur), **liste blanche d'émission**, capacités déclarées, paramètres de calibration.

Règles :

- Toute valeur incertaine est suivie de `# A VALIDER` et n'est pas utilisée en production avant test sur capture réelle.
- Transformations autorisées dans le YAML : facteur, décalage, signe, table de correspondance, bornes. Le reste va dans `decoders.py` avec tests.
- Un signal absent du véhicule reste **absent** (jamais 0).
- Une trame inconnue n'entre pas dans la liste blanche d'émission : le profil ne peut pas l'élargir sans décision explicite (voir [`../reference/SECURITE_BUS.md`](../reference/SECURITE_BUS.md)).

## 3. Capturer et vérifier

1. Faire les captures selon [`../tutorials/PREMIER_ESSAI_ROUTIER.md`](../tutorials/PREMIER_ESSAI_ROUTIER.md).
2. Contrôler le décodage : [`VERIFIER_UN_DECODAGE.md`](VERIFIER_UN_DECODAGE.md).
3. Ajouter les captures anonymisées et leurs **valeurs attendues** (tests « golden »).

## 4. Déclarer le niveau de support

- **N0** : rien d'autre à faire (profil générique).
- **N1** : tension, courant, SOC, éventuellement cellules ; marquer le profil « non validé » tant qu'il n'a pas de capture réelle.
- **N2** : calibration du modèle d'autonomie sur les trajets du véhicule.

## 5. Contrôle final

Le diff ne touche que `profiles/` ; les tests des **deux** profils (le nouveau et la Leaf) passent dans la même CI.
