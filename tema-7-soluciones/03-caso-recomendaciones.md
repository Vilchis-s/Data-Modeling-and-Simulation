# 7.3 Caso: recomendaciones basadas en el modelo

En este capítulo entrego el producto final del libro: un conjunto de recomendaciones accionables basadas en todo el trabajo anterior, con el formato del capítulo 7.1 y respaldadas por los escenarios del 7.2. Lo planteo desde la perspectiva de dos clientes distintos para mostrar que el mismo modelo, leído con funciones de pérdida distintas, produce recomendaciones distintas y ambas válidas.

## Cliente 1: un banco de desarrollo del Sur Global

Su decisión es la asignación de exposición de cartera a diez años, con un apetito de riesgo moderado y un mandato de apoyar la integración de economías emergentes.

### Recomendación 1

Acción: aumentar de forma gradual y escalonada la exposición al bloque BRICS+, no de golpe.

Justificación cuantitativa: la simulación GBM del capítulo 7.2 asigna alta probabilidad a que el producto del bloque crezca más rápido que el del G7 en el horizonte, y los tres modelos del Tema 5 convergen en esa dirección. En PPP el bloque ya pesa más que el G7.

Condición: la recomendación vale mientras no haya crisis sistémica ni cambio mayor de composición del bloque. El escalonamiento responde al riesgo de cola del escenario de sanciones, que ensancha la probabilidad de caída.

Monitoreo: revisar la asignación si el dato anual de PIB del bloque cae fuera del intervalo de predicción al 90 por ciento dos años seguidos, señal de que el régimen cambió.

### Recomendación 2

Acción: diversificar dentro del bloque en lugar de concentrar en su economía dominante.

Justificación cuantitativa: el análisis del capítulo 6.4 mostró que el agregado del bloque está dominado por una sola economía, lo que crea riesgo de concentración. Una exposición que replique el agregado hereda esa concentración.

Condición: vale si el objetivo es exposición al bloque como proceso, no a un país en particular.

Monitoreo: vigilar la contribución de cada miembro al crecimiento agregado, recalculada cada año.

## Cliente 2: un fondo de pensiones conservador del G7

Su decisión es la misma variable, exposición a bloques, pero su función de pérdida es opuesta: prioriza no perder sobre capturar crecimiento, y responde a jubilados que no toleran caídas.

### Recomendación

Acción: mantener una exposición pequeña y estable al bloque BRICS+, sin perseguir el crecimiento proyectado.

Justificación cuantitativa: aunque la mediana de la simulación favorece al bloque, el escenario de sanciones del capítulo 7.2 muestra un riesgo de cola, una probabilidad no despreciable de caída del producto, que es inaceptable para un mandato conservador.

Condición: vale mientras el perfil de los beneficiarios siga siendo averso al riesgo.

Monitoreo: revisar solo si la volatilidad estimada del bloque baja de forma sostenida, lo que reduciría el riesgo de cola.

## El mismo modelo, dos recomendaciones

Lo que quiero subrayar es que las dos recomendaciones salen del mismo modelo y del mismo pronóstico. No se contradicen, porque responden a funciones de pérdida distintas, justo lo que anticipé en el capítulo 7.1. El banco de desarrollo, que tolera riesgo y tiene mandato de apoyar al Sur Global, aumenta exposición; el fondo de pensiones conservador la limita. El modelo no decide por ninguno: les da a ambos la misma distribución de futuros con su incertidumbre, y cada uno decide según lo que está dispuesto a perder. Esta es, para mí, la lección más importante de todo el bloque: el modelo es una herramienta de iluminación, no una máquina de veredictos.

## El código que genera el tablero de recomendación

Para cerrar con algo entregable, dejo el esqueleto que produce el tablero de decisión que acompañaría estas recomendaciones, juntando proyección, escenarios y métricas de riesgo en una sola vista.

```python
import numpy as np
import pandas as pd

def tablero_decision(pib_final_por_escenario, nivel_actual):
    filas = []
    for escenario, muestras in pib_final_por_escenario.items():
        filas.append({
            "escenario": escenario,
            "mediana": np.median(muestras),
            "P10": np.percentile(muestras, 10),
            "P90": np.percentile(muestras, 90),
            "P(caída)": np.mean(muestras < nivel_actual),
            "crecimiento_mediano": np.median(muestras) / nivel_actual - 1,
        })
    return pd.DataFrame(filas).set_index("escenario").round(3)

# Se alimenta con las simulaciones por escenario del capítulo 7.2.
# El tablero es el insumo que el comité lee para decidir según su apetito de riesgo.
```

El tablero no recomienda, informa. La columna de probabilidad de caída es la que el cliente conservador mira primero; la columna de crecimiento mediano es la que mira primero el cliente agresivo. Mismo tablero, lecturas distintas.

## Una nota personal sobre el caso

Elegí este caso porque me importa, y no voy a fingir neutralidad sobre el fondo. Creo que el ascenso económico de los BRICS+ es un proceso real que reconfigura el poder global y que abre espacio para que países que durante décadas fueron periferia ganen capacidad de decidir su propio camino. Pero precisamente porque me importa, me obligué a tratarlo con el máximo rigor: reportando intervalos, distinguiendo medidas, declarando límites y resistiendo la tentación de que el modelo dijera lo que yo quería oír. La convicción política y la honestidad técnica no se estorban; al contrario, un proceso que uno toma en serio merece ser medido en serio.

## Cierre

Entregué recomendaciones accionables con su acción, justificación cuantitativa, condición y monitoreo, mostrando que el mismo modelo produce recomendaciones distintas según la función de pérdida del cliente. El tablero de decisión informa sin dictar, y cada cliente lo lee desde su apetito de riesgo. El último capítulo del libro cierra reconociendo con honestidad las limitaciones de todo este trabajo y trazando el camino para mejorarlo.

## Referencias

Law, A. M. (2014). *Simulation modeling and analysis* (5a ed.). McGraw-Hill.

Provost, F., & Fawcett, T. (2013). *Data science for business: What you need to know about data mining and data-analytic thinking*. O'Reilly Media.
