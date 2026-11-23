# Comment brancher la prise OBD-II en sécurité

Recette pour connecter un adaptateur sans risque. Contexte : [`../explanation/FONCTIONNEMENT_CAN_OBD.md`](../explanation/FONCTIONNEMENT_CAN_OBD.md).

## Avant

- Voiture éteinte, clé/contact coupé.
- Adaptateur déjà configuré en **écoute seule** (Listen-Only) si possible.

## Mesurer, ne pas supposer

1. **Résistance H-L** voiture éteinte : ~60 Ω sur un bus CAN correctement terminé. 120 Ω : un seul bout est vu ; 0 ou ∞ : mauvais contact.
2. **Tension contact mis** : H ≈ 2,5-3,5 V, L ≈ 1,5-2,5 V par rapport à la masse. Un bus qui reste à 2,5 V sans variation est mort ou mal câblé.
3. Noter le résultat dans la fiche de l'essai.

## Brochage de référence

| Broche | Signal | Remarque |
|--------|--------|----------|
| 16 | +12 V permanent | Protéger (fusible, protection contre les surtensions) pour un montage permanent |
| 4 / 5 | Masses | |
| 6 / 14 | CAN-H / CAN-L (diagnostic standard, 500 kb/s) | Sur Leaf : bus Car-CAN |
| 12 / 13 | Sur Leaf AZE0 : EV-CAN **[communauté]** | **Écoute seulement** ; libellés H/L à mesurer |

## Interdits

- Ne jamais émettre sur 12/13.
- Ne jamais brancher un câble « Y » ou un cavalier sur une voiture autre que la Leaf sans avoir vérifié ce que portent 12/13 (d'autres constructeurs y mettent autre chose).
- Ne rien débrancher/couper des fils d'origine : on passe par la prise.

## Leaf 2018 (ZE1)

Le port OBD est isolé par une passerelle qui s'éteint contact coupé. Seules les requêtes sur 6/14 contact mis sont possibles ; l'écoute du bus rapide demande un câble de dérivation ailleurs (voir le plan §4.3).
