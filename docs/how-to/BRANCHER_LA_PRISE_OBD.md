# Comment brancher la prise OBD-II en sécurité

Recette pour connecter un adaptateur sans risque. Contexte : [`../explanation/FONCTIONNEMENT_CAN_OBD.md`](../explanation/FONCTIONNEMENT_CAN_OBD.md).

## Avant

- Voiture éteinte, clé/contact coupé.
- Adaptateur déjà configuré en **écoute seule** (Listen-Only) : avec le CANable, cela suppose le micrologiciel candleLight (`ip link set can0 type can bitrate 500000 listen-only on`). En slcan, l'adaptateur acquitte les trames : 6/14 seulement (voir le tutoriel, étape 1).
- Résistance de terminaison de l'adaptateur **désactivée** (le bus de la voiture est déjà terminé).

## Mesurer, ne pas supposer

1. **Résistance H-L** voiture éteinte depuis quelques minutes : ~60 Ω sur un bus CAN correctement terminé. 120 Ω : une seule terminaison est vue ; ∞ : pas de bus sur ces broches ou mauvais contact ; ~0 Ω : court-circuit, ne rien brancher.
2. **Tension contact mis**, par rapport à la masse (broche 5) : bus au repos, H et L ≈ 2,5 V tous les deux ; avec du trafic, le multimètre lit une moyenne, H légèrement au-dessus (≈ 2,6-3,0 V), L légèrement en dessous (≈ 2,0-2,4 V), H + L ≈ 5 V **[vérifié, ISO 11898-2 ; lecture à valider]**. Les deux à ~2,5 V sans écart : bus sans trafic (voiture endormie, ou bus inactif) ; ~0 V : pas de bus alimenté sur ces broches.
3. Noter le résultat dans la fiche de l'essai.

Câblage vers le CANable : **6 → CANH, 14 → CANL, 5 → GND**.

## Brochage de référence

| Broche | Signal | Remarque |
|--------|--------|----------|
| 16 | +12 V permanent | Protéger (fusible, protection contre les surtensions) pour un montage permanent |
| 4 / 5 | Masse châssis / masse signal | Relier la masse du CANable à la 5 |
| 6 / 14 | CAN-H / CAN-L (diagnostic ISO 15765-4 ; 500 kb/s sur la plupart des voitures récentes, 250 kb/s admis par la norme) | Sur Leaf : bus Car-CAN ; sur Spark : HS-GMLAN **[communauté, à valider]** |
| 1 | Sur les GM (Spark) : GMLAN mono-fil 33,3 kb/s **[communauté, à valider]** | Ne pas y brancher le CANable (transceiver différent) |
| 12 / 13 | Sur Leaf AZE0 : EV-CAN **[communauté]** | **Écoute seulement** ; libellés H/L à mesurer |

## Interdits

- Ne jamais émettre sur 12/13.
- Ne jamais brancher un câble « Y » ou un cavalier sur une voiture autre que la Leaf sans avoir vérifié ce que portent 12/13 (d'autres constructeurs y mettent autre chose).
- Ne rien débrancher/couper des fils d'origine : on passe par la prise.

## Leaf 2018 (ZE1)

Le port OBD est isolé par une passerelle qui s'éteint contact coupé. Seules les requêtes sur 6/14 contact mis sont possibles ; l'écoute du bus rapide demande un câble de dérivation ailleurs (voir le plan §4.3).
