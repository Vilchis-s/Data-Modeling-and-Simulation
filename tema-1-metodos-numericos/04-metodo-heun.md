# 1.4 Método de Heun y Euler mejorado

El problema de Euler es que confía ciegamente en la pendiente del punto de partida y la mantiene durante todo el tramo. Pero la pendiente cambia mientras avanzo. El método de Heun, también llamado Euler mejorado o método del trapecio, corrige esto usando un promedio entre la pendiente del inicio y la del final del tramo. Es el primer salto de calidad real.

## La idea: mirar antes de saltar

Heun trabaja en dos fases, una predicción y una corrección.

Primero hace un paso de Euler común para asomarse a dónde llegaría. A ese punto tentativo lo llamo el predictor:

```
y_pred = y_k + h * f(t_k, y_k)
```

Después evalúa la pendiente en ese punto tentativo, `f(t_{k+1}, y_pred)`, y la promedia con la pendiente del inicio. Ese promedio es una estimación mucho mejor de la pendiente representativa del tramo:

```
pendiente_promedio = ( f(t_k, y_k) + f(t_{k+1}, y_pred) ) / 2
y_{k+1} = y_k + h * pendiente_promedio
```

La intuición es la del trapecio en integración: en vez de aproximar el área bajo la curva de la pendiente con un rectángulo, como hace Euler, la aproximo con un trapecio, que se ajusta mucho mejor cuando la pendiente cambia.

## Implementación

```python
import numpy as np

def paso_heun(f, t, y, h):
    k1 = f(t, y)
    y_pred = y + h * k1
    k2 = f(t + h, y_pred)
    return y + h * (k1 + k2) / 2

def integrar(f, y0, t0, t_final, h, paso):
    t = np.arange(t0, t_final + h, h)
    y = np.zeros(len(t))
    y[0] = y0
    for k in range(len(t) - 1):
        y[k + 1] = paso(f, t[k], y[k], h)
    return t, y
```

## Heun contra Euler en el mismo terreno

Vuelvo al crecimiento exponencial con solución exacta para comparar de frente. Uso el mismo paso para ambos, de modo que la única diferencia sea el método.

```python
r, y0 = 0.06, 1.0
f = lambda t, y: r * y

t_e, y_e = integrar(f, y0, 0, 30, 1.0, paso_euler)
t_h, y_h = integrar(f, y0, 0, 30, 1.0, paso_heun)
exacta = y0 * np.exp(r * t_e[-1])

err_euler = abs(y_e[-1] - exacta) / exacta
err_heun = abs(y_h[-1] - exacta) / exacta
print(f"Euler  h=1: error {err_euler:.4%}")
print(f"Heun   h=1: error {err_heun:.4%}")
```

Con el mismo paso de un año, Heun reduce el error del orden de varios puntos porcentuales a una fracción de punto. La diferencia es enorme y el costo es solo una evaluación extra de `f` por paso. En la mayoría de los problemas de negocio ese intercambio vale totalmente la pena.

## Por qué Heun es de orden 2

Heun es un método de orden 2, lo que significa que su error global es proporcional a `h^2`. La consecuencia práctica es importante: si reduzco el paso a la mitad, el error no se reduce a la mitad como en Euler, sino a la cuarta parte. Lo verifico con el mismo barrido de antes:

```python
for h in [1.0, 0.5, 0.25, 0.125]:
    _, yh = integrar(f, y0, 0, 30, h, paso_heun)
    ex = y0 * np.exp(r * 30)
    print(f"h={h:6.3f}   error={abs(yh[-1]-ex)/ex:.5%}")
```

Cada vez que parto el paso a la mitad, el error cae cuatro veces. Esa es la firma del orden 2. Por una sola evaluación adicional de la función por paso, gano una ley de convergencia cuadrática.

## La lectura para el caso del PIB

Cuando proyecto el PIB de un bloque a treinta años, el sesgo sistemático de Euler hacia abajo se vuelve un problema serio, porque distorsiona justamente la magnitud de la transición que quiero medir. Heun corrige ese sesgo casi por completo al promediar la pendiente de entrada y la de salida del tramo, capturando la curvatura del crecimiento. En un análisis donde la pregunta es cuándo el bloque emergente alcanza al G7, esa corrección puede mover la respuesta varios años, y por eso no es un detalle académico.

## Cierre

Heun mejora a Euler usando un promedio entre la pendiente inicial y una pendiente final estimada, lo que lo convierte en un método de orden 2 con convergencia cuadrática a cambio de una sola evaluación extra por paso. La pregunta natural es hasta dónde se puede llevar esta idea de evaluar la pendiente en varios puntos del tramo. La respuesta es Runge-Kutta de orden 4, que evalúa cuatro pendientes y es el caballo de batalla de la simulación continua. Esa es la siguiente página.

## Referencias

Chapra, S. C., & Canale, R. P. (2021). *Numerical methods for engineers* (8a ed.). McGraw-Hill.

Butcher, J. C. (2016). *Numerical methods for ordinary differential equations* (3a ed.). Wiley.
