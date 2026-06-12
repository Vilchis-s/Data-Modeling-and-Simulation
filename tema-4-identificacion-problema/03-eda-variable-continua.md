# 4.3 EDA de una variable continua

El análisis exploratorio de datos, o EDA, es la etapa 2 de la metodología del curso y la que más se omite por prisa. Es un error costoso. Antes de ajustar cualquier modelo es necesario conocer la forma, la dispersión, los valores atípicos y la estructura temporal de la variable, pues cada una de esas características determina algún aspecto del modelado posterior. Este capítulo realiza el EDA completo de la variable de estudio.

## Los cuatro momentos como brújula

La guía de estudio propone leer una variable a través de sus cuatro momentos: media, coeficiente de variación, asimetría y curtosis, y usarlos como guía para comprender su forma. Se calculan para el crecimiento anual del PIB de los BRICS+.

```python
import wbgapi as wb
import numpy as np
import pandas as pd
from scipy import stats

brics = ["BRA", "RUS", "IND", "CHN", "ZAF", "EGY", "ETH", "IRN", "ARE"]
df = wb.data.DataFrame("NY.GDP.MKTP.CD", brics, time=range(1995, 2023))
df.columns = [int(c.replace("YR", "")) for c in df.columns]
pib = (df.sum(axis=0) / 1e12).sort_index()
crecimiento = pib.pct_change().dropna()

print(f"Media: {crecimiento.mean():.3f}")
print(f"CV: {crecimiento.std() / crecimiento.mean():.3f}")
print(f"Asimetría: {stats.skew(crecimiento):.3f}")
print(f"Curtosis exceso: {stats.kurtosis(crecimiento):.3f}")
```

La media proporciona el ritmo típico de crecimiento del bloque. El coeficiente de variación indica cuán volátil es ese crecimiento. La asimetría revela si los choques son más frecuentes hacia abajo, las crisis, que hacia arriba, los auges; en series económicas suele haber asimetría negativa, pues las recesiones son abruptas y las expansiones graduales. La curtosis indica si hay colas pesadas, es decir, eventos extremos más frecuentes de lo que una normal predeciría, lo cual es típico de variables financieras y económicas sujetas a crisis.

## La visualización mínima obligatoria

Hay cuatro gráficos que no deben omitirse con una serie continua.

```python
import matplotlib.pyplot as plt

fig, axes = plt.subplots(2, 2, figsize=(12, 8))

axes[0, 0].plot(pib.index, pib.values, marker="o")
axes[0, 0].set_title("Serie en niveles: tendencia y choques")

axes[0, 1].plot(crecimiento.index, crecimiento.values, marker="o")
axes[0, 1].axhline(0, color="grey", lw=0.8)
axes[0, 1].set_title("Tasa de crecimiento: volatilidad")

axes[1, 0].hist(crecimiento.values, bins=10, edgecolor="black")
axes[1, 0].set_title("Distribución: forma y asimetría")

stats.probplot(crecimiento.values, dist="norm", plot=axes[1, 1])
axes[1, 1].set_title("Q-Q normal: colas")

plt.tight_layout()
plt.show()
```

La serie en niveles muestra la tendencia y los choques. La tasa de crecimiento muestra la volatilidad y los años negativos. El histograma muestra la forma de la distribución. El gráfico Q-Q frente a la normal revela si las colas son más pesadas que las de una normal, lo que se aprecia cuando los puntos de los extremos se despegan de la línea recta. Estos cuatro en conjunto informan casi todo lo necesario antes de modelar.

## Detección de valores atípicos con sentido económico

En datos económicos, un valor atípico rara vez es un error de medición: suele ser un evento real e importante. La crisis de 2008 y la pandemia de 2020 aparecen como atípicos, pero eliminarlos falsificaría la historia. El EDA no busca eliminar atípicos, sino comprenderlos y decidir cómo tratarlos en el modelo.

```python
z = (crecimiento - crecimiento.mean()) / crecimiento.std()
atipicos = crecimiento[abs(z) > 2]
print("Años atípicos en el crecimiento del bloque:")
print(atipicos.round(3))
```

Cada año que resulta atípico tiene un nombre y una causa: una crisis financiera, una pandemia, un colapso de precios de commodities. La decisión de modelado no es removerlos, sino elegir un modelo que los acomode, por ejemplo mediante una transformación que reduzca su influencia o un modelo robusto, y reportar que los choques son parte estructural del fenómeno.

## Estructura temporal: lo que el EDA tabular no ve

El EDA de una serie tiene una dimensión que el EDA tabular ignora: la dependencia temporal. Aquí se adelanta la ACF del capítulo 2.3 (tema-2-series-de-tiempo/03-acf-pacf.md) como parte del EDA, pues observar si la serie tiene memoria es tan parte de conocerla como observar su histograma.

```python
from statsmodels.graphics.tsaplots import plot_acf
plot_acf(crecimiento, lags=10)
plt.show()
```

Si la tasa de crecimiento muestra autocorrelación, significa que un año bueno tiende a seguir a otro bueno, una persistencia que el modelo deberá capturar. Si no la muestra, el crecimiento es casi impredecible de un año al siguiente, lo que limitaría de antemano cualquier pretensión de pronóstico fino.

## El EDA cambia la formulación del problema

Un punto observado en proyectos reales: el EDA con frecuencia obliga a regresar a la etapa de formulación. Si en el EDA se descubre que la varianza crece con el nivel, se sabe que se requerirá transformación logarítmica. Si se descubren colas pesadas, se sabe que los intervalos basados en normalidad subestimarán el riesgo. Si se descubren pocos datos comparables para el bloque ampliado, se sabe que debe moderarse el horizonte. El EDA no es un trámite previo al modelo, sino un diálogo con el problema que a veces lo redefine, lo cual es justamente la iteración que describe la metodología.

## Bibliografía

El EDA de una variable continua lee sus cuatro momentos, la visualiza en niveles, en tasa, en distribución y frente a la normal, comprende sus atípicos como eventos reales en lugar de eliminarlos, y examina su estructura temporal. Realizado con seriedad, el EDA orienta cada decisión posterior de modelado y a veces redefine el problema. Con el problema formulado y los datos comprendidos, el siguiente capítulo cierra el Tema 4 redactando la definición formal del problema del PIB como documento.

## Referencias

1. Hyndman, R. J., & Athanasopoulos, G. (2021). *Forecasting: Principles and practice* (3a ed.). OTexts. https://otexts.com/fpp3/
