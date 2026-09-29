# Resultado del entrenamiento V5

Completamos 45 fits, cero fallos y 21.654 predicciones de test con cobertura completa. R1, el reentrenamiento sobre la base corregida, rindió mejor que las variantes con datos adicionales o balanceo. La candidata predeclarada R2B **no pasó el criterio de aceptación** y se exporta como experimental.

## Cuánto aportó cada cambio

La métrica principal es AUROC macro: igual peso para los cuatro rollos evaluables, promediado sobre las tres semillas. Es una medida de ranking, no porcentaje de aciertos ni una probabilidad calibrada.

| Modelo o control | Macro AUROC | DE entre semillas |
|---|---:|---:|
| F0: proxy v4 archivado | 0,755782 | Referencia fija |
| O1: estadísticas de píxeles, train R1 | 0,693447 | Un ajuste por fold |
| O2: estadísticas de píxeles, train R2 | 0,659845 | Un ajuste por fold |
| R1: originales corregidos | **0,859737** | 0,016575 |
| R2: R1 + adicionales admitidos | 0,832155 | 0,006798 |
| R2B: R2 + balanceo por rollo/clase | 0,810478 | 0,022012 |

| Comparación | Cambio de AUROC |
|---|---:|
| R1 frente a v4 archivado | +0,103955 |
| Agregar datos: R2−R1 | **−0,027582** |
| Agregar balanceo: R2B−R2 | **−0,021677** |
| Candidata R2B frente a R1 | **−0,049259** |
| Candidata R2B frente a v4 archivado | +0,054695 |

R2−R1 y R2B−R2 mantienen arquitectura, inicialización, pasos, semillas y backend. R1 frente a v4 compara un reentrenamiento completo con una referencia histórica: no permite atribuir toda la diferencia exclusivamente a corregir etiquetas. v4 tuvo exposición conocida a S1/PHercParis4 y su independencia en los demás rollos no está establecida. Las diferencias frente a F0 son retrospectivas; no prueban superioridad en una nueva muestra externa.

Las semillas R1 dieron 0,868991, 0,873760 y 0,836459; las R2B, 0,782396, 0,812886 y 0,836151. No se eligió la mejor semilla de test. Su dispersión no sustituye un intervalo de confianza territorial.

## Resultados por rollo

Los valores CNN son medias de las tres semillas:

| Rollo reservado | N test | v4 | R1 | R2 | R2B |
|---|---:|---:|---:|---:|---:|
| PHerc0139 | 195 | 0,639675 | 0,829621 | 0,847481 | 0,808750 |
| PHerc0172 | 19 | — | — | — | — |
| PHerc0814 | 61 | 0,579574 | 0,775063 | 0,710944 | 0,603383 |
| PHerc1667 | 147 | 0,952739 | 0,976728 | 0,932689 | 0,985320 |
| PHercParis4 | 1.984 | 0,851141 | 0,857536 | 0,837505 | 0,844457 |

PHerc0172 tiene sólo negativos: conserva cobertura, pero AUROC no está definida. El macro usa cuatro rollos con igual peso. La peor regresión R2B−R1 es −0,171679 en PHerc0814.

![AUROC y average precision por rollo](figures/results_per_roll_metrics.png)

La AP y P/R@50/100/200 completas están en el [JSON de comparaciones](comparisons.json). PHerc1667 y PHercParis4 tienen prevalencias positivas de aproximadamente 90,5% y 95,4%; sus AP altas deben leerse junto a esas prevalencias. La [revisión independiente](results_independent_review.md) incluye denominadores, OLS y resultados detallados.

## Decisión y modelo exportado

R2B era la candidata antes de entrenar. Logró cobertura 15/15, pero falló las otras reglas: necesitaba mejorar R1 en al menos +0,01 y no retroceder más de −0,03 en ningún rollo; observamos −0,049259 y −0,171679, respectivamente.

