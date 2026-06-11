# 1.7 Integradores de SciPy: solve_ivp

Implementar Euler, Heun y RK4 a mano fue necesario para entender qué pasa por dentro. Pero en el trabajo real no reimplemento integradores: uso `scipy.integrate.solve_ivp`, que trae métodos adaptativos de calidad profesional, control automático de paso y soporte para sistemas rígidos. El árbol de decisión de la guía del curso apunta exactamente aquí cuando dice que, ante un sistema continuo con ecuaciones conocidas, la herramienta es `scipy.integrate` (Law, 2014).

## La interfaz básica

`solve_ivp` recibe la función `f(t, y)`, el intervalo de integración, la condición inicial y, opcionalmente, los puntos donde quiero la solución evaluada. La función debe devolver la derivada, y la condición inicial siempre va como un arreglo, aun cuando haya una sola variable.

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
    t_eval=np.linspace(0, 30, 31),  # puntos donde quiero la salida
    method="RK45",           # Runge-Kutta adaptativo, el default
    rtol=1e-8, atol=1e-10,   # tolerancias de error
)

print(sol.y[0][-1])          # valor final
print(sol.success)           # True si convergió
```

El método por defecto, `RK45`, es un Runge-Kutta adaptativo que ajusta el paso solo. Internamente hace lo que en el capítulo anterior hacía a mano comparando dos resoluciones, pero de forma continua y mucho más fina: estima el error local en cada paso y encoge o agranda `h` para mantenerlo dentro de las tolerancias `rtol` y `atol`. Yo solo fijo cuánta precisión quiero, no el paso.

## Qué hace un integrador por dentro, en términos simples

Vale la pena entender la idea de fondo de un integrador adaptativo, sin entrar en la matemática, porque es lo que separa una herramienta profesional de los métodos a mano del inicio del tema. La intuición es la misma que usé al final del capítulo de estabilidad, cuando comparaba la solución con paso `h` y con paso `h/2`: si los dos resultados casi coinciden, el paso es lo bastante fino; si difieren mucho, el paso es demasiado grande.

Un integrador adaptativo automatiza exactamente esa idea, pero en cada paso y de forma mucho más eficiente. En cada tramo calcula la solución de dos maneras ligeramente distintas y mide cuánto se separan una de otra. Esa separación es su estimación del error que está cometiendo en ese paso. Si la estimación es mayor de lo que yo permití, el integrador rechaza el paso, lo encoge y lo vuelve a intentar; si es mucho menor, agranda el paso siguiente para no desperdiciar cómputo. Así, el método busca solo el tamaño de paso más grande que todavía cumple con la precisión que pedí.

La precisión que pido se controla con dos números, las tolerancias `rtol` y `atol`. La tolerancia relativa `rtol` dice cuánto error acepto en proporción al tamaño de la solución, y es la que más uso porque escala bien cuando la variable crece, como el PIB. La tolerancia absoluta `atol` dice cuánto error acepto en términos absolutos, y sirve sobre todo para que el método no se vuelva loco buscando precisión imposible cuando la solución pasa cerca de cero. Bajar estas tolerancias me da más precisión a cambio de más cómputo, y subirlas hace lo contrario. La gran ventaja es que yo razono en términos de cuánta precisión necesito para mi decisión, no en términos de un paso que tendría que adivinar.

## Por qué el control adaptativo importa

El paso fijo es un compromiso torpe. En las zonas donde la solución es suave desperdicio cómputo con pasos chicos, y en las zonas donde cambia rápido me quedo corto. Un integrador adaptativo usa pasos grandes donde puede y chicos donde debe. Para el PIB esto significa que puede avanzar holgado en periodos de crecimiento estable y afinar el paso alrededor de un choque abrupto, como una crisis, sin que yo tenga que intervenir.

## Eligiendo el método según el problema

`solve_ivp` ofrece varios métodos y elegir bien es parte del oficio:

```
RK45    Runge-Kutta adaptativo. Default. Sirve para casi todo lo no rígido.
RK23    Orden más bajo, más barato, para precisión modesta.
DOP853  Orden 8, para cuando necesito precisión muy alta.
Radau   Implícito, para sistemas rígidos.
BDF     Implícito multipaso, también para sistemas rígidos.
LSODA   Detecta rigidez sola y cambia de método. Buen comodín.
```

La regla que sigo es simple. Empiezo con `RK45`. Si la integración se vuelve lentísima o falla, sospecho rigidez y cambio a `Radau` o `BDF`. Si no sé, uso `LSODA`, que decide solo. Esto evita el desastre de inestabilidad del capítulo anterior, porque los métodos implícitos son estables sin importar el paso.

## Sistemas de varias ecuaciones

La mayoría de los fenómenos interesantes no son una variable aislada sino varias acopladas. `solve_ivp` maneja sistemas sin esfuerzo extra: la función devuelve un vector de derivadas. Lo muestro con un modelo de dos bloques que compiten por participación, una idea a la que vuelvo en el caso final. Aquí el PIB de un bloque emergente y uno maduro crecen a tasas distintas y comparten un techo común de producto mundial.

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

El sistema acoplado captura algo que una sola ecuación no puede: la competencia por un recurso compartido. Cuando el bloque emergente crece, le aprieta el techo al maduro, y viceversa. Es un primer modelo crudo de transición de poder económico, y lo refino con datos reales en el caso del capítulo siguiente.

## Cómo entrego los resultados de solve_ivp

`sol.t` trae los tiempos y `sol.y` trae las trayectorias, una fila por variable. Lo paso a un DataFrame de pandas para graficarlo y analizarlo con el resto del stack:

```python
import pandas as pd
df = pd.DataFrame({"anio": sol.t, "BRICS+": sol.y[0], "G7": sol.y[1]})
df.set_index("anio").plot()
```

## Cierre

`solve_ivp` es la forma profesional de integrar ODEs en Python: trae Runge-Kutta adaptativo, control automático de paso, métodos implícitos para sistemas rígidos y soporte natural para sistemas de varias ecuaciones. Después de haber entendido los métodos a mano, usar la herramienta no es hacer trampa, es hacer ingeniería. Con esta maquinaria lista, en la siguiente página armo el caso completo del tema: un modelo logístico calibrado con datos reales del Banco Mundial para el PIB del bloque BRICS+.

## Referencias

Virtanen, P., Gommers, R., Oliphant, T. E., Haberland, M., Reddy, T., Cournapeau, D., Burovski, E., Peterson, P., Weckesser, W., Bright, J., van der Walt, S. J., Brett, M., Wilson, J., Millman, K. J., Mayorov, N., Nelson, A. R. J., Jones, E., Kern, R., Larson, E., … Vázquez-Baeza, Y. (2020). SciPy 1.0: Fundamental algorithms for scientific computing in Python. *Nature Methods, 17*(3), 261-272. https://doi.org/10.1038/s41592-019-0686-2

Law, A. M. (2014). *Simulation modeling and analysis* (5a ed.). McGraw-Hill.
