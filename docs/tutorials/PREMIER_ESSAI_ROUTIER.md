# Tutoriel — Premier essai routier (Phase 0)

> **Première voiture : Chevrolet Spark EV** (décision du 2026-11-23). Les étapes 1 à 5 et 7-8 valent pour la Spark ; l'étape 6 (requêtes `21 01/02/04` vers `0x79B`) est **spécifique à la Leaf** : sur la Spark, on utilise les PID de la liste communautaire de la Spark **[à valider]**. Les sections « Leaf » s'appliquent à l'essai suivant.

**But** : enregistrer les données d'une Leaf pendant un trajet, avec un ordinateur portable sous **Linux**, un adaptateur **CANable** et le **GPS du téléphone**, et obtenir de quoi remplacer LeafSpy.

**Statut : procédure non encore exécutée.** Les commandes sont les usages standard de `can-utils` et du noyau Linux ; toute ligne marquée **[à valider]** doit être confirmée au premier branchement, puis ce document corrigé.

**Durée** : une soirée de préparation, un trajet d'essai (30 à 60 min).

## Matériel (~30-50 $)

- Adaptateur USB-CAN **CANable 2.0** (~20-36 $).
- Câble ou breakout OBD-II mâle 16 broches (~10-15 $, estimation).
- Ordinateur portable avec **Linux** (le double démarrage Windows/Linux est prévu ; on prend Linux).
- Téléphone Android avec une appli d'enregistrement de trace GPS (gratuite, **[à choisir]**).
- Un multimètre (pour mesurer le brochage).

Prix et sources : [`../management/PLAN.md`](../management/PLAN.md) §5.5.

## Règles de sécurité (à lire avant de brancher)

- On **écoute d'abord**, sans rien envoyer. Les requêtes viennent à l'étape 6.
- Jamais d'émission sur 12/13 (bus rapide). Jamais d'écriture, d'effacement de défauts ni de service de session ou de sécurité.
- Requêtes seulement **contact mis, voiture à l'arrêt ou en roulant**, jamais sur un bus endormi.
- Détails : [`../reference/SECURITE_BUS.md`](../reference/SECURITE_BUS.md).

## Étape 1 — Préparer l'ordinateur (Linux)

1. Installer les outils : `sudo apt install can-utils python3-venv` (ou l'équivalent de votre distribution).
2. Brancher le CANable. Selon son micrologiciel, il apparaît soit comme interface réseau `can0` (micrologiciel « candleLight »), soit comme port série `/dev/ttyACM0` (micrologiciel « slcan ») **[à valider : le firmware livré dépend du vendeur]**.
3. Activer l'interface :
   - candleLight : `sudo ip link set can0 up type can bitrate 500000 listen-only on`
   - slcan : `sudo slcand -o -c -s6 /dev/ttyACM0 can0 && sudo ip link set can0 up`  (`-s6` = 500 kb/s ; ce mode n'est pas « listen-only » : **ne rien émettre** avant l'étape 6) **[à valider]**
4. Vérifier que l'interface existe : `ip -details link show can0`.

## Étape 2 — Mesurer le brochage de la Leaf

Voiture éteinte, au multimètre sur la prise OBD-II :

- **Résistance entre 6 et 14** : ~60 Ω environ (deux résistances de terminaison de 120 Ω en parallèle). Idem entre **12 et 13** sur Leaf 2016-2017 **[à valider]**.
- **Contact mis** : sur une paire, H ≈ 2,5-3,5 V et L ≈ 1,5-2,5 V par rapport à la masse (broche 4 ou 5) **[à valider]**.
- Noter lesquelles de 12 et 13 est H : les sources diffèrent.

Sur une **Leaf 2018 (ZE1)**, ne pas chercher 12/13 : elles ne sont pas exploitables au port OBD.

## Étape 3 — Écoute passive sur 6/14

1. Brancher le CANable sur **6 (CAN-H) et 14 (CAN-L)**.
2. Contact mis (sans démarrer) : `candump -L can0 | tee ~/leaf-6-14-contact.log`
3. Observer : combien d'identifiants différents ? Sur AZE0, 6/14 peut montrer très peu de trames **[communauté, à valider]** : c'est normal, les données utiles passent par les requêtes (étape 6).
4. Arrêter avec `Ctrl+C`.

## Étape 4 — Lancer l'enregistrement du trajet

1. Sur le téléphone : démarrer l'enregistrement de trace GPS et **noter l'heure** de départ.
2. Sur l'ordinateur : `candump -L -H can0 | tee ~/essai-AAAAMMJJ-HHMM.log` (horodatage absolu ; garder l'ordinateur sur batterie et le couvercle ouvert).
3. Fiche à remplir : modèle et année, kWh, température extérieure, SOC affiché, odomètre.

## Étape 5 — Faire le trajet

Scénario recommandé :

1. Contact mis à l'arrêt, 1 minute.
2. Démarrage, conduite normale.
3. **Au moins une forte accélération** (en sécurité), pour voir la chute de tension.
4. Arrêt, contact coupé, laisser enregistrer 2 minutes, puis arrêter l'ordinateur et le GPS.

## Étape 6 — Requêtes batterie (équivalent LeafSpy)

**À ne faire qu'après avoir compris l'écoute passive** ; seulement contact mis.

1. Charger le module noyau si nécessaire : `sudo modprobe can-isotp` **[à valider : disponible selon le noyau]**.
2. Dans un terminal, écouter les réponses : `isotpdump -s 79B -d 7BB can0`
3. Dans un autre : `echo "21 01" | isotpsend -s 79B -d 7BB can0`

   Les requêtes visées : `21 01` (capacité, SOH, Hx), `21 02` (tensions des 96 groupes, ~196 octets en plusieurs trames), `21 04` (températures), `21 06` **[à valider]**. Maximum **10 requêtes par seconde au total**, une seule en vol à la fois.
4. Si **aucune réponse** : la passerelle ne route peut-être pas les requêtes sur 6/14. Le noter, c'est un résultat important (voir le plan §4.2 pour le repli).

## Étape 7 — Vérifier les données

1. Recaler le GPS : les deux fichiers ont des heures à la seconde près ; confirmer sur un événement visible (départ du véhicule).
2. Décoder hors ligne. Contrôles de bon sens : voir [`../how-to/VERIFIER_UN_DECODAGE.md`](../how-to/VERIFIER_UN_DECODAGE.md).
3. Comparer avec le tableau de bord (SOC, tension).

## Étape 8 — Archiver

Placer journaux, trace GPS et fiche dans `profiles/nissan/<modèle>/captures/` **uniquement après anonymisation** (VIN, GPS) ; les fichiers complets restent sur le serveur, hors Git. Règles : [`../explanation/DONNEES_ET_VIE_PRIVEE.md`](../explanation/DONNEES_ET_VIE_PRIVEE.md).

## Critères de réussite

Pour chaque Leaf : requêtes `21 01/02/04` obtenues (ou absence de réponse documentée) ; SOH, Hx, Ah, 96 tensions et températures décodés et plausibles ; somme des 96 tensions à ≤ 1 % de la tension du pack au repos ; au moins une forte accélération capturée ; aucun code défaut nouveau au tableau de bord.

## Et ensuite

- **Leaf 2016-2017 (AZE0)** : second essai, **même CANable déplacé sur 12/13 en écoute seule uniquement** (jamais d'émission) pour le flux rapide (~100 Hz).
- Écrire le profil : [`../how-to/AJOUTER_UN_PROFIL_VEHICULE.md`](../how-to/AJOUTER_UN_PROFIL_VEHICULE.md).
