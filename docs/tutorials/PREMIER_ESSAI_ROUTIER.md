# Tutoriel — Premier essai routier (Phase 0)

> **Première voiture : Chevrolet Spark EV** (décision du 2026-11-23), **puis** chaque Leaf 2016-2018 accessible. La méthode est la même ; seules changent les broches écoutées et les requêtes batterie (repères **Spark** / **Leaf** dans chaque étape).

**But** : enregistrer, pendant un vrai trajet, les trames brutes et les signaux batterie, avec un ordinateur portable sous **Linux**, un adaptateur **CANable** et le **GPS du téléphone** — d'abord sur la Spark, ensuite sur une Leaf, pour remplacer LeafSpy.

**Statut : procédure non encore exécutée.** Les commandes sont les usages documentés de `can-utils` et du noyau Linux ; toute ligne marquée **[à valider]** doit être confirmée au premier branchement, puis ce document corrigé.

**Durée** : une soirée de préparation (dont le reflashage éventuel du CANable), une séance à l'arrêt (~30 min), un trajet (30 à 60 min).

## Matériel (~30-50 $)

- Adaptateur USB-CAN **CANable 2.0** (~19-36 $).
- Câble ou breakout OBD-II mâle 16 broches **à bornier ou fils libres** (~10-15 $, estimation) : il faut relier séparément CAN-H, CAN-L et la masse au bornier du CANable.
- Ordinateur portable avec **Linux** (le double démarrage Windows/Linux est prévu ; on prend Linux).
- Téléphone Android avec une appli d'enregistrement de trace GPS gratuite exportant en GPX (par exemple GPSLogger, logiciel libre ; **[à choisir]**).
- Un multimètre (pour mesurer le brochage).

Prix et sources : [`../management/PLAN.md`](../management/PLAN.md) §5.5.

## Prérequis (à vérifier à la maison)

- Noyau **≥ 5.10** (`uname -r`) : le module ISO-TP `can-isotp` y est intégré **[vérifié]**. `sudo modprobe can-isotp` ne doit pas renvoyer d'erreur.
- Python **≥ 3.8** (`python3 --version`) ; aucune bibliothèque à installer pour le script de l'étape 4.
- **Heure juste** : `timedatectl` doit afficher « System clock synchronized: yes » avant de partir (dans la voiture, l'ordinateur n'aura souvent pas de réseau) ; le téléphone est à l'heure du réseau.
- Mise en veille automatique désactivée (inactivité **et** fermeture du couvercle), batterie chargée.
- Répétition à blanc : CANable branché en USB seul, interface montée (étape 1), `candump can0` n'affiche rien et ne plante pas.

## Règles de sécurité (à lire avant de brancher)

- On **écoute d'abord**, sans rien envoyer. Les requêtes viennent à l'étape 4, à l'arrêt.
- Jamais d'émission sur 12/13 (bus rapide de la Leaf) : ni requête, ni même **acquittement** (ACK). Sur 12/13, **seulement** avec un mode écoute seule confirmé (micrologiciel candleLight, étape 1).
- Jamais d'écriture, d'effacement de défauts ni de service de session ou de sécurité.
- Requêtes seulement **contact mis**, jamais sur un bus endormi ; arrêter les requêtes **avant** de couper le contact.
- Toute trame d'erreur ou tout passage en `ERROR-PASSIVE`/`BUS-OFF` : **débrancher**, comprendre, ne pas insister.
- Détails : [`../reference/SECURITE_BUS.md`](../reference/SECURITE_BUS.md).

## Étape 1 — Préparer l'ordinateur et le CANable

1. Installer les outils : `sudo apt install can-utils python3` (ou l'équivalent de votre distribution).
2. Brancher le CANable en USB et lancer `sudo dmesg | tail` :
   - `gs_usb` et une interface `can0` → micrologiciel **candleLight** ;
   - `cdc_acm` et un port `/dev/ttyACM0` → micrologiciel **slcan**. C'est le micrologiciel livré par défaut sur la CANable 2.0 **[communauté, vérifier sur l'exemplaire reçu]**.
3. **Recommandé : passer en candleLight**, seul moyen simple d'avoir une **écoute seule** fiable sous Linux. Outil officiel : programme de mise à jour web `https://canable.io/updater` (Chrome ou Chromium, carte en mode démarrage selon la notice) ; pour la 2.0 il propose « slcan with FD support » ou « candlelight » **[communauté, à valider au moment du flash]**. L'opération est réversible. Certains clones (carte MKS à STM32G431) n'ont pas forcément de candleLight compatible **[communauté]** : dans ce cas, rester en slcan (point 5).
4. **candleLight** — interface en écoute seule à 500 kb/s :
   ```
   sudo ip link set can0 type can bitrate 500000 listen-only on
   sudo ip link set can0 up
   ip -details link show can0      # doit afficher <LISTEN-ONLY> et bitrate 500000
   ```
   Ne pas régler `restart-ms` : après un bus-off, pas de reprise automatique (règle de sécurité).
