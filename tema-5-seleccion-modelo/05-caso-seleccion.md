# 5.5 Caso: selección del modelo para el PIB

El Tema 5 cierra ejecutando la selección completa para el PIB de BRICS+, integrando todo lo anterior: el árbol de decisión, los criterios de información, la comparación entre familias y la validación cruzada temporal. El objetivo es llegar a un veredicto defendible sobre qué modelo usar, con la evidencia ordenada, tal como se entregaría en un proyecto real.

## El procedimiento de selección, paso a paso

```python
import wbgapi as wb
import numpy as np
import pandas as pd
from scipy.optimize import curve_fit
from statsmodels.tsa.arima.model import ARIMA
import pmdarima as pm

brics = ["BRA", "RUS", "IND", "CHN", "ZAF", "EGY", "ETH", "IRN", "ARE"]
df = wb.data.DataFrame("NY.GDP.MKTP.CD", brics, time=range(1995, 2023))
df.columns = [int(c.replace("YR", "")) for c in df.columns]
pib = (df.sum(axis=0) / 1e12).sort_index()
pib.index = pd.period_range("1995", "2022", freq="Y")
log_pib = np.log(pib)
```

### Paso 1. Preselección de orden ARIMA por AIC

```python
auto = pm.auto_arima(log_pib, seasonal=False, information_criterion="aic",
                     stepwise=True, suppress_warnings=True)
orden_arima = auto.order
print(f"Orden ARIMA preseleccionado por AIC: {orden_arima}")
```

### Paso 2. Validación con origen móvil de los candidatos

```python
def logistica(tau, r, K, y0):
    return K / (1 + ((K - y0) / y0) * np.exp(-r * tau))

def mape_walk_forward(serie, modelo, orden=None, inicio=15):
    errores = []
    for t in range(inicio, len(serie) - 1):
        train = serie.iloc[:t]
        real = serie.iloc[t]
        try:
            if modelo == "ODE":
                tau = np.arange(len(train))
                (r, K, y0), _ = curve_fit(logistica, tau, train.values,
                                          p0=[0.1, 80, train.values[0]], maxfev=10000)
                pred = logistica([len(train)], r, K, y0)[0]
            elif modelo == "ARIMA":
                m = ARIMA(np.log(train), order=orden).fit()
                pred = np.exp(m.forecast(1).iloc[0])
            elif modelo == "naive":
                pred = train.iloc[-1] * (train.iloc[-1] / train.iloc[-2])
            errores.append(abs(real - pred) / real)
        except Exception:
            pass
    return np.array(errores)

res = {
    "ODE": mape_walk_forward(pib, "ODE"),
    "ARIMA": mape_walk_forward(pib, "ARIMA", orden=orden_arima),
    "naive_drift": mape_walk_forward(pib, "naive"),
}
for k, v in res.items():
    print(f"{k:12s}: MAPE medio {v.mean():.2%}")
```

Se incluye deliberadamente la línea base ingenua con deriva, pues cualquier modelo que no la supere no merece su complejidad, según la regla del capítulo 2.8 (tema-2-series-de-tiempo/08-validacion-metricas.md). El modelo ingenuo es el estándar a superar, no un relleno.

### Paso 3. Significancia de las diferencias

```python
from scipy import stats

n = min(len(res["ODE"]), len(res["ARIMA"]))
t_stat, p_val = stats.ttest_rel(res["ODE"][:n], res["ARIMA"][:n])
print(f"ODE vs ARIMA: diferencia {res['ODE'][:n].mean()-res['ARIMA'][:n].mean():.2%}, p={p_val:.3f}")
```

## La matriz de decisión

Se reúne toda la evidencia en una matriz, pues la selección no se decide por una celda, sino por el patrón completo.

```
Criterio              ODE logística      ARIMA              Ingenuo
MAPE walk-forward     medio              menor (mejor)      mayor (peor)
Da intervalos          no                 sí                 no
Interpretabilidad      alta (r, K)        baja               nula
Parámetros             3                  pocos              0
Robustez con n corto   alta (teoría)      media              alta
Maneja choques         no                 parcial            no
```

## El veredicto razonado

La decisión para este caso, defendible con la evidencia, es usar el ARIMA como modelo principal de pronóstico y conservar la ODE logística como modelo complementario de interpretación. La justificación es que cada uno gana en lo que la decisión del Tema 4 valora.

El ARIMA gana en lo que la decisión de cartera necesita sobre todo: pronóstico con intervalos de predicción, imprescindibles para cuantificar el riesgo de la asignación. Si su MAPE de validación supera al ingenuo de forma significativa, su complejidad queda justificada.

La ODE logística gana en interpretabilidad, que la decisión también valora para comunicar el mecanismo al comité: dos parámetros, tasa intrínseca y techo, cuentan la historia de la transición de un modo que un comité comprende, mientras que los coeficientes de un ARIMA no dicen nada a un no especialista. Además, su anclaje teórico la hace robusta pese a la serie corta.

Usar ambos no es indecisión, sino reconocer que predecir y comprender son objetivos distintos que el árbol de decisión ya había separado, y que el caso requiere ambos. El ARIMA responde hacia dónde y con cuánta incertidumbre; la ODE responde por qué.

## Lo que se descarta y por qué

Se descarta el GBM como modelo principal pese a su elegancia, pues para una serie anual tan corta la estimación de la volatilidad es ruidosa y sus intervalos quedan mal calibrados; se reserva para el Tema 7, donde su capacidad de generar escenarios sí aporta. Se descartan modelos de aprendizaje automático complejos, boosting o redes, pues con dos décadas de datos anuales sobreajustarían sin remedio, violando la parsimonia. La guía de estudio es clara: cuando los datos escasean, los modelos basados en teoría y los parsimoniosos ganan.

## Bibliografía del Tema 5

La selección del mejor modelo no fue elegir un ganador único, sino construir un veredicto con evidencia ordenada: preselección por AIC, validación con origen móvil contra una línea base ingenua, prueba de significancia y una matriz de decisión que pondera precisión, incertidumbre, interpretabilidad y robustez. El resultado para el PIB de los bloques es un dúo, ARIMA para predecir y ODE para comprender, justificado por la decisión que el modelo debe informar. Con el modelo elegido, el Tema 6 aborda lo que con frecuencia se hace mal: interpretar correctamente lo que el modelo indica.

## Referencias

1. Banco Mundial. (2024). *World Development Indicators* [Conjunto de datos]. https://databank.worldbank.org/source/world-development-indicators
2. Hyndman, R. J., & Athanasopoulos, G. (2021). *Forecasting: Principles and practice* (3a ed.). OTexts. https://otexts.com/fpp3/
