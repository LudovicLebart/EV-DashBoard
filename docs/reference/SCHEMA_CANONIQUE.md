# Référence — Schéma de données canonique (v1, brouillon)

Seule vue du véhicule que connaissent la base et l'interface. Source : [`../management/PLAN.md`](../management/PLAN.md) §3.

## Règles

- **Unités SI dans le nom** : `_v`, `_a`, `_w`, `_wh`, `_c`, `_pct`, `_kmh`, `_m`. Stockage en Wh/W ; conversion à l'affichage.
- **Signe** : `pack_current_a` positif en **décharge**, négatif en recharge/régénération ; `pack_power_w` suit.
- **Nullabilité** : signal absent = `null`/absent, **jamais 0**.
- **Provenance** de chaque série : `measured`, `derived`, `model`.
- **Horodatage** : `ts` UTC en µs + `ts_quality` (`gnss`, `ntp`, `interpolated`, `unknown`).
- **Versionnage** : SemVer (`canonical_schema: 1.0.0`).
- **Extensions constructeur** : `x.<marque>.<nom>` (ex. `x.nissan.gids`, `x.nissan.hx`).

## Signaux scalaires

| Signal | Unité | Notes |
|--------|-------|-------|
| `pack_voltage_v` | V | |
| `pack_current_a` | A | + = décharge |
| `pack_power_w` | W | `derived` si non fourni |
| `soc_bms_pct` | % | SOC du BMS (définition propre au constructeur) |
| `soc_display_pct` | % | SOC affiché, si disponible |
| `energy_remaining_bms_wh` | Wh | Leaf : GIDs × Wh/GID du profil |
| `energy_usable_est_wh` | Wh | `model` uniquement |
| `capacity_ah` | Ah | |
| `soh_pct` | % | Définition constructeur (`soh_definition` déclaré) |
| `cell_v_min_mv`, `cell_v_max_mv`, `cell_v_spread_mv` | mV | |
| `pack_temp_min_c`, `pack_temp_max_c` | °C | |
| `charge_power_limit_w`, `discharge_power_limit_w` | W | |
| `vehicle_speed_kmh` | km/h | CAN de préférence, GPS en secours |
| `motor_speed_rpm`, `motor_torque_nm` | tr/min, N·m | |
| `hvac_power_w` | W | Peut n'être que `derived`/`model` **[à valider sur Leaf]** |
| `ambient_temp_c`, `cabin_temp_c` | °C | |
| `aux_12v_v` | V | |
| `odometer_km` | km | |
| `vehicle_state` | enum | `off`, `awake`, `ready`, `charging_ac`, `charging_dc` |
| `gnss_lat`, `gnss_lon` | ° | Sensible |
| `gnss_alt_m`, `elevation_dem_m` | m | `elevation_dem_m` = `derived` d'un modèle de terrain |

## Signaux tableaux

| Signal | Forme | Notes |
|--------|-------|-------|
| `cell_voltages_mv[]` | `float[n]` | `n` déclaré par le profil (96 pour la Leaf) ; instantané non simultané |
| `pack_temps_c[]` | `float?[k]` | `k` déclaré ; sonde absente = `null` (30 kWh : 3 sondes vraisemblablement **[communauté, à valider]**) |
| `cell_balancing[]` | `bool[n]` | Si disponible |

## Capacités déclarées

Chaque profil déclare, par signal : transport, fréquence attendue, résolution, provenance, condition de disponibilité. L'interface construit ses vues à partir de cette liste.
