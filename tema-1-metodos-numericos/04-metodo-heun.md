# 1.4 Método de Heun y Euler mejorado

El inconveniente de Euler es que confía por completo en la pendiente del punto de partida y la mantiene durante todo el tramo, cuando en realidad la pendiente cambia a medida que se avanza. El método de Heun, también llamado Euler mejorado o método del trapecio, corrige esto empleando un promedio entre la pendiente del inicio y la del final del tramo. Es el primer salto de calidad significativo.

## La idea: estimar antes de avanzar

Heun opera en dos fases, una predicción y una corrección.

Primero ejecuta un paso de Euler ordinario para estimar el punto de llegada. A ese punto tentativo se le denomina predictor:

```
y_pred = y_k + h * f(t_k, y_k)
```

Después evalúa la pendiente en ese punto tentativo, `f(t_{k+1}, y_pred)`, y la promedia con la pendiente del inicio. Ese promedio es una estimación mucho mejor de la pendiente representativa del tramo:

```
pendiente_promedio = ( f(t_k, y_k) + f(t_{k+1}, y_pred) ) / 2
y_{k+1} = y_k + h * pendiente_promedio
```

La intuición es la del trapecio en integración: en lugar de aproximar el área bajo la curva de la pendiente con un rectángulo, como hace Euler, se aproxima con un trapecio, que se ajusta mucho mejor cuando la pendiente cambia.

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

## Comparación entre Heun y Euler

Se retoma el crecimiento exponencial con solución exacta para comparar directamente. Se emplea el mismo paso para ambos, de modo que la única diferencia sea el método.

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

Con el mismo paso de un año, Heun reduce el error del orden de varios puntos porcentuales a una fracción de punto. La diferencia es considerable y el costo es solo una evaluación adicional de `f` por paso. En la mayoría de los problemas de negocio ese intercambio resulta plenamente conveniente.

## Por qué Heun es de orden 2

Heun es un método de orden 2, lo que significa que su error global es proporcional a `h^2`. La consecuencia práctica es relevante: al reducir el paso a la mitad, el error no se reduce a la mitad como en Euler, sino a la cuarta parte. Esto se verifica con el mismo barrido anterior:

```python
for h in [1.0, 0.5, 0.25, 0.125]:
    _, yh = integrar(f, y0, 0, 30, h, paso_heun)
    ex = y0 * np.exp(r * 30)
    print(f"h={h:6.3f}   error={abs(yh[-1]-ex)/ex:.5%}")
```

Cada vez que el paso se reduce a la mitad, el error decae cuatro veces. Esa es la firma del orden 2. Por una sola evaluación adicional de la función por paso, se obtiene una ley de convergencia cuadrática.

## La lectura para el caso del PIB

Al proyectar el PIB de un bloque a treinta años, el sesgo sistemático de Euler hacia abajo se vuelve un problema considerable, pues distorsiona justamente la magnitud de la transición por medir. Heun corrige ese sesgo casi por completo al promediar la pendiente de entrada y la de salida del tramo, capturando la curvatura del crecimiento. En un análisis cuya pregunta es cuándo el bloque emergente alcanza al G7, esa corrección puede desplazar la respuesta varios años, por lo que no es un detalle académico.

## Bibliografía

Heun mejora a Euler empleando un promedio entre la pendiente inicial y una pendiente final estimada, lo que lo convierte en un método de orden 2 con convergencia cuadrática a cambio de una sola evaluación adicional por paso. La pregunta natural es hasta dónde puede llevarse la idea de evaluar la pendiente en varios puntos del tramo. La respuesta es Runge-Kutta de orden 4, que evalúa cuatro pendientes y constituye el caballo de batalla de la simulación continua. Ese es el tema de la siguiente página.