5. **slcan** (si candleLight est impossible) :
   ```
   sudo slcand -o -c -s6 /dev/ttyACM0 can0      # -s6 = 500 kb/s
   sudo ip link set can0 up
   ```
   Dans ce mode, l'adaptateur **acquitte** les trames (il n'est pas en écoute seule). Sur un bus actif au bon débit, c'est sans effet notable, mais c'est une émission : **6/14 seulement, jamais 12/13**. L'option `slcand -l` envoie la commande `L` (écoute seule), que le micrologiciel slcan de la CANable 2.0 ne documente pas ; il documente à la place un mode silencieux `M1`, à envoyer avant l'ouverture **[vérifié dans la doc du micrologiciel ; effet réel à valider]**. Tant que ce mode n'est pas prouvé, considérer l'adaptateur slcan comme **non passif**.
6. **Résistance de terminaison** : si l'adaptateur a une résistance de 120 Ω activable (cavalier ou interrupteur selon la carte **[à valider sur la carte reçue]**), la laisser **désactivée** : le bus de la voiture est déjà terminé.

## Étape 2 — Mesurer le brochage

Au multimètre, sur la prise OBD-II :

- **Voiture éteinte depuis quelques minutes**, résistance entre **6 et 14** : ~60 Ω (deux terminaisons de 120 Ω en parallèle). 120 Ω : une seule terminaison vue ; ∞ : pas de bus sur ces broches. **Leaf 2016-2017** : idem entre 12 et 13 **[à valider]**.
- **Contact mis**, tension par rapport à la masse (broche 5, masse signal ; ou 4, masse châssis) : au repos du bus, H et L sont tous deux à ~2,5 V ; avec du trafic, le multimètre affiche une moyenne, H un peu au-dessus (≈ 2,6-3,0 V) et L un peu en dessous (≈ 2,0-2,4 V), H + L ≈ 5 V. Valeurs instantanées en bit dominant : H ≈ 3,5 V, L ≈ 1,5 V **[vérifié, ISO 11898-2 ; lecture au multimètre à valider]**.
- **Spark** : sur les GM de cette génération, la broche **1** porte le GMLAN mono-fil (33,3 kb/s) **[communauté, à valider]** : ne pas y brancher le CANable.
- **Leaf** : noter laquelle de 12 et 13 est H (la plus haute) : les sources diffèrent. Sur une **Leaf 2018 (ZE1)**, ne pas chercher 12/13 : elles ne sont pas exploitables au port OBD.

## Étape 3 — Écoute passive sur 6/14 (à l'arrêt)

1. Câbler **6 → CANH**, **14 → CANL**, **5 → GND** du CANable. Brancher sur la prise, voiture éteinte, puis mettre le contact (sans démarrer).
2. Enregistrer : `candump -L -d -e can0 > ~/spark-6-14-contact.log` (`-L` : format journal, horodatage absolu de l'horloge système ; `-d` : signale les trames perdues ; `-e` : affiche les trames d'erreur). Dans un second terminal, `cansniffer -c can0` montre les identifiants en direct.
3. Observer :
   - **Spark** : 6/14 porte le bus HS-GMLAN, on s'attend à beaucoup d'identifiants **[communauté, à valider]**.
   - **Leaf AZE0** : 6/14 peut montrer très peu de trames **[communauté, à valider]** : c'est normal, les données utiles passent par les requêtes (étape 4) ou par 12/13.
4. Arrêter avec `Ctrl+C`. Compter les identifiants : `awk '{print $3}' ~/spark-6-14-contact.log | cut -d'#' -f1 | sort | uniq -c | sort -rn | head -40`.

Aucune trame ? Voir **Dépannage** avant d'aller plus loin.

## Étape 4 — Requêtes de lecture (à l'arrêt, contact mis)

