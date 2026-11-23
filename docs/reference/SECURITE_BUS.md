# Référence — Règles de sécurité sur le bus

Règles **non négociables**. Détail et justification : [`../management/PLAN.md`](../management/PLAN.md) §5.2. Le projet n'est pas développé selon ISO 26262 : la doctrine est « lecture seule, liste blanche, passivité matérielle sur le bus moteur ».

## Règles

1. **Aucune émission sur l'EV-CAN** (broches 12/13 de la Leaf) : matériel et logiciel. En Phase 0 avec le CANable : écoute seule (Listen-Only) sur 12/13, sans exception, donc micrologiciel **candleLight** (`listen-only on`, vérifié par `ip -details link show can0`). En slcan, l'adaptateur acquitte les trames : il n'est **pas** passif et ne va jamais sur 12/13.
2. **Liste blanche d'émission** : seules les requêtes de lecture déclarées sont autorisées (PID standard `0x01`, services `0x21`, `0x22`, trames de contrôle de flux). Un profil ne peut pas élargir la liste sans décision explicite.
3. **Interdits** : services `0x10` (sauf session par défaut après revue), `0x11`, `0x14`, `0x27`, `0x28`, `0x2E`, `0x2F`, `0x31`, `0x34`-`0x37`, `0x3D`, `0x85`. Jamais d'écriture, d'effacement de défauts ni de pilotage.
4. **Jamais d'émission si le véhicule dort** : seulement si du trafic a été vu sur le bus dans les 500 dernières ms et que le véhicule est en marche ou en charge. Émettre sur un bus endormi réveille les calculateurs, décharge la batterie 12 V et peut laisser des codes défaut.
5. **Vérifier le débit avant d'émettre** : écouter d'abord ; un débit erroné en mode normal génère des trames d'erreur qui perturbent le bus.
6. **Bus-off ou compteurs d'erreurs en hausse** : arrêt des requêtes pour la session, événement journalisé, pas de reprise agressive (sous Linux : ne pas régler `restart-ms`).
7. **Limitation de débit** : ≤ 10 requêtes par seconde au total (le script de Phase 0 se limite à 5), une seule requête ISO-TP en vol par calculateur.
8. **CAN-FD** : un contrôleur CAN classique ne doit **jamais** être en mode normal sur un bus FD (il génère des trames d'erreur) ; en écoute seule seulement.
9. **Broches non standard (12/13, 3/11…)** : désactivées par défaut ; activées seulement par un profil qui les déclare.

## Option « commandes à distance » (Phase 8) : exception encadrée

Les règles 1 à 9 valent **tant que l'option est verrouillée, c'est-à-dire par défaut**. Si l'utilisateur la débloque (plan, Phase 8) :

- le déblocage est **explicite, par véhicule et par commande** ; pas de trame libre, seulement une liste blanche de commandes revues une à une ;
- **ordre des voies** (décision du 2026-11-23) : 1) API officielle, 2) API non officielle avec risques documentés, 3) **écriture sur le bus en dernier recours** ; la voie 3 est limitée au bus de diagnostic (6/14), **jamais l'EV-CAN**, une commande à la fois, reproduite d'abord sur banc ; jetons des API gardés sur le serveur, droits minimaux, révocables ;
- **jamais** : freins, direction, groupe motopropulseur, airbags, verrouillage de sécurité, mise à jour de logiciel, effacement de codes défaut ;
- voiture à l'arrêt et en état sûr, confirmation à chaque commande, journal d'audit, limitation de fréquence, **interrupteur d'arrêt** ;
- l'utilisateur accuse réception des risques (garantie, batterie 12 V, responsabilité).

## Garde-fous de conduite

- Aucune interaction avec l'écran en roulant ; support fixe.
- Un accessoire OBD n'annule pas la garantie en soi, mais un dommage qui lui est attribuable peut être refusé : une Leaf 2016-2017 peut encore être sous garantie de capacité batterie **[à valider selon pays et date]**.
- Aucune batterie lithium de secours dans l'habitacle (froid extrême, chaleur d'été).
- Après chaque essai : vérifier l'absence de **code défaut nouveau** au tableau de bord.
