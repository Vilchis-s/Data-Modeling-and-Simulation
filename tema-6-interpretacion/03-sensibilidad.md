# 6.3 Análisis de sensibilidad: Morris y Sobol

Interpretar un modelo incluye saber de qué depende su respuesta. El análisis de sensibilidad responde la pregunta: cuáles parámetros mueven de verdad la conclusión, y cuáles puedo fijar sin que cambie nada. La guía del curso dedica una sección entera a esto y propone dos métodos complementarios, Morris para tamizar y Sobol para cuantificar (Law, 2014). Los aplico a la ODE logística del PIB.

## Por qué no basta variar un parámetro a la vez

La forma ingenua de análisis de sensibilidad es OAT, variar un parámetro a la vez dejando fijos los demás. La guía advierte sus defectos: solo mide la sensibilidad en un punto, no captura interacciones y no tiene base probabilística (Law, 2014). El problema de las interacciones es serio: el efecto de la tasa r sobre la proyección puede depender del valor del techo K, y OAT nunca lo vería porque fija K mientras mueve r. Por eso uso métodos globales, que exploran todo el espacio de parámetros a la vez.

## El método de Morris para tamizar

Morris es un método de tamizado, screening, eficiente para identificar qué parámetros importan antes de un análisis más costoso. Produce dos métricas por parámetro: mu estrella, que mide la importancia, la magnitud del efecto, y sigma, que mide la no linealidad y las interacciones (Law, 2014).

```python
import numpy as np
from SALib.sample import morris as morris_sample
from SALib.analyze import morris as morris_analyze

# La salida que analizo es el PIB proyectado a 10 años por la logística
def proyeccion_logistica(r, K, y0, t=10):
    return K / (1 + ((K - y0) / y0) * np.exp(-r * t))

problema = {
    "num_vars": 3,
    "names": ["r", "K", "y0"],
    "bounds": [[0.05, 0.12],     # rango plausible de la tasa
               [60, 120],         # rango plausible del techo
               [25, 35]],         # rango plausible del valor inicial
}

X = morris_sample.sample(problema, N=200)
Y = np.array([proyeccion_logistica(r, K, y0) for r, K, y0 in X])
Si = morris_analyze.analyze(problema, X, Y)

for nombre, mu, sigma in zip(problema["names"], Si["mu_star"], Si["sigma"]):
    print(f"{nombre}: mu*={mu:.2f}  sigma={sigma:.2f}")
```

La lectura sigue el diagrama de cuadrantes de la guía (Law, 2014). Un parámetro con mu estrella bajo no influye y se puede ignorar. Uno con mu estrella alto y sigma bajo influye de forma lineal, importante para calibrar. Uno con mu estrella alto y sigma alto influye de forma no lineal, con interacciones fuertes, y es el que hay que estimar con más cuidado. Para la logística del PIB, espero que el techo K y la tasa r dominen, y que sus interacciones no sean despreciables, porque la curva en S las acopla.

## Los índices de Sobol para cuantificar

Morris dice cuáles parámetros importan; Sobol dice exactamente cuánto. Los índices de Sobol descomponen la varianza de la salida y atribuyen a cada parámetro su fracción (Law, 2014). El índice de primer orden S1 mide el efecto individual aislado; el índice total ST incluye además todas las interacciones, por lo que ST siempre es mayor o igual que S1.

```python
from SALib.sample import sobol as sobol_sample
from SALib.analyze import sobol as sobol_analyze

X = sobol_sample.sample(problema, 1024)
Y = np.array([proyeccion_logistica(r, K, y0) for r, K, y0 in X])
Si = sobol_analyze.analyze(problema, Y)

for nombre, s1, st in zip(problema["names"], Si["S1"], Si["ST"]):
    print(f"{nombre}: S1={s1:.3f}  ST={st:.3f}")
```

La interpretación sigue las reglas de la guía (Law, 2014). Si S1 es cercano a ST, el parámetro actúa de forma casi lineal, con pocas interacciones. Si ST es mucho mayor que S1, el parámetro importa sobre todo por sus interacciones, su efecto depende de los otros. Si ST es menor que 0.01, el parámetro se puede fijar a su valor nominal sin perder nada. Si S1 es mayor que 0.10, el parámetro es crítico para la calibración y conviene gastar esfuerzo en estimarlo con datos de campo.

## Lo que el análisis de sensibilidad me dice del caso

El resultado típico para la logística del PIB es que el techo K domina la incertidumbre de la proyección a largo plazo, mientras que a corto plazo la tasa r pesa más. Esto tiene una consecuencia directa para la interpretación: la enorme incertidumbre sobre cuándo el bloque BRICS+ alcanza su techo viene casi toda de no saber dónde está ese techo, no de no saber la tasa. Si quisiera reducir la incertidumbre de mi proyección de transición, dónde invertir esfuerzo está claro: en acotar K, no en afinar r.

Esto conecta con el flujo de trabajo completo que propone la guía: tamizar con Morris, calibrar los parámetros críticos con datos, cuantificar la incertidumbre residual con Sobol, y recién entonces optimizar o decidir (Law, 2014). El análisis de sensibilidad no es un adorno final, es lo que me dice dónde está la ignorancia que más importa.

## Los tres tipos de incertidumbre

La guía distingue tres tipos de incertidumbre y aclara cuál ataca el análisis de sensibilidad (Law, 2014). La estocástica, la aleatoriedad del sistema, se reduce con más corridas. La de parámetros, no conocer los valores exactos, no se reduce con más corridas sino con mejores datos, y es la que aborda el análisis de sensibilidad. La de modelo, que el modelo no capture todo, no se reduce con ninguna de las dos y es la más peligrosa porque es invisible. Para el PIB, la incertidumbre de modelo incluye todo lo que la logística ignora: política, sanciones, crisis. Ningún Sobol la mide, y por eso la honestidad sobre los supuestos del Tema 4 es irremplazable.

## Cierre

El análisis de sensibilidad global, con Morris para tamizar y Sobol para cuantificar, identifica qué parámetros mueven la conclusión y cuáles se pueden fijar, capturando las interacciones que OAT pierde. Para el PIB revela que la incertidumbre del techo domina la proyección de largo plazo, indicando dónde invertir esfuerzo de datos. Distingue la incertidumbre de parámetros, que sí ataca, de la de modelo, que ningún método numérico resuelve. Con la incertidumbre entendida, el último capítulo del Tema 6 reúne todo en una lectura honesta de los resultados BRICS+ frente a G7.

## Referencias

Law, A. M. (2014). *Simulation modeling and analysis* (5a ed.). McGraw-Hill.

Saltelli, A., Ratto, M., Andres, T., Campolongo, F., Cariboni, J., Gatelli, D., Saisana, M., & Tarantola, S. (2008). *Global sensitivity analysis: The primer*. Wiley.

Herman, J., & Usher, W. (2017). SALib: An open-source Python library for sensitivity analysis. *Journal of Open Source Software, 2*(9), 97. https://doi.org/10.21105/joss.00097
