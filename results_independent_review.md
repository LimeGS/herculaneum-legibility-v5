# Revisión científica independiente de resultados V5

Estado: **PASS de integridad analítica; el gate predeclarado de R2B es FAIL**. No encontré una discrepancia numérica o de denominadores en `results/evaluation/comparisons.json` (SHA-256 `ffa43f2e4f27652b1bf29f9de27fd58c16bd36cc7d93f4e4693dc09db658fe94`). Esta revisión no elige una seed ni cambia el brazo candidato.

## Resultado principal

El promedio de las métricas, con igual peso para los cuatro rollos evaluables y luego para las tres seeds predeclaradas, es:

| Brazo | Macro AUROC | DE poblacional entre seeds |
|---|---:|---:|
| R1 | 0,859737 | 0,016575 |
| R2 | 0,832155 | 0,006798 |
| R2B | 0,810478 | 0,022012 |

R2−R1 es −0,027582. Por rollo, R2 mejora PHerc0139 en +0,017861 y disminuye PHerc0814 en −0,064119, PHerc1667 en −0,044039 y PHercParis4 en −0,020031. El aporte de los 430 ejemplos históricos adicionales, bajo esta receta fija, no produjo un incremento consistente entre rollos.

R2B−R2 es −0,021677. El balance por celdas mejora PHerc1667 en +0,052632 y PHercParis4 en +0,006951, pero disminuye PHerc0139 en −0,038732 y PHerc0814 en −0,107561. Frente a R1, R2B queda en −0,049259. Los deltas R2B−R1 son negativos en las tres seeds: −0,086595, −0,060874 y −0,000308. Esto muestra heterogeneidad territorial y por seed, no soporte para una mejora general del candidato predeclarado.

R2B sí tiene cobertura completa, 15/15 jobs, pero falla las otras dos reglas del gate:

- mejora macro exigida ≥ +0,01; observada −0,049259;
- regresión mínima por rollo ≥ −0,03; peor rollo PHerc0814, −0,171679.

El resultado contractual es `quality_status=EXPERIMENTAL`, sin reemplazo del baseline público ni afirmación de validación independiente.

## Resultado por rollo

Los valores CNN son la media de las tres seeds. F0/O1/O2 tienen un único valor por fold.

| Test | R1 | R2 | R2B | F0 | O1 | O2 |
|---|---:|---:|---:|---:|---:|---:|
| PHerc0139 | 0,829621 | 0,847481 | 0,808750 | 0,639675 | 0,742324 | 0,731337 |
| PHerc0172 | — | — | — | — | — | — |
| PHerc0814 | 0,775063 | 0,710944 | 0,603383 | 0,579574 | 0,550125 | 0,541353 |
| PHerc1667 | 0,976728 | 0,932689 | 0,985320 | 0,952739 | 0,812567 | 0,851235 |
| PHercParis4 | 0,857536 | 0,837505 | 0,844457 | 0,851141 | 0,668771 | 0,515454 |

PHerc0172 contiene 19 negativos y ningún positivo. Sus 9 jobs CNN y los tres baselines conservan cobertura, pero AUROC y average precision son no evaluables. Cada macro usa exactamente cuatro rollos por seed; cada comparación CNN–CNN o CNN–baseline contiene 12 pares fold/seed evaluables. En las comparaciones con un baseline fijo, su valor por fold se empareja con cada una de las tres seeds; no representa tres ajustes independientes del baseline.

La average precision acompaña la degradación en los rollos de menor prevalencia: en PHerc0139 las medias R1/R2/R2B son 0,7252/0,7155/0,6483 y en PHerc0814, 0,6125/0,5415/0,4351. PHerc1667 y PHercParis4 tienen prevalencias 0,905 y 0,954, por lo que sus AP cercanas a uno deben leerse junto a esa prevalencia y no compararse directamente entre rollos.

![AUROC y average precision por rollo](figures/results_per_roll_metrics.png)

La figura usa barras de error de una DE poblacional entre las tres seeds para R1/R2/R2B. F0/O1/O2 son puntos sin barra porque hay un ajuste o referencia por fold. PHerc0172 queda sombreado y excluido explícitamente. Artefactos: PNG SHA-256 `45c84f0080c73ac0c2476c419170ece13e5e6675b64ffa2e74173b42c6d6abaa`; SVG SHA-256 `3731658344c5d9e8fce7d881146562c0a3ebe4c736f6aecb82bab77ec9b97a0a`.

## Denominadores y controles

Verifiqué directamente los 45 `test_predictions.jsonl`: cada archivo contiene los IDs exactos, únicos y en el orden del test completo de su fold. Los conteos por job son 195, 19, 61, 147 y 1.984; coverage es 1,0 en los 45. Cada original aparece como test una vez por brazo/seed. No se seleccionó la mejor seed: los agregados usan las tres.

Recalculé los 10 ajustes OLS desde `features.npy` y `labels.npy`. O1 usa exactamente `train_r1` y O2 exactamente `train_r2` en cada fold, con intercepto. Sus conteos, cinco coeficientes y AUROC de test coinciden con el JSON hasta tolerancia absoluta 1e-14. No se reutilizó el OLS LOSO histórico que incluía el futuro rollo de validación. Los macros descriptivos de estas referencias son O1 0,693447 y O2 0,659845 sobre los mismos cuatro rollos evaluables.

También reconstruí F0 desde las fuentes hasheadas en `training_config.json`: 2.395 scores cacheados y las 11 inferencias de completado forman los 2.406 IDs, sin faltantes. El macro F0 recalculado es 0,755782. PHercParis4 está marcado `known_s1_training_exposure`; los otros cuatro rollos, `non_s1_independence_not_established`. Por eso los deltas emparejados frente a F0 describen este archivo pero no prueban superioridad en un test independiente. En particular, R1−F0 es +0,103955 y R2B−F0 +0,054695 en los 12 pares evaluables, con esta limitación de exposición intacta.

## Refit predeclarado

Las 15 fracciones `selected_step/optimizer_steps` de R2B tienen mediana 2/3. Para los 2.836 casos, el presupuesto completo es `6*ceil(2836/32)=534`; la regla congelada produce 356 updates. El plan conserva R2B, seed 0, `quality_status=EXPERIMENTAL` y declara `uses_outer_test_metrics=false`. Ese refit genera un artefacto utilizable, pero no es una evaluación fuera de muestra y no modifica el FAIL del gate.

## Límites científicos

El alcance sigue siendo `historical_unharmonized_diagnostic`: faltan grupos territoriales finos, máscaras físicas, escala geométrica certificada y un test confirmatorio nuevo. No hay intervalo territorial porque `physical_group` es desconocido. Tres seeds describen sensibilidad de inicialización/orden, pero no sustituyen replicación territorial. No se ejecutó nuevamente el evaluador ni se modificaron resultados, protocolo, brazo o seed.
