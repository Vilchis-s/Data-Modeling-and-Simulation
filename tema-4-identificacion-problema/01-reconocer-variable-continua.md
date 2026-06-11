# 4.1 Cómo reconocer una variable continua

Todo el bloque depende de una decisión que se toma al principio y que casi nadie examina con cuidado: la variable que voy a modelar es continua o discreta. Equivocarse aquí lleva a aplicar toda la maquinaria incorrecta. Este capítulo trata cómo reconocer una variable continua y, sobre todo, los casos límite donde la respuesta no es obvia.

## La definición y la prueba rápida

Una variable es continua cuando puede tomar cualquier valor dentro de un rango, incluyendo los infinitos valores intermedios entre dos puntos. Una variable es discreta cuando solo puede tomar valores separados, típicamente enteros, sin nada en medio.

La prueba mental que uso es preguntarme si tiene sentido un valor a medio camino entre dos observaciones. El PIB puede ser de 18.3 o de 18.4 billones, y también de 18.347: todos los intermedios tienen sentido. Es continua. El número de países en un bloque es 9 o 10, nunca 9.5: es discreta. La guía del curso ancla toda su rama de árbol de decisión en esta distinción, porque determina si vamos hacia ecuaciones diferenciales o hacia cadenas de Markov y eventos discretos (Law, 2014).

## Por qué importa tanto para la elección de modelo

La naturaleza de la variable selecciona familias enteras de técnicas. Una variable continua que evoluciona en el tiempo se modela con ODEs, series de tiempo o movimiento browniano geométrico. Una variable discreta de conteo se modela con procesos de Poisson o cadenas de Markov. Aplicar un ARIMA a un conteo pequeño, o una cadena de Markov a un precio continuo, no es un detalle menor: es usar la herramienta equivocada y obtener resultados que no significan nada.

## Los casos límite que confunden

La distinción parece clara hasta que aparecen los casos intermedios, que son los que de verdad hay que saber resolver.

El primer caso es la variable de conteo grande. El número de transacciones diarias en un sistema de pagos es técnicamente discreto, pero cuando son millones, tratarlo como continuo es una aproximación excelente y mucho más cómoda. La regla práctica es que un conteo con valores grandes y muchos niveles distintos se modela bien como continuo.

El segundo caso es la variable continua medida de forma discreta. El PIB es un flujo continuo, pero lo observo una vez al año. El proceso subyacente es continuo aunque mis datos sean una muestra discreta en el tiempo. Aquí la variable es continua; lo discreto es la frecuencia de medición, que es otra cosa.

El tercer caso es la proporción acotada. La participación de un bloque en el PIB mundial es continua, pero vive en el intervalo cero a uno, lo que exige cuidado, como vi con la transformación logit en el capítulo 3.4. Es continua, pero acotada, y eso cambia cómo la modelo.

## Diagnóstico con datos

Una forma empírica de orientarse es contar cuántos valores distintos toma la variable respecto al número de observaciones. Muchos valores únicos sugiere continua; pocos valores repetidos sugiere discreta o categórica.

```python
import wbgapi as wb
import pandas as pd

pib = wb.data.DataFrame("NY.GDP.MKTP.CD", "BRA", time=range(1980, 2023)).iloc[0]
pib = pib.dropna().astype(float)

n_unicos = pib.nunique()
n_total = len(pib)
print(f"Valores únicos: {n_unicos} de {n_total} observaciones")
print(f"Ratio de unicidad: {n_unicos / n_total:.2f}")
```

Para el PIB el ratio de unicidad es prácticamente 1, cada observación es un valor distinto, lo que confirma su carácter continuo. Para una variable como el número de miembros del bloque, el ratio sería bajísimo. No es una prueba formal, pero es un diagnóstico rápido y honesto que combino con el razonamiento sobre el significado de la variable.

## La variable de estudio del libro

Reúno el diagnóstico para nuestro caso. El PIB de un bloque es continuo: es un flujo monetario que puede tomar cualquier valor positivo, sus oscilaciones tienen valores intermedios con sentido, y empíricamente cada observación es única. Lo medimos de forma discreta, una vez al año, pero eso no lo vuelve discreto, solo limita la resolución temporal de los datos. Es exactamente el tipo de variable para el que el bloque de sistemas continuos fue diseñado, y por eso es el hilo conductor del libro.

## Cierre

Una variable es continua si admite valores intermedios con sentido, y esa propiedad selecciona la familia de modelos. Los casos límite, conteos grandes, variables continuas medidas de forma discreta y proporciones acotadas, son los que exigen criterio. El PIB de los bloques es inequívocamente continuo. Identificar la variable es el primer paso; el siguiente es formular el problema completo alrededor de ella, con las cinco dimensiones que trato en la próxima página.

## Referencias

Law, A. M. (2014). *Simulation modeling and analysis* (5a ed.). McGraw-Hill.

Wooldridge, J. M. (2019). *Introductory econometrics: A modern approach* (7a ed.). Cengage Learning.
