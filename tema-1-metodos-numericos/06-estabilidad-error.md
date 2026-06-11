# 1.6 Estabilidad, paso y error

Hasta aquí podría dar la impresión de que reducir el paso siempre mejora las cosas y que un método de orden alto es siempre seguro. Las dos ideas son falsas en general. Esta página trata el otro lado del problema: cuándo un método explota, por qué, y cómo elegir el paso sin caer en esas trampas. Es la diferencia entre un resultado confiable y un gráfico que parece razonable pero está envenenado.

## Dos errores distintos que conviene no confundir

Cuando integro numéricamente cargo con dos fuentes de error de naturaleza opuesta.

El error de truncamiento viene de aproximar la curva con segmentos. Disminuye cuando reduzco el paso `h`. Es el que controlan el orden del método y el tamaño del paso.

El error de redondeo viene de que la computadora guarda los números con precisión finita. Aumenta cuando reduzco el paso, porque más pasos significan más operaciones y más acumulación de redondeo, y porque sumar incrementos diminutos a números grandes pierde cifras.

La consecuencia es que existe un paso óptimo. Si lo hago demasiado grande, domina el truncamiento. Si lo hago demasiado chico, domina el redondeo y encima pago un cómputo absurdo. La idea ingenua de que más chico es siempre mejor es falsa.

## Estabilidad: cuando el método explota

Hay un problema más grave que la imprecisión, y es la inestabilidad. Para ciertos sistemas, si el paso supera un umbral, el error no solo crece, se amplifica de forma explosiva en cada iteración hasta producir números sin sentido. El ejemplo canónico es el decaimiento, una ODE que debería tender suavemente a cero:

```
dy/dt = -lambda * y,   con lambda > 0
```

La solución exacta es `y(t) = y0 * exp(-lambda*t)`, que decae monótona hacia cero. Pero el Euler explícito da

```
y_{k+1} = y_k * (1 - lambda*h)
```

Si `lambda*h > 2`, el factor `(1 - lambda*h)` tiene valor absoluto mayor que 1, y la secuencia oscila con amplitud creciente: el método produce una explosión numérica para un sistema que en realidad se apaga. Lo muestro:

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

Con `h=2.5` la solución, que debería ir a cero, se dispara a millones. Esto pasa cuando el paso es demasiado grande para la velocidad a la que el sistema cambia. La lección es dura: un método puede dar un resultado completamente falso sin avisar, y el único síntoma es que las cifras se vuelven absurdas.

## Cuando un paso chico no basta

Hay un caso especial en el que un sistema obliga a usar pasos diminutos para no explotar, aunque a uno solo le interese su comportamiento de largo plazo. A esos sistemas se les llama rígidos, y los métodos explícitos como Euler y RK4 sufren mucho con ellos, porque el paso queda atado a la parte más rápida del sistema aunque esa parte no sea la que me importa. La buena noticia es que no tengo que resolver esto a mano ni aprender la teoría de fondo. Las librerías traen integradores especiales, llamados métodos implícitos, diseñados justo para sistemas rígidos, que se mantienen estables con pasos grandes. En la práctica me basta con saber que existen y cuándo cambiar a ellos. En la siguiente página veo cómo `solve_ivp` los ofrece, así que el problema se resuelve eligiendo otro método en una línea de código, no con matemática adicional.

## Cómo elijo el paso en la práctica

Mi receta de trabajo, cuando no uso un integrador adaptativo, es esta. Integro con un paso `h`, integro otra vez con `h/2`, y comparo los resultados. Si casi no cambian, el paso ya es suficiente. Si cambian mucho, el paso es demasiado grande y lo sigo reduciendo. Esta comparación entre dos resoluciones es la idea que está detrás del control de paso adaptativo, que automatiza exactamente este criterio.

```python
def converge(f, y0, t0, tf, paso, h):
    _, y_grueso = integrar(f, y0, t0, tf, h, paso)
    _, y_fino = integrar(f, y0, t0, tf, h / 2, paso)
    return abs(y_grueso[-1] - y_fino[-1]) / abs(y_fino[-1])
```

Si `converge` devuelve un número pequeño relativo a la precisión que necesito para la decisión, el paso está bien. Es el mismo espíritu de la guía del curso al hablar de control de convergencia en Monte Carlo: no se reporta un resultado sin verificar que ya es estable (Law, 2014).

## La conexión con el caso del PIB

En modelos de crecimiento económico el riesgo de inestabilidad numérica es bajo, porque la dinámica es suave y de escala anual. Pero el principio de verificar la convergencia del paso sigue siendo obligatorio antes de afirmar nada sobre cuándo un bloque alcanza a otro. Una proyección a treinta años que cambia de respuesta al refinar el paso no es una proyección, es un artefacto del integrador, y presentarla como un hallazgo geopolítico sería un error grave.

## Cierre

El error tiene dos caras opuestas, truncamiento y redondeo, y entre ellas hay un paso óptimo. La estabilidad es un problema aparte y más peligroso, porque un paso excesivo puede hacer explotar la solución de un sistema que en realidad es manso. La defensa práctica es comparar dos resoluciones y, mejor aún, dejar que un integrador adaptativo controle el paso por nosotros. Eso es precisamente lo que ofrece `solve_ivp` de SciPy, la herramienta que uso en producción y que presento en la siguiente página.

## Referencias

Hairer, E., Nørsett, S. P., & Wanner, G. (1993). *Solving ordinary differential equations I: Nonstiff problems* (2a ed.). Springer.

Law, A. M. (2014). *Simulation modeling and analysis* (5a ed.). McGraw-Hill.
