# 1.5 Runge-Kutta de cuarto orden

Si Euler usa una pendiente y Heun usa dos, la pregunta natural es qué ocurre al usar más. El método de Runge-Kutta de cuarto orden, abreviado RK4, evalúa cuatro pendientes dentro de cada tramo y las combina con pesos cuidadosamente elegidos. El resultado es de orden 4 y constituye, sin exageración, el método estándar de la simulación de sistemas continuos. Cuando se requiere integrar una ODE sin restricciones particulares, RK4 o su versión adaptativa es la opción habitual.

## Las cuatro pendientes

RK4 evalúa la pendiente en cuatro ubicaciones estratégicas del tramo:

```
k1 = f(t,         y)               pendiente al inicio
k2 = f(t + h/2,   y + h/2 * k1)    pendiente a media altura, usando k1
k3 = f(t + h/2,   y + h/2 * k2)    pendiente a media altura, usando k2
k4 = f(t + h,     y + h * k3)      pendiente al final, usando k3
```

Y combina las cuatro con un promedio ponderado donde las pendientes centrales pesan el doble:

```
y_{k+1} = y_k + (h/6) * (k1 + 2*k2 + 2*k3 + k4)
```

La intuición es que `k1` y `k4` aportan información de los extremos, mientras que `k2` y `k3` aportan información del centro del tramo, donde la curva suele presentar la pendiente más representativa. El peso doble al centro y la combinación con `1/6` no son arbitrarios: están elegidos para que los términos de error de orden bajo se cancelen, dejando un error de orden 4.

## Implementación

```python
import numpy as np

def paso_rk4(f, t, y, h):
    k1 = f(t, y)
    k2 = f(t + h / 2, y + h / 2 * k1)
    k3 = f(t + h / 2, y + h / 2 * k2)
    k4 = f(t + h, y + h * k3)
    return y + (h / 6) * (k1 + 2 * k2 + 2 * k3 + k4)

def integrar(f, y0, t0, t_final, h, paso):
    t = np.arange(t0, t_final + h, h)
    y = np.zeros(len(t))
    y[0] = y0
    for k in range(len(t) - 1):
        y[k + 1] = paso(f, t[k], y[k], h)
    return t, y
```

## La comparación de los tres métodos

Ahora pueden contrastarse Euler, Heun y RK4 con exactamente el mismo paso para apreciar la diferencia de precisión. Se mantiene el exponencial, pues se dispone de la verdad analítica.

```python
r, y0 = 0.06, 1.0
f = lambda t, y: r * y
exacta = y0 * np.exp(r * 30)

for nombre, paso in [("Euler", paso_euler), ("Heun", paso_heun), ("RK4", paso_rk4)]:
    _, y = integrar(f, y0, 0, 30, 1.0, paso)
    print(f"{nombre:6s} h=1: error {abs(y[-1]-exacta)/exacta:.2e}")
```

Con paso anual, Euler comete un error del orden de `1e-2`, Heun del orden de `1e-3` y RK4 desciende hasta el orden de `1e-6` o mejor. Con un solo año de paso, RK4 ya entrega una proyección a tres décadas con seis cifras correctas. Esto ilustra la afirmación de que un buen método con paso grande supera con holgura a un método deficiente con paso pequeño.

## El orden 4 en la práctica

RK4 reduce su error proporcionalmente a `h^4`. Esto implica que reducir el paso a la mitad reduce el error dieciséis veces. Se confirma a continuación:

```python
prev = None
for h in [1.0, 0.5, 0.25, 0.125]:
    _, y = integrar(f, y0, 0, 30, h, paso_rk4)
    err = abs(y[-1] - exacta) / exacta
    factor = "" if prev is None else f"factor de mejora x{prev/err:.1f}"
    print(f"h={h:6.3f}  error={err:.2e}  {factor}")
    prev = err
```

El factor de mejora ronda 16 en cada refinamiento. Esa convergencia acelerada es la razón de la eficiencia de RK4: gana precisión mucho más rápido de lo que consume cómputo.

## El costo y por qué aun así conviene

RK4 realiza cuatro evaluaciones de `f` por paso, frente a una de Euler y dos de Heun. Parece costoso, pero, como su error decae con `h^4`, admite pasos mucho mayores para la misma precisión. Para una precisión objetivo dada, RK4 suele requerir menos evaluaciones totales que Euler, no más. La eficiencia no se mide en evaluaciones por paso, sino en evaluaciones por dígito de precisión, y en esa métrica RK4 resulta superior.

## Bibliografía

RK4 evalúa cuatro pendientes por tramo y las combina con pesos que cancelan el error de orden bajo, logrando convergencia de orden 4. Es el integrador de propósito general por defecto. Resta por tratar qué ocurre cuando el paso es demasiado grande para la dinámica del sistema, pues en ese caso incluso RK4 puede volverse inestable y producir resultados sin sentido. Ese es el tema de estabilidad, paso y error, que viene a continuación.
