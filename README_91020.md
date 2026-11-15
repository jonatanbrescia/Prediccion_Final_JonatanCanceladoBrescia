# Labo 1 - Analista Jr - Grupo A (Universidad Austral)

Prediccion de BAJA+2 para el mes 202109.
Competencia Kaggle: `labo-1-virtual-2026-analista-jr`.

## Contenido del repositorio
- `Prueba_91020_noche.ipynb`: notebook completo (kernel R), con `MODO_PRUEBA <- FALSE`.
- `progreso.txt`: log de la corrida (carpeta `WF91020`).
- `README.md`: este archivo.

## Pipeline
1. **Lectura** del dataset `analistajr_competencia_2026.csv.gz`.
2. **Catastrophe analysis**: metodo `MachineLearning` (pone NA en las variables rotas de ciertos meses).
3. **Data drifting**: `deflacion` por IPC sobre las variables monetarias.
4. **Feature engineering**: `kmes`, `mpayroll_sobre_edad`, y para cada variable lag1, lag2, delta1 y delta2 (273 columnas).
5. **Variables de entrenamiento**: se excluyen `clase_ternaria`, `clase01`, `azar`, `numero_de_cliente` y `foto_mes` (270 variables).
6. **Training strategy**: training 201901-202105, validacion 202107, final train 201901-202107, futuro 202109. Clase positiva: BAJA+1 y BAJA+2.
7. **Grid search** de 36 combinaciones: `num_leaves` {128, 256, 512, 768} x `min_data_in_leaf` {512, 1024, 2048} x `feature_fraction` {0.5, 0.65, 0.8}. Los hiperparametros se aplican con `modifyList`.
   - Cada combinacion se entrena con early stopping por AUC.
   - Se elige la mejor por **ganancia suavizada en 202107** (ganancia acumulada con media movil de 400, BAJA+2 = +0.975, resto = -0.025), no por AUC.
   - Ganadora: `num_leaves=128`, `min_data_in_leaf=512`, `feature_fraction=0.65`, 85 iteraciones (gan_val 102.75).
8. **Entrenamiento final** con semillerio de 40 modelos. Semillas: primos entre 100000 y 1000000 elegidos con `set.seed(999917, "L'Ecuyer-CMRG")`.
9. **Prediccion** de 202109 como promedio de las probabilidades de los 40 modelos, ordenadas de mayor a menor.
10. **Envios a Kaggle** con cortes de 1200 a 2400 (cada 100) mas el corte sugerido por validacion (2252).

## Envio final elegido
- **Corte 1800** (`KA91020_s40_1800`, 87.06 en el publico).
- La validacion en 202107 sugirio dos cortes segun la corrida: 1767 (prueba rapida) y 2252 (corrida completa). Ambos son plausibles: el corte sugerido varia de 1573 a 2601 segun la combinacion del grid.
- Entre ambos se eligio el 1800 por tener la nota publica mas alta (87.06 contra 84.98 del 2252). La diferencia es menor que el ruido del publico (hasta 5 puntos entre cortes vecinos), por lo que no implica que 1800 sea mejor.

## Como reproducir
1. Dataset en `/content/datasets/analistajr_competencia_2026.csv.gz`.
2. Carpeta de trabajo `/content/buckets/b1/exp/` (se puede cambiar al inicio del notebook).
3. Credenciales de Kaggle configuradas, para los envios automaticos.
4. Abrir `Prueba_91020_noche.ipynb`, dejar `MODO_PRUEBA <- FALSE` y ejecutar todo. Duracion aproximada: 50 minutos
   (grid 35 min, semillerio 7 min, envios 7 min).
5. Con `MODO_PRUEBA <- TRUE` hace una prueba rapida (2 combinaciones, 2 modelos, sin Kaggle).

Nota: LightGBM con varios hilos puede cambiar los ultimos decimales entre maquinas. El ranking resultante deberia ser practicamente el mismo.

## Resultados (Kaggle, publico)
Media de los cortes 1200-2400: 83.8. Mejor corte: 1500 (87.98). Envio elegido, corte 1800: 87.06. Corte sugerido por validacion en la corrida completa (2252): 84.98.

## Hallazgos
- Se corrigio un bug del baseline: `c(fijos, mejores)` duplicaba nombres de parametros y LightGBM usaba el primero, ignorando el grid. Se reemplazo por `modifyList`.
- Las variantes probadas (con y sin ID, grids distintos, variables de tendencia) quedan entre 82 y 84 de media sobre los cortes: dentro del ruido del leaderboard publico. La variante con tendencias quedo por debajo del control.
- La nota publica varia hasta 5 puntos entre cortes vecinos del mismo modelo, por lo que no se eligio ni la semilla ni el corte por el pico publico.
- El AUC y la ganancia de un solo mes no distinguen entre combinaciones cercanas del grid (36 combinaciones entre 97 y 103 de ganancia de validacion).
- Un ensamble por ranking de 5 corridas rindio igual que la corrida individual, por lo que no se uso en la entrega.