Su estado queda **EXPERIMENTAL**. No se cambió de candidato ni de semilla tras consultar el test. R1 queda como una hipótesis prometedora para confirmar independientemente; sus 15 checkpoints permanecen disponibles.

El refit final partió otra vez de ImageNet V1, con R2B, semilla 0 y los 2.836 casos. La mediana de las 15 fracciones de pasos seleccionados por validación fue 2/3. Aplicada al presupuesto completo de 534 pasos produjo **356 pasos**, fijados antes del refit. Son 11.392 presentaciones de train. El refit no tiene evaluación fuera de muestra propia; sus métricas no deben confundirse con las de los modelos CV.

El paquete utilizable (checkpoint, scorer, preprocesado, metadata, dependencias y quick start) está en
https://huggingface.co/LimeGS/herculaneum-legibility-proxy-v5. El checkpoint pesa 44.782.603 bytes; SHA-256: `7179f40882a755d885d9eca88eb0a9b54c436264cd3bcd8c913054812294865c`. No incluye recortes ni datos de revisores.

## Velocidad y verificación

El perfil sintético favoreció BF16/channels-last: 2.882,59 presentaciones/s frente a 1.303,10 FP32 contiguous, con pérdidas y gradientes finitos. Esa aceleración de 2,21× es técnica, no una medida de calidad.

Los 45 fits reales tardaron **200,57 segundos**, completando 15.228 actualizaciones, 487.296 presentaciones de train y 5.841 predicciones de validación seleccionada. El pico de memoria CUDA asignada fue 540.990.976 bytes. El proceso del refit tardó **8,23 segundos**; su fit interno, 4,79 segundos. Estos tiempos excluyen preparación y transferencias. Se descargaron y verificaron unos 2 GB de checkpoints y 321 archivos en total.

La suite final pasó **66 tests en 6,30 segundos**. El evaluador verificó hashes, IDs, cobertura, pasos, finitud, selección por validación y emparejamiento de draws R2/R2B. El exportador vuelve a calcular el criterio de aceptación y presupuesto; verifica los 2.836 IDs, muestreo, pesos, estado ResNet18 estricto/finito y código realmente utilizado. La revisión independiente reconstruyó también diez ajustes OLS y F0 (2.395 scores cacheados + 11 completados), confirmando las cifras y el FAIL.

El [CLI exportado se probó en CPU](export_cpu_smoke.json) con dos crops históricos reales y su preprocesado exacto. Es una prueba funcional con datos usados en el refit, no nueva evidencia de precisión. Las métricas CNN anteriores usan CUDA BF16; el paquete usa CPU FP32, sin paridad numérica validada entre backends.

## Qué queda por demostrar

Esta comparación mide estas modificaciones sobre el archivo histórico. No prueba reconocimiento de letras individuales ni explica causalmente la degradación. Ruido de etiquetas, diferencias entre mapas o demasiado peso para grupos pequeños son hipótesis, no hallazgos confirmados.

El alcance sigue siendo `historical_unharmonized_diagnostic`: faltan máscaras transportadas físicamente, correspondencia crop/raster/geometría, grupos territoriales finos y revisiones humanas nuevas independientes. Todos los `physical_group` siguen desconocidos. Los 430 adicionales no son una muestra nueva certificada y los aproximadamente 10k scores archivados no se convirtieron en 10k casos admisibles.

No se dispone del CSV externo de pscamillo para replicar su comparación en mapas nativos de 9 µm. No se validó un umbral de descarte ni el riesgo de perder texto. La confirmación requiere mapas/territorios nuevos y criterios fijados antes de consultar sus etiquetas; estos tests ya abiertos no deben reutilizarse como prueba independiente tras ajustar recetas.

El repositorio publicado, el experimento anterior y la GPU reservada para otras tareas se conservaron sin cambios. La instancia nueva permanece disponible y los resultados están respaldados localmente.
