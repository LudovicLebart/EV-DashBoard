# Documentation EV Dashboard

Organisation **Diataxis** : on cherche selon ce qu'on veut faire, pas selon le sujet.

| Je veux… | Section | Contenu |
|----------|---------|---------|
| **Apprendre** en le faisant | [`tutorials/`](tutorials/) | Pas à pas guidés pour un premier résultat |
| **Faire** une tâche précise | [`how-to/`](how-to/) | Recettes courtes, orientées but |
| **Consulter** un fait exact | [`reference/`](reference/) | Schéma de données, règles de sécurité, trames Leaf, catalogue des fonctionnalités, glossaire |
| **Comprendre** pourquoi | [`explanation/`](explanation/) | Comment fonctionne le CAN/OBD, choix d'architecture |
| **Suivre** le projet | [`management/`](management/) | Plan, TODO, journal |

## Par où commencer

1. Comprendre : [`explanation/FONCTIONNEMENT_CAN_OBD.md`](explanation/FONCTIONNEMENT_CAN_OBD.md)
2. Faire le premier essai : [`tutorials/PREMIER_ESSAI_ROUTIER.md`](tutorials/PREMIER_ESSAI_ROUTIER.md)
3. État du projet et prochaines étapes : [`management/TODO.md`](management/TODO.md)
4. Le plan complet : [`management/PLAN.md`](management/PLAN.md) (source de vérité)

## Convention de fiabilité

Tout fait technique non confirmé porte une étiquette :

- **[vérifié]** : confirmé par une source publique sérieuse (documentation OVMS, fiche de composant).
- **[communauté]** : rapporté de façon concordante par la communauté, non confirmé officiellement.
- **[à valider]** : hypothèse à confirmer **sur le véhicule ou sur le matériel** avant de s'y fier.

Rien d'étiqueté « à valider » ne doit être codé en dur sans test sur une capture réelle.

## Règle de gestion

Le plan (`management/PLAN.md`) reste la source de vérité. Les autres documents le détaillent par usage ; en cas de contradiction, **corriger l'écart** dans les deux fichiers plutôt que de laisser deux versions. Le dépôt doit rester **privé** (voir `explanation/DONNEES_ET_VIE_PRIVEE.md`).