**Seulement après une étape 3 réussie** (trames reçues, aucune trame d'erreur : le débit est confirmé).

1. Repasser l'interface en mode normal (pour émettre, il faut acquitter) :
   - candleLight : `sudo ip link set can0 down && sudo ip link set can0 type can bitrate 500000 listen-only off && sudo ip link set can0 up` ;
   - slcan : déjà en mode normal.
2. `sudo modprobe can-isotp`.
3. Enregistrer le script ci-dessous sous `~/lire_isotp.py`. Il n'utilise que la bibliothèque standard : une seule socket ISO-TP du noyau envoie la requête **et** reçoit la réponse (le noyau gère le contrôle de flux des réponses longues), une requête à la fois, ≤ 5 par seconde, services de lecture uniquement.

   ```python
   #!/usr/bin/env python3
   """lire_isotp.py TX RX REQ [REQ...] : requêtes de LECTURE ISO-TP, une à la fois.
   Ex. Leaf : python3 lire_isotp.py 79B 7BB 2101 2102 2104"""
   import socket, struct, sys, time
   SOL_CAN_ISOTP, CAN_ISOTP_OPTS, CAN_ISOTP_TX_PADDING = 106, 1, 0x004  # <linux/can/isotp.h>
   LECTURE = (0x01, 0x21, 0x22)          # liste blanche minimale (PID standard, 0x21, 0x22)
   tx, rx = int(sys.argv[1], 16), int(sys.argv[2], 16)
   reqs = [bytes.fromhex(r) for r in sys.argv[3:]]
   for r in reqs:
       if r[0] not in LECTURE:
           sys.exit(f"refusé : service 0x{r[0]:02X} hors liste blanche")
   s = socket.socket(socket.AF_CAN, socket.SOCK_DGRAM, socket.CAN_ISOTP)
   # trames complétées à 8 octets (0x00) : beaucoup de calculateurs l'exigent [à valider]
   s.setsockopt(SOL_CAN_ISOTP, CAN_ISOTP_OPTS,
                struct.pack("=IIBBBB", CAN_ISOTP_TX_PADDING, 0, 0, 0x00, 0, 0))
   s.bind(("can0", rx, tx))              # (interface, id de réponse, id de requête)
   s.settimeout(1.0)
   for r in reqs:
       s.send(r)
       try:
           rep = s.recv(4095).hex(" ")
       except socket.timeout:
           rep = "PAS DE REPONSE"
       print(f"{time.time():.3f} {tx:03X} {r.hex(' ')} -> {rep}", flush=True)
       time.sleep(0.2)
   ```
4. Dans un autre terminal, pour voir les échanges trame par trame : `isotpdump -s <TX> -d <RX> can0` (par exemple `-s 79B -d 7BB`).
5. **Toute voiture — PID standard (mode 01)** : `python3 ~/lire_isotp.py 7DF 7E8 0100` (PID 00 = liste des PID pris en charge, demande « à tous » sur 0x7DF, première réponse lue sur 0x7E8 ; d'autres calculateurs peuvent répondre de 0x7E9 à 0x7EF, visibles dans `isotpdump`/`candump`). Puis quelques PID : `010D` (vitesse), `0142` (tension module), `0146` (température extérieure), `015B` (batterie hybride restante). **Un VÉ n'est pas tenu d'y répondre.**
6. **Spark** — calculateur batterie (BECM), service 0x22 : en-tête **0x7E7**, réponse attendue sur **0x7EF** (convention GM +8) **[communauté, à valider]**. Les PID viennent de la **liste communautaire de la Bolt EV** (utilisée par les propriétaires de Spark avec Torque Pro ; aucune liste propre à la Spark n'a été trouvée) **[communauté, à valider PID par PID]** : `python3 ~/lire_isotp.py 7E7 7EF 22xxxx`. Réponse positive : commence par `62` ; `7F 22 31` (par exemple) = PID refusé, le noter.
7. **Leaf** : `python3 ~/lire_isotp.py 79B 7BB 2101 2102 2104` (+ `2106` **[à valider]**). Réponse positive : commence par `61 01`, `61 02`… ; `21 02` fait ~196 octets en plusieurs trames.
8. **Aucune réponse** : voir Dépannage. Pour la Leaf, cela peut signifier que la passerelle ne route pas les requêtes sur 6/14 : le noter, c'est un résultat important (plan §4.2).

## Étape 5 — Le trajet

1. Sur le téléphone : démarrer l'enregistrement de trace GPS et **noter l'heure** de départ.
2. Terminal 1 (pendant tout l'essai) : `candump -L -d -e can0 > ~/essai-$(date +%Y%m%d-%H%M).log`. Ne pas afficher le flux dans le terminal (jusqu'à ~2000 trames/s) ; `cansniffer` dans un autre terminal si besoin.
3. Terminal 2 (facultatif, seulement si l'étape 4 a réussi) : boucle lente de requêtes, par exemple Leaf :
   `while true; do python3 ~/lire_isotp.py 79B 7BB 2101 2102 2104; sleep 10; done | tee ~/requetes-$(date +%Y%m%d-%H%M).log`
   (Spark : les PID qui ont répondu à l'étape 4). Les réponses figurent aussi dans le journal `candump`.
4. Fiche à remplir : modèle et **année**, kWh, température extérieure, SOC affiché, odomètre, chauffage utilisé.
5. Scénario recommandé :
   1. Contact mis à l'arrêt, 1 minute.
   2. Démarrage, conduite normale.
   3. **Au moins une forte accélération** (en sécurité), pour voir la chute de tension.
   4. Arrêt ; **arrêter la boucle de requêtes (Ctrl+C)**, puis couper le contact ; laisser `candump` tourner 2 minutes (le bus doit se taire), puis arrêter l'ordinateur et le GPS.
6. Personne ne manipule l'ordinateur en roulant : passager ou rien.

## Étape 6 — Vérifier les données

1. Recaler le GPS : les deux fichiers sont à l'heure à la seconde près si les horloges étaient synchronisées **[à valider]** ; confirmer sur un événement visible (départ du véhicule : vitesse GPS > 0 et premières trames de traction).
2. Décoder hors ligne. Contrôles de bon sens : [`../how-to/VERIFIER_UN_DECODAGE.md`](../how-to/VERIFIER_UN_DECODAGE.md).
3. Comparer avec le tableau de bord (SOC, tension).

## Étape 7 — Archiver

Journaux, trace GPS et fiche complets restent **sur le serveur**, hors Git. Seules des captures **anonymisées** (VIN masqué, GPS retiré) entrent dans `profiles/<marque>/<modèle>/captures/` (par exemple `profiles/chevrolet/spark-ev/captures/`). Règles : [`../explanation/DONNEES_ET_VIE_PRIVEE.md`](../explanation/DONNEES_ET_VIE_PRIVEE.md).

## Dépannage

| Symptôme | Causes probables | Que faire |
|----------|------------------|-----------|
| Pas d'interface `can0` | Micrologiciel slcan sans `slcand` ; câble USB de charge seule | `sudo dmesg \| tail` ; lancer `slcand` (étape 1.5) ; changer de câble |
| `ip link set can0 up` refuse (« Invalid argument ») | Débit non réglé avant `up` (candleLight) ; en slcan, le débit se règle par `slcand -s6`, pas par `ip` | Régler `type can bitrate 500000` puis `up` |
| Interface montée, **aucune trame** | Contact non mis (bus endormi) ; CANH/CANL inversés ; masse non reliée ; broches sans bus ; mauvais débit | `ip -details -statistics link show can0` (état, compteurs) ; revérifier l'étape 2 ; **essayer 250 kb/s en écoute seule uniquement** (candleLight) |
| Trames d'erreur, état `ERROR-PASSIVE` ou `BUS-OFF` | Débit faux, câblage, terminaison ajoutée | **Débrancher immédiatement** ; ne pas insister ; vérifier débit, H/L, terminaison |
| `candump -d` signale des pertes | Ordinateur trop chargé, tampon trop petit | Fermer les autres programmes ; `candump -r 1048576 …` |
| La requête échoue à l'envoi | Interface encore en `listen-only on` ; `can-isotp` non chargé | Étape 4.1 et 4.2 |
| Aucune réponse à une requête | Mauvaises adresses ; contact non mis ; complétion (padding) attendue différente ; passerelle qui ne route pas ; calculateur qui exige une session de diagnostic | Vérifier les adresses dans `isotpdump` ; essayer sans complétion (retirer `setsockopt`) ; **ne pas** ouvrir de session (service 0x10, interdit hors revue) ; documenter |
| Réponse `7F xx 11` / `12` / `31` | Service / sous-fonction / PID non pris en charge | Normal pour une partie des PID : le noter dans la fiche |

## Critères de réussite

- **Spark** : brochage confirmé par mesure ; table des identifiants observés avec leur période ; au moins un lot de signaux batterie (tension, courant, SOC) décodé et cohérent avec le tableau de bord ; au moins 5 trajets enregistrés dont une forte accélération, trace GPS recalée ; aucun code défaut nouveau au tableau de bord.
- **Chaque Leaf** : requêtes `21 01/02/04` obtenues (ou absence de réponse documentée) ; SOH, Hx, Ah, 96 tensions et températures décodés et plausibles ; somme des 96 tensions à ≤ 1 % de la tension du pack au repos ; au moins une forte accélération capturée ; aucun code défaut nouveau.

## Et ensuite

- **Leaf 2016-2017 (AZE0)** : second essai, **même CANable déplacé sur 12/13, en écoute seule confirmée (candleLight)** — jamais d'émission ni d'acquittement — pour le flux rapide (~100 Hz).
- Écrire le profil : [`../how-to/AJOUTER_UN_PROFIL_VEHICULE.md`](../how-to/AJOUTER_UN_PROFIL_VEHICULE.md).
