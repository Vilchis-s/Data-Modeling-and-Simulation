# 2.3 Autocorrelación: ACF y PACF

Una vez que tengo una serie estacionaria, la pregunta es qué memoria tiene: cuánto depende el valor de hoy de los valores pasados. Las dos herramientas que responden esto son la función de autocorrelación, ACF, y la función de autocorrelación parcial, PACF. Son los gráficos diagnósticos que me dicen qué orden de modelo proponer, y leerlos bien es buena parte del oficio de las series de tiempo.

## Autocorrelación: la correlación de la serie consigo misma

La autocorrelación con rezago k es la correlación entre la serie y una copia de sí misma desplazada k periodos. La ACF con rezago 1 mide qué tanto se parece el valor de hoy al de ayer; con rezago 2, al de antier, y así. Si la ACF es alta en rezagos pequeños y decae despacio, la serie tiene memoria larga. Si cae rápido a cero, la memoria es corta.

```python
import wbgapi as wb
import pandas as pd
import numpy as np
from statsmodels.graphics.tsaplots import plot_acf, plot_pacf
import matplotlib.pyplot as plt

pib = wb.data.DataFrame("NY.GDP.MKTP.CD", "IND", time=range(1970, 2023)).iloc[0]
pib = pib.sort_index().astype(float) / 1e12
pib.index = range(1970, 2023)

serie_est = np.log(pib).diff().dropna()   # log-diferencia: tasa de crecimiento

fig, (ax1, ax2) = plt.subplots(1, 2, figsize=(12, 4))
plot_acf(serie_est, lags=15, ax=ax1)
plot_pacf(serie_est, lags=15, ax=ax2, method="ywm")
plt.show()
```

Trabajo sobre la log-diferencia del PIB de India, que es aproximadamente su tasa de crecimiento y es estacionaria, porque la ACF y la PACF solo son interpretables sobre series estacionarias. Aplicarlas a una serie con tendencia da una ACF que decae lentísimo y no informa nada útil sobre la estructura.

## Autocorrelación parcial: aislando el efecto directo

La PACF con rezago k mide la correlación entre el valor de hoy y el de hace k periodos, pero descontando el efecto de los rezagos intermedios. La diferencia con la ACF es clave. Si hoy depende de ayer, y ayer dependía de antier, entonces hoy y antier estarán correlacionados de forma indirecta, a través de ayer. La ACF captura esa correlación indirecta; la PACF la elimina y deja solo la dependencia directa con cada rezago.

## La regla de lectura clásica

La razón por la que estos dos gráficos importan tanto es que su forma identifica el tipo de modelo. La regla de Box y Jenkins es la columna vertebral de la identificación clásica:

```
Proceso       ACF                          PACF
AR(p)         decae gradual                corta de golpe tras el rezago p
MA(q)         corta de golpe tras q        decae gradual
ARMA(p,q)     decae gradual                decae gradual
```

En palabras: si la PACF se corta de golpe después de cierto rezago y la ACF decae suave, estoy ante un proceso autorregresivo y ese rezago de corte es el orden p. Si es al revés, la ACF se corta y la PACF decae, es un proceso de medias móviles y el rezago de corte de la ACF es el orden q. Si ambas decaen suave, es una mezcla ARMA. Estos conceptos, AR y MA, los desarrollo en el capítulo 2.5; aquí lo que importa es que la lectura de los gráficos es la que me los sugiere.

## Las bandas de significancia

Los gráficos de `statsmodels` traen una banda azul de confianza. Una barra que cae dentro de la banda no es estadísticamente distinta de cero: no hay evidencia de autocorrelación real en ese rezago. Solo me fijo en las barras que sobresalen de la banda. Esto evita el error de leer estructura donde solo hay ruido muestral, algo especialmente importante con series cortas como el PIB anual, donde hay pocas observaciones y el azar genera correlaciones espurias con facilidad.

## La prueba de Ljung-Box como complemento

La inspección visual se complementa con la prueba de Ljung-Box, que contrasta de forma conjunta si un grupo de autocorrelaciones es cero. La uso sobre todo al final, sobre los residuos del modelo: si los residuos pasan Ljung-Box, no les queda autocorrelación y el modelo capturó toda la estructura temporal disponible.

```python
from statsmodels.stats.diagnostic import acorr_ljungbox

lb = acorr_ljungbox(serie_est, lags=[5, 10], return_df=True)
print(lb)
```

Un p-valor alto en Ljung-Box sobre los residuos es una buena noticia: significa que no rechazo la hipótesis de que no hay autocorrelación residual.

## El límite de la lectura clásica en series cortas

Voy a ser honesto sobre una limitación que vivo con datos económicos. El PIB anual de un país tiene a lo sumo unas cinco o seis décadas de observaciones, y eso es poco para que la ACF y la PACF muestren patrones nítidos. En la práctica, la lectura visual me da una o dos propuestas razonables de orden, y la decisión final la tomo comparando modelos con criterios de información, AIC y BIC, que veo en el Tema 5. La lectura clásica propone; los criterios de información disponen.

## Cierre

La ACF mide la memoria total de la serie y la PACF la memoria directa rezago a rezago. Su forma combinada, según la regla de Box y Jenkins, sugiere si el proceso es AR, MA o ARMA y de qué orden. Las bandas de confianza y la prueba de Ljung-Box me protegen de leer ruido como estructura. Con esta capacidad de diagnóstico, antes de saltar a los modelos paramétricos conviene ver las técnicas más directas de descomposición y suavizamiento, que es el tema siguiente.

## Referencias

Box, G. E. P., Jenkins, G. M., Reinsel, G. C., & Ljung, G. M. (2015). *Time series analysis: Forecasting and control* (5a ed.). Wiley.

Ljung, G. M., & Box, G. E. P. (1978). On a measure of lack of fit in time series models. *Biometrika, 65*(2), 297-303. https://doi.org/10.1093/biomet/65.2.297

Hyndman, R. J., & Athanasopoulos, G. (2021). *Forecasting: Principles and practice* (3a ed.). OTexts. https://otexts.com/fpp3/
