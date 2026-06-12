# 1.7 Integradores de SciPy: solve_ivp

Implementar Euler, Heun y RK4 manualmente fue necesario para comprender su funcionamiento interno. En el trabajo real, sin embargo, no se reimplementan integradores: se utiliza `scipy.integrate.solve_ivp`, que ofrece métodos adaptativos de calidad profesional, control automático de paso y soporte para sistemas rígidos. El árbol de decisión de la guía de estudio apunta precisamente aquí cuando indica que, ante un sistema continuo con ecuaciones conocidas, la herramienta es `scipy.integrate`.

## La interfaz básica

`solve_ivp` recibe la función `f(t, y)`, el intervalo de integración, la condición inicial y, opcionalmente, los puntos donde se desea evaluar la solución. La función debe devolver la derivada, y la condición inicial se pasa siempre como un arreglo, aun cuando exista una sola variable.

```python
import numpy as np
from scipy.integrate import solve_ivp

r = 0.06
def f(t, y):
    return r * y

sol = solve_ivp(
    f,
    t_span=(0, 30),          # integra de t=0 a t=30
    y0=[1.0],                # condición inicial como lista
    t_eval=np.linspace(0, 30, 31),  # puntos donde se desea la salida
    method="RK45",           # Runge-Kutta adaptativo, el default
    rtol=1e-8, atol=1e-10,   # tolerancias de error
)

print(sol.y[0][-1])          # valor final
print(sol.success)           # True si convergió
```

El método por defecto, `RK45`, es un Runge-Kutta adaptativo que ajusta el paso de forma automática. Internamente realiza lo que en el capítulo anterior se hacía manualmente al comparar dos resoluciones, pero de manera continua y mucho más fina: estima el error local en cada paso y reduce o amplía `h` para mantenerlo dentro de las tolerancias `rtol` y `atol`. Solo se fija cuánta precisión se desea, no el paso.

## Qué hace un integrador por dentro, en términos simples

Conviene comprender la idea de fondo de un integrador adaptativo, sin entrar en la matemática, pues es lo que distingue una herramienta profesional de los métodos manuales del inicio del tema. La intuición es la misma que se empleó al final del capítulo de estabilidad, al comparar la solución con paso `h` y con paso `h/2`: si ambos resultados casi coinciden, el paso es suficientemente fino; si difieren de forma apreciable, el paso es demasiado grande.

Un integrador adaptativo automatiza exactamente esa idea, pero en cada paso y de forma mucho más eficiente. En cada tramo calcula la solución de dos maneras ligeramente distintas y mide cuánto se separan entre sí. Esa separación es su estimación del error cometido en ese paso. Si la estimación supera lo permitido, el integrador rechaza el paso, lo reduce y lo reintenta; si es mucho menor, amplía el paso siguiente para no desperdiciar cómputo. De este modo, el método busca el tamaño de paso más grande que aún cumple con la precisión solicitada.

La precisión solicitada se controla con dos números, las tolerancias `rtol` y `atol`. La tolerancia relativa `rtol` indica cuánto error se acepta en proporción al tamaño de la solución, y es la de uso más frecuente, pues escala adecuadamente cuando la variable crece, como el PIB. La tolerancia absoluta `atol` indica cuánto error se acepta en términos absolutos, y sirve sobre todo para evitar que el método persiga una precisión imposible cuando la solución pasa cerca de cero. Reducir estas tolerancias aumenta la precisión a cambio de mayor cómputo, y aumentarlas produce el efecto contrario. La ventaja principal es que el razonamiento se hace en términos de cuánta precisión exige la decisión, no en términos de un paso que habría que adivinar.

## Por qué el control adaptativo importa

El paso fijo es un compromiso poco eficiente. En las zonas donde la solución es suave se desperdicia cómputo con pasos pequeños, y en las zonas donde cambia rápido se queda corto. Un integrador adaptativo emplea pasos grandes donde puede y pequeños donde debe. Para el PIB, esto implica que puede avanzar con holgura en periodos de crecimiento estable y afinar el paso alrededor de un choque abrupto, como una crisis, sin intervención manual.

