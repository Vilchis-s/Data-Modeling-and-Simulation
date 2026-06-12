# 6.3 Análisis de sensibilidad: Morris y Sobol

Interpretar un modelo incluye conocer de qué depende su respuesta. El análisis de sensibilidad responde la pregunta: cuáles parámetros mueven de verdad la conclusión y cuáles pueden fijarse sin que cambie nada. La guía de estudio dedica una sección entera a esto y propone dos métodos complementarios, Morris para tamizar y Sobol para cuantificar. Se aplican a la ODE logística del PIB.

## Por qué no basta variar un parámetro a la vez

La forma ingenua de análisis de sensibilidad es OAT, variar un parámetro a la vez dejando fijos los demás. La guía de estudio advierte sus defectos: solo mide la sensibilidad en un punto, no captura interacciones y carece de base probabilística. El problema de las interacciones es serio: el efecto de la tasa r sobre la proyección puede depender del valor del techo K, y OAT nunca lo vería, pues fija K mientras mueve r. Por ello se utilizan métodos globales, que exploran todo el espacio de parámetros a la vez.

## El método de Morris para tamizar

Morris es un método de tamizado, screening, eficiente para identificar qué parámetros importan antes de un análisis más costoso. Produce dos métricas por parámetro: mu estrella, que mide la importancia, la magnitud del efecto, y sigma, que mide la no linealidad y las interacciones.

```python
import numpy as np
from SALib.sample import morris as morris_sample
from SALib.analyze import morris as morris_analyze

# La salida que se analiza es el PIB proyectado a 10 años por la logística
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

La lectura sigue el diagrama de cuadrantes de la guía de estudio. Un parámetro con mu estrella bajo no influye y puede ignorarse. Uno con mu estrella alto y sigma bajo influye de forma lineal, importante para calibrar. Uno con mu estrella alto y sigma alto influye de forma no lineal, con interacciones fuertes, y es el que debe estimarse con mayor cuidado. Para la logística del PIB, se espera que el techo K y la tasa r dominen, y que sus interacciones no sean despreciables, pues la curva en S las acopla.

## Los índices de Sobol para cuantificar

Morris indica cuáles parámetros importan; Sobol indica exactamente cuánto. Los índices de Sobol descomponen la varianza de la salida y atribuyen a cada parámetro su fracción. El índice de primer orden S1 mide el efecto individual aislado; el índice total ST incluye además todas las interacciones, por lo que ST siempre es mayor o igual que S1.

```python
from SALib.sample import sobol as sobol_sample
from SALib.analyze import sobol as sobol_analyze

X = sobol_sample.sample(problema, 1024)
Y = np.array([proyeccion_logistica(r, K, y0) for r, K, y0 in X])
Si = sobol_analyze.analyze(problema, Y)

for nombre, s1, st in zip(problema["names"], Si["S1"], Si["ST"]):
    print(f"{nombre}: S1={s1:.3f}  ST={st:.3f}")
```

La interpretación sigue las reglas de la guía de estudio. Si S1 es cercano a ST, el parámetro actúa de forma casi lineal, con pocas interacciones. Si ST es mucho mayor que S1, el parámetro importa sobre todo por sus interacciones, su efecto depende de los otros. Si ST es menor que 0.01, el parámetro puede fijarse a su valor nominal sin pérdida. Si S1 es mayor que 0.10, el parámetro es crítico para la calibración y conviene invertir esfuerzo en estimarlo con datos de campo.

## Lo que el análisis de sensibilidad indica del caso

El resultado típico para la logística del PIB es que el techo K domina la incertidumbre de la proyección a largo plazo, mientras que a corto plazo la tasa r pesa más. Esto tiene una consecuencia directa para la interpretación: la enorme incertidumbre sobre cuándo el bloque BRICS+ alcanza su techo proviene casi por completo de no conocer dónde está ese techo, no de no conocer la tasa. Para reducir la incertidumbre de la proyección de transición, está claro dónde invertir esfuerzo: en acotar K, no en afinar r.

Esto conecta con el flujo de trabajo completo que propone la guía de estudio: tamizar con Morris, calibrar los parámetros críticos con datos, cuantificar la incertidumbre residual con Sobol, y recién entonces optimizar o decidir. El análisis de sensibilidad no es un adorno final, sino lo que indica dónde está la ignorancia que más importa.

## Los tres tipos de incertidumbre

La guía de estudio distingue tres tipos de incertidumbre y aclara cuál aborda el análisis de sensibilidad. La estocástica, la aleatoriedad del sistema, se reduce con más corridas. La de parámetros, no conocer los valores exactos, no se reduce con más corridas, sino con mejores datos, y es la que aborda el análisis de sensibilidad. La de modelo, que el modelo no capture todo, no se reduce con ninguna de las dos y es la más peligrosa, pues es invisible. Para el PIB, la incertidumbre de modelo incluye todo lo que la logística ignora: política, sanciones, crisis. Ningún Sobol la mide, y por ello la honestidad sobre los supuestos del Tema 4 es irremplazable.

## Bibliografía

El análisis de sensibilidad global, con Morris para tamizar y Sobol para cuantificar, identifica qué parámetros mueven la conclusión y cuáles pueden fijarse, capturando las interacciones que OAT pierde. Para el PIB revela que la incertidumbre del techo domina la proyección de largo plazo, indicando dónde invertir esfuerzo de datos. Distingue la incertidumbre de parámetros, que sí aborda, de la de modelo, que ningún método numérico resuelve. Con la incertidumbre comprendida, el último capítulo del Tema 6 reúne todo en una lectura honesta de los resultados BRICS+ frente a G7.

## Referencias

1. Herman, J., & Usher, W. (2017). SALib: An open-source Python library for sensitivity analysis. *Journal of Open Source Software, 2*(9), 97. https://doi.org/10.21105/joss.00097
