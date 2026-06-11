# 1.3 Método de Euler

El método de Euler es el integrador más simple que existe y, justamente por eso, es el mejor punto de entrada. Es la traducción más literal de la idea de avanzar usando la pendiente que da la ODE. Todo lo que viene después son refinamientos de esta misma intuición.

## La regla

Estoy en `(t_k, y_k)`. La ODE me da la pendiente exacta ahí: `f(t_k, y_k)`. Euler supone que esa pendiente se mantiene constante durante todo el tramo `h` y avanza en línea recta:

```
y_{k+1} = y_k + h * f(t_k, y_k)
```

Eso es todo. Geométricamente, estoy reemplazando la curva real por su recta tangente en cada punto. El error nace de que la curva se va curvando mientras yo avanzo recto, así que me desvío un poco en cada paso, y esos desvíos se acumulan.

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

## Probándolo donde sí conozco la respuesta

Para medir qué tan bueno es un método numérico necesito un caso con solución exacta. El crecimiento exponencial es perfecto porque ya conozco la solución cerrada `y(t) = y0 * exp(r*t)`. Voy a usar una tasa realista, del orden de la que tuvo India en su mejor década, y comparar Euler contra la verdad.

```python
r = 0.06           # tasa anual tipo economía emergente
y0 = 1.0           # PIB inicial normalizado
f = lambda t, y: r * y

t_eu, y_eu = integrar(f, y0, 0, 30, h=1.0, paso=paso_euler)
y_exacta = y0 * np.exp(r * t_eu)

error_final = abs(y_eu[-1] - y_exacta[-1]) / y_exacta[-1]
print(f"Error relativo a 30 años con h=1: {error_final:.3%}")
```

Con paso anual el error final ronda el 2 a 3 por ciento. No es catastrófico, pero tampoco es despreciable cuando hablamos de proyectar PIB a tres décadas, donde un par de puntos porcentuales son cientos de miles de millones de dólares.

## La sistemática del error: Euler siempre se queda corto en lo convexo

Hay un detalle que vale la pena ver porque revela el carácter del método. En una curva convexa que crece, como la exponencial, la tangente siempre va por debajo de la curva. Entonces Euler subestima de forma sistemática: paso a paso se queda corto. No es ruido aleatorio, es un sesgo direccional. Esto importa para el caso del PIB, porque significa que un Euler ingenuo tendería a infravalorar el crecimiento del bloque emergente, justo al revés de lo que uno querría al estimar una transición de poder.

## El orden 1 en acción: reducir el paso a la mitad reduce el error a la mitad

Euler es un método de orden 1, lo que significa que su error global es proporcional a `h`. Si reduzco el paso a la mitad, el error se reduce aproximadamente a la mitad. Lo compruebo barriendo varios pasos:

```python
for h in [1.0, 0.5, 0.25, 0.125]:
    t_h, y_h = integrar(f, y0, 0, 30, h, paso_euler)
    exacta = y0 * np.exp(r * t_h[-1])
    err = abs(y_h[-1] - exacta) / exacta
    print(f"h={h:6.3f}   error relativo={err:.4%}")
```

La salida muestra que el error cae casi exactamente en proporción al paso. Esa es la firma del orden 1. El problema es que para ganar un dígito de precisión necesito diez veces más cómputo, lo cual es muy caro. Los métodos de orden alto del próximo capítulo rompen ese intercambio: con Runge-Kutta de orden 4, reducir el paso a la mitad reduce el error dieciséis veces.

## Cuándo Euler es suficiente y cuándo no

Euler sirve para prototipar rápido, para entender la dinámica cualitativa de un sistema y para horizontes cortos donde el error no alcanza a acumularse. No sirve para proyecciones largas que exijan precisión, ni para sistemas rígidos donde se vuelve inestable, algo que trato en el capítulo de estabilidad. En la práctica profesional casi nunca uso Euler explícito para entregar un resultado, pero siempre lo implemento primero para tener una referencia contra la cual juzgar métodos mejores.

## Cierre

El método de Euler avanza usando la pendiente del punto de partida y nada más. Es intuitivo, barato y de orden 1, lo que lo hace impreciso para horizontes largos. Su error es sistemático, no aleatorio. La forma de mejorar sin pagar el precio de un paso diminuto es usar mejor información sobre la pendiente del tramo, y eso es lo que hace el método de Heun en la siguiente página.

## Referencias

Chapra, S. C., & Canale, R. P. (2021). *Numerical methods for engineers* (8a ed.). McGraw-Hill.

Butcher, J. C. (2016). *Numerical methods for ordinary differential equations* (3a ed.). Wiley.
