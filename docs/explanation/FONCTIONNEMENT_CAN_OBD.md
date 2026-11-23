# Comment fonctionnent le CAN et la prise OBD-II

Explication, sans manipulation. Pour agir : voir [`../tutorials/PREMIER_ESSAI_ROUTIER.md`](../tutorials/PREMIER_ESSAI_ROUTIER.md).

## 1. Le bus CAN

Les calculateurs d'une voiture (batterie, onduleur, tableau de bord…) communiquent sur un **bus CAN** : deux fils (CAN-H et CAN-L) partagés par tous. Chaque calculateur y envoie de petits messages, appelés **trames** :

- un **identifiant** (par exemple `0x1DB`) qui dit *de quoi* parle la trame ;
- jusqu'à **8 octets** de données.

Il n'y a pas de destinataire : tout le monde entend tout. Les trames **périodiques** (courant du pack, par exemple) reviennent toutes les 10 à 500 ms. Aucune n'est « demandée ».

## 2. Les trames sont brutes

Une trame dit « identifiant 0x1DB : 8 octets ». Pour savoir que les octets 0 et 1 sont le courant, il faut un **dictionnaire de décodage** (fichier DBC, ou profil). Il donne, pour chaque signal : position dans la trame, facteur d'échelle, décalage, signe, unité. Une erreur de signe ou de facteur donne des chiffres faux **sans aucune alerte** : d'où les contrôles croisés du plan (somme des tensions de cellules contre tension du pack, etc.).

## 3. Écouter ou demander

Il y a deux façons d'obtenir des données :

| Mode | Principe | Exemple |
|------|----------|---------|
| **Passif (écoute)** | On lit ce qui passe déjà | Courant et tension du pack, vitesse du moteur |
| **Actif (requête)** | On envoie une question à un calculateur, il répond | Tensions des 96 groupes de cellules, SOH, Hx |

Les données lourdes (les 96 tensions, environ 196 octets) ne passent pas toutes seules : elles tiennent dans plusieurs trames, d'où le protocole **ISO-TP** (voir le glossaire) qui découpe la réponse en morceaux et gère le contrôle de flux.

**Conséquence importante** : écouter ne perturbe pas le bus (si le matériel est bien en mode « écoute seule »), mais **émettre** peut le perturber (codes défaut, réveil de calculateurs, batterie 12 V qui se décharge). Le projet s'interdit d'émettre sur le bus moteur et limite l'émission à une liste blanche de requêtes de lecture (voir [`../reference/SECURITE_BUS.md`](../reference/SECURITE_BUS.md)).

## 4. La prise OBD-II

La prise sous le volant donne accès à des broches fixes :

- **16** : +12 V permanent ; **4 / 5** : masses ;
- **6 / 14** : CAN-H / CAN-L du bus de diagnostic standard (500 kb/s) ;
- sur la Leaf (AZE0, 2013-2017) : **12 / 13**, un second bus (EV-CAN) qui porte le flux rapide de la batterie et du moteur **[communauté]** ; les libellés H/L de ces deux broches diffèrent selon les sources, donc **on les mesure** avant de brancher.

Sur la Leaf **ZE1 (2018)**, le port OBD est isolé par une **passerelle** : le bus rapide n'y est plus accessible (il faut un câble de dérivation ailleurs dans la voiture), mais les requêtes de diagnostic passent par 6/14 contact mis **[vérifié pour la passerelle ; communauté pour les requêtes]**.

## 5. Pourquoi un simple « câble OBD vers le téléphone » ne fait pas l'affaire

- Un téléphone ne parle pas CAN ; il lui faut un **traducteur** (adaptateur USB-CAN, ou dongle Bluetooth/Wi-Fi) qui contient lui-même un petit processeur et une puce CAN.
- Les dongles courants (ELM327) ne tiennent pas le débit complet du bus : on ne voit qu'une partie des données.
- Sans enregistreur autonome, le téléphone doit rester allumé et l'appli ouverte pendant tout le trajet.

C'est ce qui justifie, par étapes, l'architecture décrite dans [`ARCHITECTURE_ET_CHOIX.md`](ARCHITECTURE_ET_CHOIX.md).

## 6. Ce que LeafSpy fait, et ce qu'on veut de plus

LeafSpy interroge la batterie à intervalles de quelques secondes et n'affiche/n'enregistre que des valeurs décodées. Ce projet veut :

- garder les **trames brutes complètes**, donc pouvoir **redécoder** plus tard ;
- capter le flux rapide (≈ 100 Hz) pour voir les **chutes de tension sous forte charge** (« voltage sag ») ;
- relier ces données au GPS, à la température et au chauffage, pour prédire l'**autonomie hivernale**.
