# 7.2 Escenarios y simulación de políticas

Una recomendación robusta no descansa en un solo futuro, sino en cómo se comporta la decisión a través de muchos futuros plausibles. Este capítulo usa la simulación para generar esos futuros y para evaluar el efecto de políticas, cerrando el segundo paso del flujo de la guía de estudio, simulación genera escenarios, y preparando el tercero, la optimización. Aquí el movimiento browniano geométrico, reservado en el Tema 5, encuentra su lugar.

## Por qué simular en lugar de proyectar un punto

Un pronóstico central es un solo camino; el mundo es un abanico de caminos. Simular consiste en generar muchas trayectorias posibles del PIB del bloque, cada una con su propia secuencia de choques aleatorios, y estudiar la distribución de resultados en lugar de un número. Es exactamente el método de Monte Carlo de la guía de estudio: en lugar de un resultado, una distribución de resultados sobre la cual calcular probabilidades y riesgos.

## Simulación del PIB con GBM

El GBM es el modelo natural para esto, pues es una ODE de crecimiento con ruido y garantiza valores positivos, apropiado para un PIB. Se calibra el drift y la volatilidad de los datos históricos y se generan trayectorias, respetando la corrección de Ito en el exponente que la guía de estudio señala como obligatoria.

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

Con 10000 trayectorias se obtiene una distribución del PIB del bloque a diez años, y sobre ella se calculan las métricas de riesgo que la guía de estudio propone para Monte Carlo.

```python
p10, p50, p90 = np.percentile(pib_final, [10, 50, 90])
prob_supera = np.mean(pib_final > 1.5 * s0)   # P de crecer más del 50%

print(f"P10 (escenario adverso): {p10:.1f}")
print(f"P50 (mediana):           {p50:.1f}")
print(f"P90 (escenario favorable): {p90:.1f}")
print(f"P(crecer más de 50% en 10 años): {prob_supera:.1%}")
```

La verificación que la guía de estudio exige para no equivocarse con Ito es que el promedio de las trayectorias finales se acerque a `s0 * exp(mu * 10)`, sin la corrección, pues la corrección reside en el exponente de cada trayectoria, no en la media.

```python
media_teorica = s0 * np.exp(mu * 10)
print(f"Media simulada: {pib_final.mean():.1f}  vs  teórica: {media_teorica:.1f}")
```

## Simular una política: el efecto de un choque

Lo más potente de simular es que permite introducir políticas o choques y observar su efecto sobre la distribución. Se modela un escenario de interés: el efecto de un programa sostenido de integración comercial intra-bloque que eleve el drift, frente al efecto de una intensificación de sanciones que lo reduzca y aumente la volatilidad.

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

Cada escenario produce una distribución distinta, y comparar sus medianas y sus colas indica cuánto mueve cada política el resultado. El escenario de integración intra-bloque desplaza toda la distribución hacia arriba; el de sanciones la desplaza hacia abajo y ensancha su cola inferior, el riesgo de caída. Esto traduce un debate geopolítico, integración frente a sanciones, en una diferencia cuantificable de distribuciones de producto.

## La lectura del riesgo de cola

Para una decisión, las medianas importan menos que las colas. La guía de estudio insiste en que la métrica de riesgo principal no es el promedio, sino la probabilidad de resultado adverso y las medidas de cola como el valor en riesgo. El escenario de sanciones no solo baja la mediana, sino que engrosa la probabilidad de una caída del producto, y ese riesgo de cola es lo que una política prudente debe ponderar. Una recomendación que solo observara la mediana subestimaría el daño potencial del escenario adverso.

## Bibliografía

Simular convierte un pronóstico de un solo camino en una distribución de futuros sobre la cual se calculan probabilidades y riesgos de cola, usando GBM con la corrección de Ito correctamente aplicada. Introducir políticas como cambios en el drift y la volatilidad permite comparar sus efectos sobre toda la distribución, traduciendo debates geopolíticos en diferencias cuantificables. El riesgo de cola, no la mediana, es lo que debe ponderar una recomendación prudente. Con escenarios sobre la mesa, el siguiente capítulo construye recomendaciones concretas basadas en ellos.

