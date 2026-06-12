# 1.3 Método de Euler

El método de Euler es el integrador más simple que existe y, por esa razón, el mejor punto de entrada. Es la traducción más literal de la idea de avanzar usando la pendiente que proporciona la ODE. Todo lo que sigue son refinamientos de esta misma intuición.

## La regla

Se parte del punto `(t_k, y_k)`. La ODE proporciona la pendiente exacta allí: `f(t_k, y_k)`. Euler supone que esa pendiente se mantiene constante durante todo el tramo `h` y avanza en línea recta:

```
y_{k+1} = y_k + h * f(t_k, y_k)
```

Eso es todo. Geométricamente, se reemplaza la curva real por su recta tangente en cada punto. El error surge de que la curva se va curvando mientras el avance es recto, de modo que en cada paso se produce una desviación, y esas desviaciones se acumulan.

## Implementación

Siguiendo el esqueleto del capítulo anterior, el método de Euler es una sola función de paso:

```python
import numpy as np

def paso_euler(f, t, y, h):
    return y + h * f(t, y)

def integrar(f, y0, t0, t_final, h, paso):
    t = np.arange(t0, t_final + h, h)
    y = np.zeros(len(t))
    y[0] = y0
    for k in range(len(t) - 1):
        y[k + 1] = paso(f, t[k], y[k], h)
    return t, y
```

## Evaluación sobre un caso de solución conocida

Para medir la calidad de un método numérico se requiere un caso con solución exacta. El crecimiento exponencial es idóneo, pues se conoce la solución cerrada `y(t) = y0 * exp(r*t)`. Se emplea una tasa realista, del orden de la que registró India en su mejor década, y se compara Euler con el valor verdadero.

```python
r = 0.06           # tasa anual tipo economía emergente
y0 = 1.0           # PIB inicial normalizado
f = lambda t, y: r * y

t_eu, y_eu = integrar(f, y0, 0, 30, h=1.0, paso=paso_euler)
y_exacta = y0 * np.exp(r * t_eu)

error_final = abs(y_eu[-1] - y_exacta[-1]) / y_exacta[-1]
print(f"Error relativo a 30 años con h=1: {error_final:.3%}")
```

Con paso anual el error final ronda el 2 a 3 por ciento. No es catastrófico, pero tampoco despreciable al proyectar PIB a tres décadas, donde un par de puntos porcentuales equivalen a cientos de miles de millones de dólares.

## El carácter sistemático del error

Existe un detalle que conviene observar, pues revela el carácter del método. En una curva convexa creciente, como la exponencial, la tangente queda siempre por debajo de la curva. En consecuencia, Euler subestima de forma sistemática: paso a paso se queda corto. No se trata de ruido aleatorio, sino de un sesgo direccional. Esto importa para el caso del PIB, pues implica que un Euler ingenuo tendería a infravalorar el crecimiento del bloque emergente, justo lo contrario de lo deseable al estimar una transición de poder.

## El orden 1 en la práctica

Euler es un método de orden 1, lo que significa que su error global es proporcional a `h`. Al reducir el paso a la mitad, el error se reduce aproximadamente a la mitad. Esto se comprueba barriendo varios pasos:

```python
for h in [1.0, 0.5, 0.25, 0.125]:
    t_h, y_h = integrar(f, y0, 0, 30, h, paso_euler)
    exacta = y0 * np.exp(r * t_h[-1])
    err = abs(y_h[-1] - exacta) / exacta
    print(f"h={h:6.3f}   error relativo={err:.4%}")
```

La salida muestra que el error decae casi exactamente en proporción al paso. Esa es la firma del orden 1. El inconveniente es que ganar un dígito de precisión exige diez veces más cómputo, lo cual resulta muy costoso. Los métodos de orden alto del siguiente capítulo rompen ese intercambio: con Runge-Kutta de orden 4, reducir el paso a la mitad reduce el error dieciséis veces.

## Cuándo es suficiente Euler y cuándo no

Euler sirve para prototipar con rapidez, para comprender la dinámica cualitativa de un sistema y para horizontes cortos donde el error no alcanza a acumularse. No sirve para proyecciones largas que exijan precisión, ni para sistemas rígidos donde se vuelve inestable, aspecto que se trata en el capítulo de estabilidad. En la práctica profesional rara vez se utiliza Euler explícito para entregar un resultado, pero conviene implementarlo primero para disponer de una referencia frente a la cual juzgar métodos superiores.

## Bibliografía

El método de Euler avanza usando la pendiente del punto de partida y nada más. Es intuitivo, económico y de orden 1, lo que lo hace impreciso para horizontes largos. Su error es sistemático, no aleatorio. La forma de mejorar sin pagar el precio de un paso diminuto es emplear mejor información sobre la pendiente del tramo, que es lo que hace el método de Heun en la siguiente página.
