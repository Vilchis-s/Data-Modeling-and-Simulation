# 2.1 Qué es una serie de tiempo y sus componentes

En el Tema 1 se modeló el PIB postulando una ley, la ecuación logística, y dejando que los datos solo calibraran sus parámetros. El Tema 2 cambia de filosofía. Aquí no se supone ninguna ley del sistema: se dispone del histórico de una variable continua medida en el tiempo y se buscan patrones de dependencia temporal para comprenderla y proyectarla. Esta es la rama estadística de los sistemas continuos, y su objeto de estudio es la serie de tiempo.

## Definición operativa

Una serie de tiempo es una secuencia de observaciones de una variable ordenadas en el tiempo y, en general, equiespaciadas: el PIB anual de un país, el precio mensual del petróleo, las exportaciones trimestrales de un bloque. Lo que distingue a una serie de tiempo de un conjunto de datos cualquiera es que el orden importa y que las observaciones cercanas en el tiempo tienden a parecerse. Esa dependencia temporal es precisamente lo que los modelos de series de tiempo explotan.

La diferencia con el aprendizaje supervisado clásico es profunda. En un problema tabular pueden barajarse las filas sin pérdida de información. En una serie de tiempo, barajar destruye todo, pues la información reside justamente en la secuencia. Por ello, las técnicas, la validación e incluso las métricas son distintas.

## La descomposición clásica

La forma más útil de concebir una serie de tiempo consiste en descomponerla en componentes. El esquema clásico separa tres.

La tendencia es el movimiento de largo plazo, la dirección general. El PIB de BRICS+ presenta una tendencia ascendente clara de dos décadas.

La estacionalidad es el patrón que se repite con periodo fijo, típicamente intraanual: mayor consumo eléctrico en verano, mayores ventas en diciembre. En datos anuales de PIB la estacionalidad desaparece, pero en datos trimestrales o mensuales resulta central.

El residuo o componente irregular es lo que queda tras retirar tendencia y estacionalidad. Idealmente es ruido sin estructura. Si conserva estructura, el modelo está dejando información sin aprovechar.

Estos componentes se combinan de dos maneras. En el modelo aditivo la serie es la suma `tendencia + estacionalidad + residuo`, apropiado cuando la amplitud de las oscilaciones es estable. En el modelo multiplicativo es el producto, apropiado cuando las oscilaciones crecen con el nivel de la serie, que es lo habitual en variables económicas como el PIB, donde una recesión del 5 por ciento es mucho mayor en valor absoluto cuando la economía es grande.

## Carga de una serie real

Se trabaja con el crecimiento anual del PIB de China, una serie con tendencia, ciclos y choques bien visibles, idónea para ilustrar los componentes. Se utiliza el indicador de crecimiento porcentual del Banco Mundial.

```python
import wbgapi as wb
import pandas as pd

# NY.GDP.MKTP.KD.ZG = crecimiento anual del PIB en porcentaje
serie = wb.data.DataFrame("NY.GDP.MKTP.KD.ZG", "CHN", time=range(1980, 2023))
crecimiento = serie.iloc[0].sort_index()
crecimiento.index = pd.PeriodIndex([c.replace("YR", "") for c in crecimiento.index],
                                   freq="Y")
crecimiento = crecimiento.astype(float)
print(crecimiento.describe())
```

En esta serie se aprecian a simple vista la tendencia de largo plazo del crecimiento chino desacelerando desde tasas de dos dígitos, el choque de 1989-1990, la crisis financiera de 2008 y el desplome de la pandemia en 2020. Cada uno de esos eventos recuerda que las series económicas no son ruido puro: cargan historia política.

## Visualizar siempre primero

La primera regla de las series de tiempo es graficar antes de modelar. La inspección visual detecta tendencias, quiebres, valores atípicos y cambios de varianza que ninguna prueba estadística revela con la misma rapidez.

```python
import matplotlib.pyplot as plt

crecimiento.plot(marker="o")
plt.axhline(0, color="grey", lw=0.8)
plt.title("Crecimiento anual del PIB de China (%)")
plt.ylabel("Crecimiento (%)")
plt.show()
```

## Por qué los componentes guían el modelo

Identificar los componentes no es un ejercicio decorativo, sino que determina qué modelo se utilizará. Si la serie tiene tendencia, será necesario diferenciarla o modelar la tendencia de forma explícita. Si tiene estacionalidad, se requerirá un modelo estacional como SARIMA. Si la varianza crece con el nivel, se requerirá una transformación logarítmica. Toda la maquinaria de los próximos capítulos se organiza en torno a tratar cada componente como corresponde.

## Bibliografía

Una serie de tiempo es una variable continua observada en el tiempo, donde el orden y la dependencia temporal constituyen la información clave. Se descompone en tendencia, estacionalidad y residuo, en forma aditiva o multiplicativa. Esa descomposición orienta toda la modelación posterior. El primer obstáculo técnico que casi todos los modelos exigen resolver es la estacionariedad, concepto que se aborda en la siguiente página.

## Referencias

1. Hyndman, R. J., & Athanasopoulos, G. (2021). *Forecasting: Principles and practice* (3a ed.). OTexts. https://otexts.com/fpp3/