## Elección del método según el problema

`solve_ivp` ofrece varios métodos, y elegir adecuadamente forma parte del oficio:

```
RK45    Runge-Kutta adaptativo. Default. Apto para casi todo lo no rígido.
RK23    Orden más bajo, más económico, para precisión modesta.
DOP853  Orden 8, para precisión muy alta.
Radau   Implícito, para sistemas rígidos.
BDF     Implícito multipaso, también para sistemas rígidos.
LSODA   Detecta rigidez por sí mismo y cambia de método. Buen comodín.
```

La regla práctica es sencilla. Se comienza con `RK45`. Si la integración se vuelve muy lenta o falla, se sospecha rigidez y se cambia a `Radau` o `BDF`. En caso de duda, se utiliza `LSODA`, que decide de forma automática. Esto evita el problema de inestabilidad del capítulo anterior, pues los métodos implícitos son estables sin importar el paso.

## Sistemas de varias ecuaciones

La mayoría de los fenómenos relevantes no son una variable aislada, sino varias acopladas. `solve_ivp` maneja sistemas sin esfuerzo adicional: la función devuelve un vector de derivadas. Se ilustra con un modelo de dos bloques que compiten por participación, idea que se retoma en el caso final. Aquí el PIB de un bloque emergente y uno maduro crecen a tasas distintas y comparten un techo común de producto mundial.

```python
def dos_bloques(t, estado):
    brics, g7 = estado
    techo = 200.0                      # techo conjunto de producto mundial
    r_brics, r_g7 = 0.055, 0.018       # tasas tipo emergente vs maduro
    total = brics + g7
    dbrics = r_brics * brics * (1 - total / techo)
    dg7 = r_g7 * g7 * (1 - total / techo)
    return [dbrics, dg7]

sol = solve_ivp(dos_bloques, (0, 60), [30, 45],
                t_eval=np.linspace(0, 60, 61), method="RK45")

brics_final, g7_final = sol.y[0][-1], sol.y[1][-1]
print(f"A 60 años:  BRICS+={brics_final:.1f}   G7={g7_final:.1f}")
```

El sistema acoplado captura algo que una sola ecuación no puede: la competencia por un recurso compartido. Cuando el bloque emergente crece, presiona el techo del maduro, y a la inversa. Es un primer modelo elemental de transición de poder económico, que se refina con datos reales en el caso del capítulo siguiente.

## Cómo se entregan los resultados de solve_ivp

`sol.t` contiene los tiempos y `sol.y` las trayectorias, una fila por variable. Se convierte a un DataFrame de pandas para graficarlo y analizarlo con el resto del stack:

```python
import pandas as pd
df = pd.DataFrame({"anio": sol.t, "BRICS+": sol.y[0], "G7": sol.y[1]})
df.set_index("anio").plot()
```

## Bibliografía

`solve_ivp` es la forma profesional de integrar ODEs en Python: ofrece Runge-Kutta adaptativo, control automático de paso, métodos implícitos para sistemas rígidos y soporte natural para sistemas de varias ecuaciones. Tras comprender los métodos manualmente, utilizar la herramienta no es un atajo, sino ingeniería. Con esta maquinaria disponible, la siguiente página presenta el caso completo del tema: un modelo logístico calibrado con datos reales del Banco Mundial para el PIB del bloque BRICS+.

## Referencias

1. Virtanen, P., Gommers, R., Oliphant, T. E., Haberland, M., Reddy, T., Cournapeau, D., Burovski, E., Peterson, P., Weckesser, W., Bright, J., van der Walt, S. J., Brett, M., Wilson, J., Millman, K. J., Mayorov, N., Nelson, A. R. J., Jones, E., Kern, R., Larson, E., … Vázquez-Baeza, Y. (2020). SciPy 1.0: Fundamental algorithms for scientific computing in Python. *Nature Methods, 17*(3), 261-272. https://doi.org/10.1038/s41592-019-0686-2
