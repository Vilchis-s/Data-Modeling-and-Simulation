# 1.2 El problema de valor inicial y la discretización

En la página anterior se llegó a la forma estándar de una ODE. A continuación se nombra el problema en su forma completa y se expone la idea que comparten todos los métodos numéricos que vienen después: la discretización.

## El problema de valor inicial

Un problema de valor inicial, o PVI, está compuesto por dos elementos:

```
dy/dt = f(t, y)        la ley del sistema
y(t0) = y0             el punto de partida
```

La ley por sí sola no basta. La ecuación `dy/dt = r*y` describe infinitas economías que crecen a la misma tasa pero parten de tamaños distintos. Lo que selecciona una trayectoria concreta es la condición inicial. Esto admite una lectura pertinente para el caso de los bloques: dos países pueden compartir la misma dinámica de crecimiento y terminar en posiciones radicalmente distintas únicamente por el punto en que arrancaron. La historia importa, y en una ODE la historia entra por `y0`.

## Por qué no se resuelve de forma exacta en la computadora

La computadora no opera con el infinito. No puede recorrer un continuo de instantes entre `t0` y el horizonte final. Solo puede ejecutar operaciones discretas, una tras otra. Por ello, en lugar de la trayectoria continua `y(t)`, se calcula una secuencia de puntos

```
t0, t1, t2, ..., tN
y0, y1, y2, ..., yN
```

donde cada `yk` es una aproximación de `y(tk)`. El paso del problema continuo a esta secuencia de puntos se denomina discretizar. Es la misma idea de cuando se dibuja una curva en pantalla: no se traza la curva real, sino muchos puntos cercanos que el ojo une.

## El paso de integración

El parámetro que controla todo el proceso es el paso de integración, denotado `h`. Es la distancia temporal entre dos puntos consecutivos:

```
t_{k+1} = t_k + h
```

Un paso pequeño implica muchos puntos, mayor cómputo y, en general, mayor precisión. Un paso grande implica pocos puntos, cómputo económico y mayor error. Toda la ingeniería de los métodos numéricos consiste, en el fondo, en extraer la máxima precisión de un paso dado. Un buen método con paso grande puede superar en precisión a un método deficiente con paso pequeño, como se muestra al comparar Euler con Runge-Kutta más adelante.

## La idea común a todos los métodos: avanzar usando la pendiente

Todos los métodos de un paso comparten la misma estructura. Se parte del punto `(t_k, y_k)`. La ODE proporciona la pendiente exacta en ese punto, pues `f(t_k, y_k)` es justamente `dy/dt` allí. Al avanzar en línea recta con esa pendiente durante un tramo `h`, se obtiene una estimación del siguiente punto. La diferencia entre métodos radica únicamente en qué pendiente emplean: la del punto de partida, un promedio de varias, o una combinación ponderada de pendientes evaluadas en puntos intermedios. La estructura central es siempre la misma:

```
y_{k+1} = y_k + h * (alguna pendiente representativa del tramo)
```

## Un esqueleto genérico en Python

Para que la estructura quede explícita antes de presentar los métodos concretos, se incluye el esqueleto de un integrador de un paso. Los capítulos siguientes solo modifican la función `paso`.

```python
import numpy as np

def integrar(f, y0, t0, t_final, h, paso):
    """Integra dy/dt = f(t,y) desde t0 hasta t_final con paso h.
    'paso' es la regla concreta del método (Euler, Heun, RK4...)."""
    t = np.arange(t0, t_final + h, h)
    y = np.zeros(len(t))
    y[0] = y0
    for k in range(len(t) - 1):
        y[k + 1] = paso(f, t[k], y[k], h)
    return t, y
```

Con este esqueleto, implementar un método numérico se reduce a escribir su función `paso`. Esa es la promesa del tema: una sola arquitectura, varias reglas de paso, distinta calidad.

## El error de discretización

Al reemplazar la curva por una secuencia de segmentos rectos se introduce un error. Conviene distinguir desde ya dos errores, pues reaparecen en el capítulo de estabilidad.

El error local es el que se comete en un solo paso, suponiendo partir del valor exacto. El error global es el acumulado tras recorrer todo el horizonte. La relación entre ambos no es trivial, pues los errores pueden amplificarse de un paso al siguiente. Un método se denomina de orden p cuando su error global se reduce proporcionalmente a `h^p`. Euler es de orden 1, Heun de orden 2, Runge-Kutta clásico de orden 4. Cuanto mayor el orden, más rápido decae el error al reducir el paso, lo que justifica el cómputo adicional de los métodos de orden alto.

## Bibliografía

El problema de valor inicial es la ley más la condición inicial. Se resuelve discretizando el tiempo en pasos de tamaño `h` y avanzando con la pendiente que proporciona la ODE. Todos los métodos comparten esa arquitectura y difieren únicamente en cómo eligen la pendiente del tramo. En la siguiente página se implementa el más simple de todos, el método de Euler, que es la versión más literal de esta idea.
