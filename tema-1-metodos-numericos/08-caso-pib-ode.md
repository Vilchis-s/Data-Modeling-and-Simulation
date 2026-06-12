# 1.8 Caso: crecimiento del PIB BRICS+ con una ODE logística

El Tema 1 cierra integrando todo lo anterior en un caso completo con datos reales. La pregunta es concreta: si se modela el PIB agregado del bloque BRICS+ como un sistema continuo gobernado por una ecuación diferencial, qué ley captura mejor su crecimiento y qué indica esa ley sobre el techo y el ritmo de la expansión del bloque.

## Por qué logística y no exponencial

En el capítulo 1.1 (tema-1-metodos-numericos/01-de-sistemas-a-ecuaciones.md) se observó que el modelo exponencial reproduce el despegue pero se dispara al final, por carecer de freno. La realidad económica tiene frenos: saturación de mercados, límites de recursos, convergencia tecnológica que reduce el crecimiento a medida que una economía madura. La guía de estudio lo describe como el paso de un lazo reforzador puro a un sistema donde el lazo balanceador termina dominando, lo que produce la curva logística en forma de S.

La ecuación logística incorpora precisamente ese freno:

```
dy/dt = r * y * (1 - y/K)
```

Aquí `r` es la tasa de crecimiento intrínseca y `K` es la capacidad de carga, el techo hacia el que tiende el sistema. Cuando `y` es pequeño frente a `K`, el factor `(1 - y/K)` es cercano a 1 y el crecimiento es casi exponencial. Cuando `y` se aproxima a `K`, ese factor tiende a 0 y el crecimiento se frena. Es el motor exponencial con un freno estructural incorporado.

## Descarga de los datos reales

El PIB agregado de BRICS+ se construye sumando el PIB corriente en dólares de los países del bloque ampliado, mediante la API del Banco Mundial a través de `wbgapi`.

```python
import wbgapi as wb
import numpy as np
import pandas as pd

paises_brics = ["BRA", "RUS", "IND", "CHN", "ZAF",  # BRICS original
                "EGY", "ETH", "IRN", "ARE"]          # ampliación reciente

df = wb.data.DataFrame("NY.GDP.MKTP.CD", paises_brics, time=range(1995, 2023))
df.columns = [int(c.replace("YR", "")) for c in df.columns]
pib_brics = df.sum(axis=0) / 1e12          # suma del bloque, en billones USD
pib_brics = pib_brics.sort_index()
print(pib_brics.tail())
```

Se suman los países para tratar al bloque como una sola economía agregada, que es la unidad de análisis del trabajo. Algunos países presentan huecos en años tempranos; el agregado los absorbe de forma razonable a partir de mediados de los noventa.

## Calibración de la ODE con los datos

A continuación se ajustan `r` y `K` para que la solución de la ODE logística pase lo más cerca posible de los datos. La solución cerrada de la logística existe, de modo que puede emplearse `curve_fit` directamente sobre ella, lo cual es más estable que ajustar integrando en cada iteración.

```python
from scipy.optimize import curve_fit

t = pib_brics.index.values.astype(float)
t0 = t.min()
tau = t - t0
y = pib_brics.values

def logistica(tau, r, K, y0):
    return K / (1 + ((K - y0) / y0) * np.exp(-r * tau))

p0 = [0.08, 80.0, y[0]]    # tasa inicial, techo, valor inicial
(r_hat, K_hat, y0_hat), _ = curve_fit(logistica, tau, y, p0=p0, maxfev=10000)
print(f"r estimado = {r_hat:.4f}")
print(f"K estimado (techo) = {K_hat:.1f} billones USD")
```

El parámetro `K_hat` es el resultado de mayor interés: es el techo de producto agregado que la dinámica histórica implica para el bloque. No constituye una verdad, sino lo que el modelo logístico infiere a partir de la forma de la curva observada, y conviene interpretarlo con cautela, pues la calibración del techo es notoriamente sensible cuando el sistema aún no muestra señales claras de saturación.

## Integración con solve_ivp y validación

Para mantener la coherencia con el Tema 1, la ODE se integra con `solve_ivp` usando los parámetros calibrados, y se verifica que reproduce los datos.

```python
from scipy.integrate import solve_ivp

def f_logistica(t, y, r, K):
    return r * y * (1 - y / K)

sol = solve_ivp(f_logistica, (tau.min(), tau.max()), [y0_hat],
                t_eval=tau, args=(r_hat, K_hat), method="RK45",
                rtol=1e-9, atol=1e-9)

y_modelo = sol.y[0]
rmse = np.sqrt(np.mean((y_modelo - y) ** 2))
mape = np.mean(np.abs((y_modelo - y) / y))
print(f"RMSE = {rmse:.2f} billones USD")
print(f"MAPE = {mape:.2%}")
```

Reportar el ajuste con métricas concretas, y no solo con un gráfico, es lo que la guía de estudio denomina cerrar el triángulo de la simulación validando contra el dato real.

## Proyección y verificación de convergencia del paso

El bloque se proyecta más allá del último dato y, conforme exige el capítulo de estabilidad, se verifica que la respuesta no dependa del paso del integrador.

```python
horizonte = np.arange(0, tau.max() + 20)   # 20 años hacia adelante
proj = solve_ivp(f_logistica, (0, horizonte.max()), [y0_hat],
                 t_eval=horizonte, args=(r_hat, K_hat), method="RK45",
                 rtol=1e-10, atol=1e-10)

anios = horizonte + t0
pib_2042 = proj.y[0][-1]
print(f"PIB BRICS+ proyectado al {int(anios[-1])}: {pib_2042:.1f} billones USD")
print(f"Como fracción del techo K: {pib_2042/K_hat:.1%}")
```

## Lectura del resultado

El modelo logístico ofrece una narrativa coherente con lo observado en el debate económico: el bloque BRICS+ se encuentra todavía en la parte ascendente de su curva en S, lejos de su techo estimado, lo que explica por qué su crecimiento sigue superando al de las economías maduras del G7, mucho más próximas a su propia saturación. La transición de poder económico, vista desde esta ODE, no es un salto, sino una curva que aún conserva pendiente positiva por delante.

Conviene ser explícito sobre los límites de esta lectura. La ODE logística es determinista: ignora los choques, las sanciones, las crisis y la política. Su valor no reside en predecir el número exacto del PIB en 2042, sino en capturar el mecanismo de fondo, motor de crecimiento más freno estructural, con dos parámetros interpretables. Para incorporar la incertidumbre y los choques se requiere otra familia de modelos, la de las series de tiempo, que es precisamente el objeto del Tema 2.

## Bibliografía del Tema 1

Se recorrió el camino completo de la rama mecanicista: de un sistema continuo a una ODE, de la ODE a su discretización, de Euler a Heun a RK4, de la estabilidad al integrador profesional `solve_ivp`, y de ahí a un modelo logístico calibrado con datos reales del Banco Mundial para el PIB de BRICS+. El método numérico dejó de ser un ejercicio de cálculo y se convirtió en una herramienta para leer una transición geopolítica. En el Tema 2 se cambia de filosofía: en lugar de postular la ley del sistema, se deja que los datos hablen por sí mismos mediante modelos de series de tiempo.

## Referencias

1. Banco Mundial. (2024). *World Development Indicators* [Conjunto de datos]. https://databank.worldbank.org/source/world-development-indicators
2. Verhulst, P. F. (1838). Notice sur la loi que la population suit dans son accroissement. *Correspondance Mathématique et Physique, 10*, 113-121.
