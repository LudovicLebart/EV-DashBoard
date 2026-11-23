# Planification de trajets avec arrêts de recharge

Explication de la fonction « comme une Tesla » (plan, Phase 6b). Statut : **conception**, rien n'est implémenté.

## Le problème

Sur un long trajet, on ne veut pas seulement savoir « si on arrive ». On veut un **plan** : où s'arrêter, combien de temps, jusqu'à quel niveau de charge, avec assez de marge pour ne jamais tomber à plat, et des arrêts **assez courts** pour que le voyage reste agréable.

## Pourquoi c'est plus qu'une carte de bornes

Un bon plan combine quatre choses :

1. **Combien d'énergie coûte chaque tronçon** : dépend de la vitesse, de la pente, de la température et du chauffage. C'est le modèle d'autonomie de la Phase 6.
2. **À quelle vitesse la batterie se recharge** : la puissance n'est pas constante ; elle baisse quand le niveau de charge monte, et une batterie froide accepte moins de puissance. C'est le modèle de charge, appris des recharges enregistrées (Phase 4).
3. **Où sont les bornes** : connecteur compatible, puissance réelle, réseau, et une borne de repli si celle prévue est occupée ou en panne.
4. **Quoi optimiser** : le temps total, sous des contraintes de sécurité (marge à chaque arrivée) et de confort (durée d'arrêt proche de x minutes).

## Pourquoi « x minutes » change les choix

Recharger de 10 % à 80 % est rapide ; de 80 % à 100 % est lent. Un arrêt de 20 minutes sur une borne puissante peut donc donner presque autant d'énergie qu'un arrêt de 40 minutes. Le planificateur choisit, pour chaque arrêt, **jusqu'où charger** pour respecter la durée voulue, et ajoute un arrêt de plus si une seule pause ne suffit pas.

## Sécurité : conseiller prudemment

- Les marges se basent sur le **pire cas raisonnable**, pas sur la moyenne.
- Aucun arrêt ni arrivée sous la marge minimale.
- Un **plan B** à chaque arrêt.
- Ce qui est estimé et non calibré est affiché comme tel.
- Le planificateur ne commande jamais le véhicule et n'exige aucune interaction en roulant.

## Démarrage progressif

Avant d'avoir des données de votre voiture, une **v0** utilise des hypothèses explicites (consommation moyenne, courbe de charge générique) pour valider l'algorithme et l'interface. La **v1** se calibre sur vos trajets et vos recharges.

## Limites à connaître

- La **disponibilité en temps réel** des bornes n'est généralement pas accessible : d'où la borne de repli.
- La qualité des **bases de bornes ouvertes** varie selon la région **[à valider pour le Québec]**.
- Un modèle de charge fiable demande des recharges enregistrées, d'où la dépendance à la Phase 4.

Détails : [`../management/PLAN.md`](../management/PLAN.md), Phase 6b.
