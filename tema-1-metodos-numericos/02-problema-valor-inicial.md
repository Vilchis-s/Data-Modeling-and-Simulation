# 1.2 El problema de valor inicial y la discretización

En la página anterior llegué a la forma estándar de una ODE. Ahora le pongo nombre completo al problema y explico la idea que comparten todos los métodos numéricos que vienen después: la discretización.

## El problema de valor inicial

Un problema de valor inicial, o PVI, son dos cosas juntas:

```
dy/dt = f(t, y)        la ley del sistema
y(t0) = y0             el punto de partida
```

La ley sola no basta. La ecuación `dy/dt = r*y` describe infinitas economías que crecen a la misma tasa pero parten de tamaños distintos. Lo que selecciona una trayectoria concreta es la condición inicial. Esto tiene una lectura que me gusta para el caso de los bloques: dos países pueden compartir la misma dinámica de crecimiento y terminar en lugares radicalmente distintos solo por dónde arrancaron. La historia importa, y en una ODE la historia entra por `y0`.

## Por qué no podemos resolverlo de forma exacta en la computadora

La computadora no sabe de infinito. No puede recorrer un continuo de instantes entre `t0` y el horizonte final. Solo sabe hacer operaciones discretas, una tras otra. Entonces, en lugar de la trayectoria continua `y(t)`, vamos a calcular una secuencia de puntos

```
t0, t1, t2, ..., tN
y0, y1, y2, ..., yN
```

donde cada `yk` es una aproximación de `y(tk)`. Pasar del problema continuo a esta secuencia de puntos es lo que se llama discretizar. Es la misma idea que cuando dibujamos una curva en pantalla: no trazamos la curva real, trazamos muchos puntos cercanos y el ojo los une.

## El paso de integración

El parámetro que controla todo es el paso de integración, que voy a llamar `h`. Es la distancia temporal entre dos puntos consecutivos:

```
t_{k+1} = t_k + h
```

Un paso chico significa muchos puntos, más cómputo y, en general, más precisión. Un paso grande significa pocos puntos, cómputo barato y más error. Toda la ingeniería de los métodos numéricos es, en el fondo, sacarle la máxima precisión a un paso dado. Un buen método de paso grande puede ser más preciso que un método malo de paso chico, y eso es exactamente lo que voy a mostrar comparando Euler con Runge-Kutta más adelante.

## La idea común a todos los métodos: avanzar usando la pendiente

Todos los métodos de un paso comparten la misma estructura. Estoy parado en `(t_k, y_k)`. La ODE me dice la pendiente exacta en ese punto, porque `f(t_k, y_k)` es justamente `dy/dt` ahí. Si me muevo en línea recta con esa pendiente durante un tramo `h`, llego a una estimación del siguiente punto. La diferencia entre métodos es solo qué pendiente usan: la del punto de partida, un promedio de varias, o una combinación pesada de pendientes evaluadas en puntos intermedios. Pero la columna vertebral es siempre la misma:

```
y_{k+1} = y_k + h * (alguna pendiente representativa del tramo)
```

## Un esqueleto genérico en Python

Para que la estructura quede explícita antes de ver los métodos concretos, dejo el esqueleto de un integrador de un paso. Los capítulos siguientes solo van a cambiar la función `paso`.

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

Al reemplazar la curva por una secuencia de segmentos rectos introduzco un error. Hay dos errores que conviene distinguir desde ya, porque vuelven en el capítulo de estabilidad.

El error local es el que cometo en un solo paso, suponiendo que partí del valor exacto. El error global es el acumulado tras recorrer todo el horizonte. La relación entre ambos no es trivial, porque los errores se pueden amplificar de un paso al siguiente. Un método se llama de orden p cuando su error global se reduce proporcionalmente a `h^p`. Euler es de orden 1, Heun de orden 2, Runge-Kutta clásico de orden 4. Cuanto mayor el orden, más rápido cae el error cuando reduzco el paso, y por eso vale la pena el cómputo extra de los métodos de orden alto.

## Cierre

El problema de valor inicial es la ley más la condición inicial. Lo resolvemos discretizando el tiempo en pasos de tamaño `h` y avanzando con la pendiente que da la ODE. Todos los métodos comparten esa arquitectura y se diferencian solo en cómo eligen la pendiente del tramo. En la siguiente página implemento el más simple de todos, el método de Euler, que es la versión más literal de esta idea.

## Referencias

Chapra, S. C., & Canale, R. P. (2021). *Numerical methods for engineers* (8a ed.). McGraw-Hill.

Law, A. M. (2014). *Simulation modeling and analysis* (5a ed.). McGraw-Hill.
