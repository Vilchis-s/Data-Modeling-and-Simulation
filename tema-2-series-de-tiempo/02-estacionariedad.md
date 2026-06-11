# 2.2 Estacionariedad y pruebas ADF y KPSS

La estacionariedad es el concepto más importante y, al principio, el más resbaladizo de las series de tiempo. Casi todos los modelos clásicos suponen que la serie es estacionaria, y casi ninguna serie económica real lo es en su forma cruda. Entender qué es, cómo se diagnostica y cómo se consigue es el cuello de botella de todo el Tema 2.

## Qué significa estacionario

Una serie es estacionaria, en el sentido débil que usamos en la práctica, cuando sus propiedades estadísticas no cambian con el tiempo. En concreto, tres condiciones:

La media es constante, la serie no tiene tendencia ni hacia arriba ni hacia abajo.

La varianza es constante, la amplitud de las oscilaciones no crece ni se encoge con el tiempo.

La autocovarianza depende solo de la distancia entre observaciones, no del momento en que ocurren. La relación entre un punto y el de hace dos periodos es la misma en 1990 que en 2020.

La razón por la que esto importa es que un modelo solo puede aprender de un patrón si el patrón se repite. Si la media se mueve, el modelo persigue un blanco móvil y no puede estimar nada estable. El PIB en niveles es el ejemplo perfecto de serie no estacionaria: tiene tendencia ascendente y varianza creciente. No se puede modelar directamente con ARMA.

## El diagnóstico visual

Antes de cualquier prueba, miro la serie. Una serie con tendencia evidente no es estacionaria, punto. Una serie cuyas oscilaciones se ensanchan con el tiempo tiene varianza no constante. El ojo resuelve la mayoría de los casos. Las pruebas formales sirven para los casos dudosos y para documentar la decisión con rigor.

## La prueba ADF

La prueba de Dickey-Fuller aumentada, o ADF, es la más usada. Su hipótesis nula es que la serie tiene una raíz unitaria, es decir, que es no estacionaria. Si el p-valor es pequeño, rechazo la nula y concluyo que la serie es estacionaria.

```python
from statsmodels.tsa.stattools import adfuller

def reporte_adf(serie, nombre=""):
    stat, pvalor, *_ = adfuller(serie.dropna())
    veredicto = "estacionaria" if pvalor < 0.05 else "NO estacionaria"
    print(f"ADF {nombre}: estadístico={stat:.3f}  p={pvalor:.4f}  -> {veredicto}")
    return pvalor

import wbgapi as wb
import numpy as np
import pandas as pd

pib = wb.data.DataFrame("NY.GDP.MKTP.CD", "CHN", time=range(1980, 2023)).iloc[0]
pib = pib.sort_index().astype(float) / 1e12
pib.index = range(1980, 2023)

reporte_adf(pib, "PIB en niveles")
reporte_adf(pib.diff(), "PIB diferenciado una vez")
```

El PIB en niveles casi siempre da no estacionario, con un p-valor alto. Tras diferenciar una vez, suele volverse estacionario. Esa diferenciación es la operación central que conecta con el modelo ARIMA del capítulo 2.6.

## La prueba KPSS y por qué uso las dos juntas

La prueba KPSS tiene la hipótesis nula opuesta: su nula es que la serie es estacionaria. Esto la hace complementaria de ADF, y usarlas juntas evita conclusiones apresuradas.

```python
from statsmodels.tsa.stattools import kpss

def reporte_kpss(serie, nombre=""):
    stat, pvalor, *_ = kpss(serie.dropna(), regression="c", nlags="auto")
    veredicto = "estacionaria" if pvalor > 0.05 else "NO estacionaria"
    print(f"KPSS {nombre}: estadístico={stat:.3f}  p={pvalor:.4f}  -> {veredicto}")
    return pvalor
```

La forma correcta de leerlas en conjunto es esta:

```
ADF rechaza (p<0.05)  y  KPSS no rechaza (p>0.05)  -> estacionaria, caso claro
ADF no rechaza        y  KPSS rechaza              -> no estacionaria, caso claro
Ambas rechazan o ninguna rechaza                   -> ambiguo, investigar más
```

Cuando las dos pruebas coinciden tengo una conclusión sólida. Cuando se contradicen, suele indicar que la serie tiene una estructura más complicada, por ejemplo una tendencia determinista en lugar de una raíz unitaria, y conviene mirar con más cuidado antes de elegir la transformación.

## Estacionariedad en media y en varianza son problemas distintos

Un punto que confunde al principio: diferenciar arregla la no estacionariedad en media, la tendencia, pero no la no estacionariedad en varianza. Si las oscilaciones del PIB crecen con su nivel, ningún número de diferenciaciones lo resuelve. Eso se arregla con una transformación logarítmica antes de diferenciar, tema que desarrollo en el capítulo 2.7. El orden correcto es: primero estabilizo la varianza con logaritmo, después estabilizo la media con diferenciación.

## Cierre

La estacionariedad exige media, varianza y autocovarianza estables en el tiempo. Es el supuesto que casi todos los modelos clásicos necesitan y que casi ninguna serie económica cumple en crudo. Se diagnostica con la vista y se confirma con ADF y KPSS leídas en conjunto. Las series no estacionarias se vuelven estacionarias diferenciando y, si hace falta, transformando. Una vez que tengo una serie estacionaria, la siguiente pregunta es qué estructura de dependencia temporal tiene, y eso se lee en la autocorrelación, el tema de la próxima página.

## Referencias

Dickey, D. A., & Fuller, W. A. (1979). Distribution of the estimators for autoregressive time series with a unit root. *Journal of the American Statistical Association, 74*(366), 427-431. https://doi.org/10.2307/2286348

Kwiatkowski, D., Phillips, P. C. B., Schmidt, P., & Shin, Y. (1992). Testing the null hypothesis of stationarity against the alternative of a unit root. *Journal of Econometrics, 54*(1-3), 159-178. https://doi.org/10.1016/0304-4076(92)90104-Y

Hyndman, R. J., & Athanasopoulos, G. (2021). *Forecasting: Principles and practice* (3a ed.). OTexts. https://otexts.com/fpp3/
