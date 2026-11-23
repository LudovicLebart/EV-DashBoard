# Données et vie privée

Pourquoi ces données sont sensibles et comment on les traite.

## Ce qui est sensible

- **Traces GPS** : elles décrivent des déplacements.
- **VIN** (numéro de série du véhicule) : il apparaît dans certaines réponses de diagnostic et identifie une voiture.
- **Journaux bruts complets** : ils contiennent tout ce qui précède.

## Règles

1. **Le dépôt GitHub doit être privé** (préférence de l'utilisateur ; **à vérifier dans les réglages GitHub**), et l'être **dès sa création** : rendre privé un dépôt déjà public n'efface pas ce qui a pu être copié.
2. Les **journaux bruts complets et les traces GPS réelles vivent sur le serveur**, pas dans Git (ce sont des données, pas du code).
3. Les captures de test versionnées (`profiles/*/captures/`) sont **anonymisées** : VIN masqué, GPS retiré. Cela permet d'ouvrir le dépôt un jour sans réécrire l'historique.
4. Les jetons des modules et les identifiants de base de données ne sont jamais versionnés.
5. Le projet partage le serveur avec un autre projet : base, rôle et conteneur distincts (voir le plan, §6.1).

## Licences

Les DBC communautaires et le code OVMS ont leurs propres licences : les **vérifier avant** de les copier dans le dépôt ; sinon, réécrire à partir de captures.
