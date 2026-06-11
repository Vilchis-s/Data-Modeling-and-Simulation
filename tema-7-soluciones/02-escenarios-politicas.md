# 7.2 Escenarios y simulación de políticas

Una recomendación robusta no descansa en un solo futuro, descansa en cómo se comporta la decisión a través de muchos futuros plausibles. Este capítulo usa la simulación para generar esos futuros y para evaluar el efecto de políticas, cerrando el segundo paso del flujo de la guía, simulación genera escenarios, y preparando el tercero, la optimización (Law, 2014). Aquí el movimiento browniano geométrico, que reservé en el Tema 5, por fin gana su lugar.

## Por qué simular en vez de proyectar un punto

Un pronóstico central es un solo camino; el mundo es un abanico de caminos. Simular significa generar muchas trayectorias posibles del PIB del bloque, cada una con su propia secuencia de choques aleatorios, y estudiar la distribución de resultados en lugar de un número. Esto es exactamente el método de Monte Carlo de la guía: en vez de un resultado, una distribución de resultados sobre la cual calcular probabilidades y riesgos (Law, 2014).

## Simulación del PIB con GBM

El GBM es el modelo natural para esto, porque es una ODE de crecimiento con ruido y garantiza valores positivos, apropiado para un PIB. Calibro el drift y la volatilidad de los datos históricos y genero trayectorias, respetando la corrección de Ito en el exponente que la guía marca como obligatoria (Law, 2014).

```python
import wbgapi as wb
import numpy as np
import pandas as pd

brics = ["BRA", "RUS", "IND", "CHN", "ZAF", "EGY", "ETH", "IRN", "ARE"]
df = wb.data.DataFrame("NY.GDP.MKTP.CD", brics, time=range(1995, 2023))
df.columns = [int(c.replace("YR", "")) for c in df.columns]
pib = (df.sum(axis=0) / 1e12).sort_index()

log_ret = np.log(pib / pib.shift(1)).dropna()
mu, sigma = log_ret.mean(), log_ret.std()
s0 = pib.values[-1]

def simular_gbm(s0, mu, sigma, anios, n_trayectorias, rng):
    pasos = np.zeros((n_trayectorias, anios + 1))
    pasos[:, 0] = s0
    for t in range(1, anios + 1):
        z = rng.standard_normal(n_trayectorias)
        # corrección de Ito (mu - sigma^2/2) en el exponente de las trayectorias
        pasos[:, t] = pasos[:, t-1] * np.exp((mu - sigma**2/2) + sigma * z)
    return pasos

rng = np.random.default_rng(7)
trayectorias = simular_gbm(s0, mu, sigma, anios=10, n_trayectorias=10000, rng=rng)
pib_final = trayectorias[:, -1]
```

## De las trayectorias a las métricas de riesgo

Con 10000 trayectorias tengo una distribución del PIB del bloque a diez años, y sobre ella calculo las métricas de riesgo que la guía propone para Monte Carlo (Law, 2014).

```python
p10, p50, p90 = np.percentile(pib_final, [10, 50, 90])
prob_supera = np.mean(pib_final > 1.5 * s0)   # P de crecer más del 50%

print(f"P10 (escenario adverso): {p10:.1f}")
print(f"P50 (mediana):           {p50:.1f}")
print(f"P90 (escenario favorable): {p90:.1f}")
print(f"P(crecer más de 50% en 10 años): {prob_supera:.1%}")
```

La verificación que la guía exige para no equivocarse con Ito es que el promedio de las trayectorias finales se acerque a `s0 * exp(mu * 10)`, sin la corrección, porque la corrección vive en el exponente de cada trayectoria, no en la media (Law, 2014).

```python
media_teorica = s0 * np.exp(mu * 10)
print(f"Media simulada: {pib_final.mean():.1f}  vs  teórica: {media_teorica:.1f}")
```

## Simular una política: el efecto de un choque

Lo más potente de simular es que puedo introducir políticas o choques y ver su efecto sobre la distribución. Modelo un escenario que me interesa políticamente: el efecto de un programa sostenido de integración comercial intra-bloque que eleve el drift, contra el efecto de una intensificación de sanciones que lo reduzca y aumente la volatilidad.

```python
escenarios = {
    "base": (mu, sigma),
    "integracion_intrabloque": (mu + 0.010, sigma),        # mayor drift
    "sanciones_intensificadas": (mu - 0.015, sigma * 1.4), # menor drift, más volatilidad
}

resumen = {}
for nombre, (m, s) in escenarios.items():
    fin = simular_gbm(s0, m, s, 10, 10000, np.random.default_rng(7))[:, -1]
    resumen[nombre] = {
        "mediana": np.median(fin),
        "P10": np.percentile(fin, 10),
        "P(caída)": np.mean(fin < s0),
    }
print(pd.DataFrame(resumen).T.round(2))
```

Cada escenario produce una distribución distinta, y comparar sus medianas y sus colas dice cuánto mueve cada política el resultado. El escenario de integración intra-bloque desplaza toda la distribución hacia arriba; el de sanciones la desplaza hacia abajo y le ensancha la cola inferior, el riesgo de caída. Esto traduce un debate geopolítico, integración contra sanciones, en una diferencia cuantificable de distribuciones de producto.

## La lectura del riesgo de cola

Para una decisión, las medianas importan menos que las colas. La guía insiste en que la métrica de riesgo principal no es el promedio sino la probabilidad de resultado adverso y las medidas de cola como el valor en riesgo (Law, 2014). El escenario de sanciones no solo baja la mediana, sino que engorda la probabilidad de una caída del producto, y ese riesgo de cola es lo que una política prudente debe pesar. Una recomendación que solo mirara la mediana subestimaría el daño potencial del escenario adverso.

## Cierre

Simular convierte un pronóstico de un solo camino en una distribución de futuros sobre la cual calculo probabilidades y riesgos de cola, usando GBM con la corrección de Ito bien puesta. Introducir políticas como cambios en el drift y la volatilidad permite comparar sus efectos sobre toda la distribución, traduciendo debates geopolíticos en diferencias cuantificables. El riesgo de cola, no la mediana, es lo que debe pesar una recomendación prudente. Con escenarios sobre la mesa, el siguiente capítulo construye recomendaciones concretas basadas en ellos.

## Referencias

Law, A. M. (2014). *Simulation modeling and analysis* (5a ed.). McGraw-Hill.

Hull, J. C. (2018). *Options, futures, and other derivatives* (10a ed.). Pearson.
