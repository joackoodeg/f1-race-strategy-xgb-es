# F1 Optimization

Trabajo práctico integrador de Inteligencia Computacional. El proyecto optimiza la **estrategia de neumáticos y pit stops** de una carrera de Fórmula 1 combinando:

- Un **modelo predictivo (XGBoost)** entrenado con datos reales de carreras (vía [FastF1](https://github.com/theOehrly/Fast-F1)) que estima el tiempo de vuelta en función del compuesto de neumático, su desgaste y la carga de combustible.
- Un **algoritmo evolutivo (Estrategia Evolutiva con DEAP)** que busca, para una carrera dada, la secuencia de compuestos y pit stops que minimiza el tiempo total de carrera predicho por el modelo.

## Estructura del proyecto

```
f1_optimization/
├── ES.py                  # Algoritmo evolutivo (cruza desactivada por defecto)
├── ES_alternativo.py      # Variante del algoritmo evolutivo (cruza activada)
├── model.py                # Entrenamiento del modelo XGBoost de tiempos de vuelta
├── debug_model.py          # Script para inspeccionar predicciones/importancia de features
├── cruza_test.py            # Pruebas puntuales del operador de cruza (mate)
├── utils/
│   ├── ES/main.py           # Configuración de la carrera, carga de modelo y funciones de predicción
│   └── XGBoost/main.py      # Carga de datos de FastF1 y feature engineering para el modelo
├── test/
│   └── cruza_test.py
└── requirements.txt
```

## Requisitos

- Python 3.10+
- [`scoop`](https://github.com/soft-matter/scoop) para paralelizar la evaluación del algoritmo evolutivo (`pip install scoop`)
- Dependencias listadas en `requirements.txt`:

```bash
pip install -r requirements.txt
pip install deap scoop
```

## Uso

### 1. Entrenar el modelo de tiempos de vuelta

El año, circuito y años históricos a usar se configuran en `utils/XGBoost/main.py` (variables `year`, `gp`, `historical_years`).

```bash
python model.py
```

Esto descarga/cachea datos de carrera desde FastF1, construye las features de degradación de neumáticos y combustible, entrena un `XGBRegressor` y guarda el modelo en `utils/XGBoost/models/model_<año>_gp<circuito>.pkl`.

### 2. Optimizar la estrategia de carrera

El circuito, cantidad de vueltas y parámetros del algoritmo evolutivo (tamaño de población, generaciones, probabilidad de mutación, tiempo de pit stop) se configuran en `utils/ES/main.py`.

```bash
python -m scoop ES.py
```

o, para la variante con cruza activada:

```bash
python -m scoop ES_alternativo.py
```

El resultado (mejor estrategia encontrada) se imprime en consola y se guarda en `best_strategy_<año>_<gp>.txt`.

## Cómo funciona

1. **Modelo de tiempos de vuelta** (`model.py` + `utils/XGBoost/main.py`): se entrena con datos históricos de carreras del mismo circuito, usando como features el compuesto de neumático, la vida del neumático (`TyreLife`), la carga de combustible estimada y la temperatura de pista, para predecir el tiempo de vuelta.
2. **Representación de la estrategia** (`ES.py`): cada individuo es una lista de compuestos de neumático (uno por vuelta), junto con metadatos de pit stops y edad de neumático. Se valida que toda estrategia tenga al menos un pit stop y use al menos dos compuestos distintos.
3. **Función de fitness**: se recorre vuelta por vuelta prediciendo el tiempo con el modelo entrenado, sumando una penalización fija (`PIT_STOP_TIME`) por cada parada en boxes.
4. **Evolución**: DEAP ejecuta selección, mutación (y opcionalmente cruza) sobre la población durante varias generaciones hasta converger a la estrategia de menor tiempo total estimado.

## Notas

- Los modelos entrenados (`*.pkl`) y la caché de FastF1 no se versionan (ver `.gitignore`); hay que generarlos localmente con `model.py`.
- `ES.py` y `ES_alternativo.py` difieren principalmente en si el operador de cruza está activado (`ENABLE_CROSSOVER`).
