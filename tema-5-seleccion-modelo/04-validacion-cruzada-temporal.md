# 5.4 Validación cruzada temporal

El capítulo anterior comparó modelos sobre un solo corte temporal. Un corte único es frágil: el ganador podría serlo solo porque ese tramo final le fue favorable. La validación cruzada temporal corrige esto evaluando los modelos sobre muchos cortes sucesivos, lo que da una estimación mucho más confiable de cuál generaliza mejor. Es la versión rigurosa de la selección de modelo y el filtro final antes de comprometerme con uno.

## Por qué un solo corte no basta

Si elijo el modelo que ganó en el corte de 2017, estoy sobreajustando a ese tramo específico. Quizá 2018 a 2022 incluyó la pandemia, que penalizó a un modelo y no a otro por razones que no se repetirán. La solución es no apostar todo a un corte, sino promediar el desempeño sobre una secuencia de cortes que recorran distintos tramos de la historia. Si un modelo gana de forma consistente en la mayoría de los cortes, ese sí es un ganador creíble.

## La validación con origen móvil

La técnica estándar para series de tiempo es la validación con origen móvil, también llamada walk-forward. Avanzo un punto de origen a través de la serie; en cada posición entreno con todo lo anterior y predigo el siguiente paso, registrando el error. Al final tengo una distribución de errores por modelo, no un solo número.

```python
import numpy as np
import wbgapi as wb
import pandas as pd
from scipy.optimize import curve_fit
from statsmodels.tsa.arima.model import ARIMA

brics = ["BRA", "RUS", "IND", "CHN", "ZAF", "EGY", "ETH", "IRN", "ARE"]
df = wb.data.DataFrame("NY.GDP.MKTP.CD", brics, time=range(1995, 2023))
df.columns = [int(c.replace("YR", "")) for c in df.columns]
pib = (df.sum(axis=0) / 1e12).sort_index()
pib.index = pd.period_range("1995", "2022", freq="Y")

def logistica(tau, r, K, y0):
    return K / (1 + ((K - y0) / y0) * np.exp(-r * tau))

def walk_forward(serie, inicio=15):
    err_ode, err_arima = [], []
    for t in range(inicio, len(serie) - 1):
        train = serie.iloc[:t]
        real = serie.iloc[t]
        tau = np.arange(len(train))
        try:
            (r, K, y0), _ = curve_fit(logistica, tau, train.values,
                                      p0=[0.1, 80, train.values[0]], maxfev=10000)
            pred_ode = logistica([len(train)], r, K, y0)[0]
            err_ode.append(abs(real - pred_ode) / real)
        except Exception:
            pass
        try:
            m = ARIMA(np.log(train), order=(1, 1, 1)).fit()
            pred_arima = np.exp(m.forecast(1).iloc[0])
            err_arima.append(abs(real - pred_arima) / real)
        except Exception:
            pass
    return np.array(err_ode), np.array(err_arima)

e_ode, e_arima = walk_forward(pib)
print(f"ODE   : MAPE medio {e_ode.mean():.2%}  (n={len(e_ode)})")
print(f"ARIMA : MAPE medio {e_arima.mean():.2%}  (n={len(e_arima)})")
```

Ahora cada modelo tiene un MAPE promedio sobre muchos orígenes, mucho más estable que el del corte único. La cantidad de evaluaciones por modelo aparece como n, y reportarla es parte de la honestidad: pocos cortes significan un promedio ruidoso.

## Comparar con rigor estadístico, no solo a ojo

Que un modelo tenga menor MAPE promedio no prueba que sea mejor; la diferencia podría ser ruido. La guía del curso insiste en este punto al hablar de comparar escenarios: hay que ver si la diferencia es estadísticamente significativa y no solo numérica (Law, 2014). Como tengo errores apareados, el mismo origen evaluado por ambos modelos, uso una prueba apareada.

```python
from scipy import stats

# Emparejo solo los orígenes donde ambos modelos produjeron predicción
n = min(len(e_ode), len(e_arima))
diff = e_ode[:n] - e_arima[:n]
t_stat, p_val = stats.ttest_rel(e_ode[:n], e_arima[:n])
print(f"Diferencia media de MAPE: {diff.mean():.2%}")
print(f"Prueba apareada: t={t_stat:.2f}, p={p_val:.3f}")
if p_val < 0.05:
    print("La diferencia entre modelos es estadísticamente significativa.")
else:
    print("No hay evidencia de que un modelo sea mejor; elijo por parsimonia.")
```

Si la prueba no es significativa, la conclusión correcta no es elegir el del MAPE ligeramente menor, es elegir por otro criterio, normalmente la parsimonia o la interpretabilidad. Esto evita el autoengaño de leer una diferencia real donde solo hay azar muestral, que es exactamente el error que la guía advierte sobre intervalos que se superponen (Law, 2014).

## Una nota sobre pruebas más especializadas

Existen pruebas diseñadas específicamente para comparar la precisión de dos pronósticos, como la de Diebold-Mariano (1995), que afinan la comparación cuando los errores de pronóstico tienen autocorrelación. Las menciono solo para que sepas que existen y que en un trabajo muy especializado se usarían. Para el alcance de este libro, la prueba apareada que ya apliqué es suficiente y más fácil de interpretar: lo importante no es qué prueba uso, sino que no declare a un modelo ganador por una diferencia que podría ser azar.

## El veredicto se construye, no se declara

La selección final del modelo es la suma de varias evidencias, no un único número. Junto el ranking de AIC dentro de cada familia, el MAPE promedio de la validación con origen móvil, la significancia de las diferencias, y los criterios cualitativos de interpretabilidad e incertidumbre. Un modelo gana cuando es bueno en varias de estas dimensiones a la vez, no cuando arrasa en una sola. Esa es la diferencia entre seleccionar un modelo y enamorarse de uno.

## Cierre

La validación cruzada temporal con origen móvil reemplaza el corte único frágil por un promedio sobre muchos cortes, y la comparación se hace con una prueba estadística apareada para no confundir azar con superioridad real. Cuando la diferencia no es significativa, decide la parsimonia. El veredicto final integra AIC, MAPE de validación, significancia e interpretabilidad. Con esta metodología lista, el último capítulo del Tema 5 la aplica de principio a fin para seleccionar el modelo del PIB de los bloques.

## Referencias

Diebold, F. X., & Mariano, R. S. (1995). Comparing predictive accuracy. *Journal of Business & Economic Statistics, 13*(3), 253-263. https://doi.org/10.1080/07350015.1995.10524599

Law, A. M. (2014). *Simulation modeling and analysis* (5a ed.). McGraw-Hill.

Hyndman, R. J., & Athanasopoulos, G. (2021). *Forecasting: Principles and practice* (3a ed.). OTexts. https://otexts.com/fpp3/
