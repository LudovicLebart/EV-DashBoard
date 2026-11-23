# Glossaire

| Terme | Sens |
|-------|------|
| **CAN** | Bus de communication à deux fils entre calculateurs. Trames de 8 octets avec un identifiant |
| **EV-CAN / Car-CAN** | Sur Leaf : bus rapide batterie/moteur (broches 12/13) et bus de diagnostic standard (6/14) |
| **OBD-II** | Prise de diagnostic standard sous le volant |
| **DBC** | Fichier décrivant comment lire les signaux dans les trames |
| **LBC** | *Lithium Battery Controller* : calculateur de la batterie Nissan |
| **GID** | Unité d'énergie de la batterie Nissan (~77,5 à 80 Wh) |
| **SOC / SOH** | État de charge / état de santé de la batterie |
| **Hx** | Indicateur Nissan de conductance interne de la batterie |
| **ISO-TP** | Protocole qui découpe un message long (>8 octets) en plusieurs trames avec contrôle de flux |
| **UDS / KWP2000** | Protocoles de diagnostic. `0x21` est KWP2000 ; `0x22` est UDS |
| **Listen-Only** | Mode d'un contrôleur CAN qui écoute sans émettre ni acquitter |
| **Voltage sag** | Chute de tension d'une cellule sous forte charge ; signe d'une cellule faible |
| **Passerelle (gateway)** | Calculateur qui filtre le trafic entre le port OBD et les bus internes |
| **Profil** | Dossier décrivant un véhicule (trames, requêtes, capacités) |
| **Canonique** | Schéma de données commun à tous les véhicules |
| **N0 / N1 / N2** | Niveaux de support : générique, communautaire, référence |
| **TWAI** | Contrôleur CAN intégré à l'ESP32 |
| **PWA** | Application web installable |
| **EVLOG** | Format de journal brut du module (voir le plan §3.5) |
