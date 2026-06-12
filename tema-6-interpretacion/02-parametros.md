# 6.2 Interpretación de los parámetros del modelo

Un pronóstico indica hacia dónde se dirige la variable. Los parámetros indican por qué. Interpretar los parámetros es donde el modelo deja de ser una caja que produce números y se convierte en una herramienta para comprender el fenómeno. Este capítulo lee los parámetros de los modelos del PIB y muestra cómo cada uno carga una historia económica, y a veces política.

## Los parámetros de la ODE logística

La logística del Tema 1 tiene dos parámetros con significado directo, y esa es su gran virtud frente al ARIMA.

La tasa intrínseca r es el ritmo de crecimiento del bloque cuando está lejos de su techo. Un r alto significa un motor de crecimiento potente. Para el bloque BRICS+, un r del orden del 8 al 10 por ciento refleja la fase de despegue de economías que todavía tienen amplio margen de convergencia tecnológica.

La capacidad de carga K es el techo estructural hacia el que tiende el producto del bloque. Es el parámetro más cargado de interpretación y el más delicado, pues cuando el sistema aún no muestra saturación, K se estima con enorme incertidumbre.

```python
import wbgapi as wb
import numpy as np
import pandas as pd
from scipy.optimize import curve_fit

brics = ["BRA", "RUS", "IND", "CHN", "ZAF", "EGY", "ETH", "IRN", "ARE"]
df = wb.data.DataFrame("NY.GDP.MKTP.CD", brics, time=range(1995, 2023))
df.columns = [int(c.replace("YR", "")) for c in df.columns]
pib = (df.sum(axis=0) / 1e12).sort_index()

tau = np.arange(len(pib))
def logistica(tau, r, K, y0):
    return K / (1 + ((K - y0) / y0) * np.exp(-r * tau))

popt, pcov = curve_fit(logistica, tau, pib.values,
                       p0=[0.1, 80, pib.values[0]], maxfev=10000)
r, K, y0 = popt
# curve_fit, además del valor de cada parámetro, devuelve una estimación de
# cuánta incertidumbre tiene cada uno; np.sqrt(np.diag(pcov)) la extrae
errores = np.sqrt(np.diag(pcov))

print(f"r = {r:.3f} ± {errores[0]:.3f}")
print(f"K = {K:.1f} ± {errores[1]:.1f} billones USD")
print(f"Fracción del techo alcanzada hoy: {pib.values[-1]/K:.1%}")
```

Además del valor de cada parámetro, `curve_fit` entrega una medida de cuán incierto es ese valor, y esa incertidumbre es parte de la interpretación. Si el error de K es enorme comparado con K, eso indica que el modelo no puede determinar el techo con los datos disponibles, lo cual es información valiosa: significa que el bloque está tan lejos de saturar que la curva todavía no revela dónde se aplanará. Reportar K sin su error sería fingir una precisión inexistente.

## Los parámetros del ARIMA

El ARIMA tiene parámetros menos interpretables, pero no opacos. El coeficiente autorregresivo phi mide la persistencia: cuánto un año arrastra al siguiente. El coeficiente de medias móviles theta mide cómo se disipa un choque. Para una serie de crecimiento, un phi positivo y significativo indica que el crecimiento tiene inercia, que los buenos años se agrupan, lo que en lenguaje de la guía de estudio constituye un lazo reforzador suave operando en el corto plazo.

```python
from statsmodels.tsa.arima.model import ARIMA
m = ARIMA(np.log(pib), order=(1, 1, 1)).fit()
print(m.summary().tables[1])
```

La tabla de coeficientes contiene el valor, el error estándar y el p-valor de cada parámetro. Un parámetro con p-valor alto no es estadísticamente distinto de cero, lo que significa que el modelo no necesita ese término y podría simplificarse, conectando de nuevo con la parsimonia del Tema 5.

## La significancia no es la importancia

Conviene separar siempre estos dos conceptos, pues la guía de estudio los distingue al tratar el tamaño de efecto frente a la significancia. Que un parámetro sea estadísticamente significativo no significa que sea importante en la práctica. Un coeficiente puede ser significativo, es decir, distinto de cero con alta confianza, y aun así tener un efecto tan pequeño que no cambie ninguna decisión. La pregunta de la significancia es si existe el efecto; la pregunta de la importancia es si importa el efecto. Ambas deben responderse, y solo la segunda decide.

## La elasticidad como interpretación de negocio

Para traducir un parámetro a lenguaje de negocio se utilizan elasticidades cuando es posible. Como el ARIMA se ajustó sobre el logaritmo del PIB, las diferencias del modelo se interpretan aproximadamente como tasas de crecimiento, lo que vuelve los coeficientes legibles en términos porcentuales. Esa es una de las razones para modelar en escala logarítmica: además de estabilizar la varianza, hace que los parámetros hablen el idioma del crecimiento porcentual que un economista o un comité comprende sin traducción.

## La advertencia sobre sobreinterpretar

Conviene cerrar con una advertencia. Resulta tentador construir un relato económico elaborado a partir de cada coeficiente, sobre todo cuando el tema es tan sugerente como la transición de poder entre bloques. Pero los parámetros de un modelo estimado sobre dos décadas de datos anuales son frágiles: cambian si se agrega o quita un país, si se extiende la muestra un par de años, o si se cambia la unidad de medida. La interpretación honesta de un parámetro siempre viene acompañada de su error estándar y de la pregunta de cuán robusto es a esas decisiones. Esa robustez es justamente lo que mide el análisis de sensibilidad de la siguiente página.

## Bibliografía

Los parámetros cuentan el porqué del fenómeno: r y K de la logística dan motor y techo, phi y theta del ARIMA dan persistencia y disipación de choques. Cada uno se reporta con su error estándar, y la significancia estadística no se confunde con la importancia práctica. Modelar en escala logarítmica hace que los parámetros hablen en porcentajes. La fragilidad de los parámetros frente a las decisiones de modelado se cuantifica con el análisis de sensibilidad, que se desarrolla a continuación.

