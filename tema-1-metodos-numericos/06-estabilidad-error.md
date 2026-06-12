# 1.6 Estabilidad, paso y error

Hasta aquí podría suponerse que reducir el paso siempre mejora el resultado y que un método de orden alto es siempre seguro. Ambas ideas son falsas en general. Esta página trata el otro lado del problema: cuándo un método explota, por qué, y cómo elegir el paso sin caer en esas trampas. Es la diferencia entre un resultado confiable y un gráfico que parece razonable pero está contaminado.

## Dos errores distintos que conviene no confundir

En la integración numérica concurren dos fuentes de error de naturaleza opuesta.

El error de truncamiento proviene de aproximar la curva con segmentos. Disminuye al reducir el paso `h`. Lo controlan el orden del método y el tamaño del paso.

El error de redondeo proviene de que la computadora almacena los números con precisión finita. Aumenta al reducir el paso, pues más pasos implican más operaciones y mayor acumulación de redondeo, y porque sumar incrementos diminutos a números grandes ocasiona pérdida de cifras.

La consecuencia es que existe un paso óptimo. Si es demasiado grande, domina el truncamiento. Si es demasiado pequeño, domina el redondeo y, además, se paga un cómputo excesivo. La idea ingenua de que un paso menor es siempre mejor resulta falsa.

## Estabilidad: cuando el método explota

Existe un problema más grave que la imprecisión: la inestabilidad. Para ciertos sistemas, si el paso supera un umbral, el error no solo crece, sino que se amplifica de forma explosiva en cada iteración hasta producir valores sin sentido. El ejemplo canónico es el decaimiento, una ODE que debería tender suavemente a cero:

```
dy/dt = -lambda * y,   con lambda > 0
```

La solución exacta es `y(t) = y0 * exp(-lambda*t)`, que decae de forma monótona hacia cero. Sin embargo, el Euler explícito produce

```
y_{k+1} = y_k * (1 - lambda*h)
```

Si `lambda*h > 2`, el factor `(1 - lambda*h)` tiene valor absoluto mayor que 1, y la secuencia oscila con amplitud creciente: el método produce una explosión numérica para un sistema que en realidad se apaga. Se ilustra a continuación:

```python
import numpy as np

def euler_decaimiento(lam, h, n):
    y = np.zeros(n)
    y[0] = 1.0
    for k in range(n - 1):
        y[k + 1] = y[k] * (1 - lam * h)
    return y

lam = 1.0
for h in [0.5, 1.5, 2.5]:
    y = euler_decaimiento(lam, h, 20)
    estado = "estable" if abs(1 - lam * h) <= 1 else "INESTABLE"
    print(f"h={h}: y final={y[-1]: .3e}  ({estado})")
```

Con `h=2.5` la solución, que debería tender a cero, se dispara a millones. Esto ocurre cuando el paso es demasiado grande para la velocidad a la que el sistema cambia. La lección es severa: un método puede entregar un resultado completamente falso sin advertencia, y el único síntoma es que las cifras se vuelven absurdas.

## Cuando un paso pequeño no basta

Existe un caso especial en el que un sistema obliga a usar pasos diminutos para no explotar, aunque solo interese su comportamiento de largo plazo. A esos sistemas se les denomina rígidos, y los métodos explícitos como Euler y RK4 sufren con ellos, pues el paso queda atado a la parte más rápida del sistema aunque esa parte no sea la relevante. La ventaja práctica es que no se requiere resolver esto manualmente ni dominar la teoría de fondo. Las librerías incluyen integradores especiales, denominados métodos implícitos, diseñados precisamente para sistemas rígidos, que se mantienen estables con pasos grandes. En la práctica basta con conocer su existencia y saber cuándo recurrir a ellos. En la siguiente página se muestra cómo `solve_ivp` los ofrece, de modo que el problema se resuelve eligiendo otro método en una línea de código, no con matemática adicional.

## Cómo elegir el paso en la práctica

La práctica recomendada, cuando no se utiliza un integrador adaptativo, es la siguiente. Se integra con un paso `h`, se integra de nuevo con `h/2`, y se comparan los resultados. Si apenas cambian, el paso ya es suficiente. Si cambian de forma apreciable, el paso es demasiado grande y se reduce. Esta comparación entre dos resoluciones es la idea que subyace al control de paso adaptativo, que automatiza este criterio.

```python
def converge(f, y0, t0, tf, paso, h):
    _, y_grueso = integrar(f, y0, t0, tf, h, paso)
    _, y_fino = integrar(f, y0, t0, tf, h / 2, paso)
    return abs(y_grueso[-1] - y_fino[-1]) / abs(y_fino[-1])
```

Si `converge` devuelve un número pequeño relativo a la precisión requerida para la decisión, el paso es adecuado. Es el mismo espíritu de la guía de estudio al tratar el control de convergencia en Monte Carlo: no se reporta un resultado sin verificar que ya es estable.

## La conexión con el caso del PIB

En los modelos de crecimiento económico el riesgo de inestabilidad numérica es bajo, pues la dinámica es suave y de escala anual. No obstante, el principio de verificar la convergencia del paso sigue siendo obligatorio antes de afirmar nada sobre cuándo un bloque alcanza a otro. Una proyección a treinta años que cambia de respuesta al refinar el paso no es una proyección, sino un artefacto del integrador, y presentarla como un hallazgo geopolítico constituiría un error grave.

## Bibliografía

El error tiene dos caras opuestas, truncamiento y redondeo, y entre ellas existe un paso óptimo. La estabilidad es un problema distinto y más peligroso, pues un paso excesivo puede hacer explotar la solución de un sistema que en realidad es manso. La defensa práctica consiste en comparar dos resoluciones y, mejor aún, dejar que un integrador adaptativo controle el paso de forma automática. Eso es precisamente lo que ofrece `solve_ivp` de SciPy, la herramienta de uso profesional que se presenta en la siguiente página.
