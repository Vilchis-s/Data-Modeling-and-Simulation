# 3.2 Las siete etapas de modelado aplicadas

La guía de estudio organiza la construcción de un modelo en siete etapas. No constituyen un trámite: son la diferencia entre un modelo que se sostiene y uno que se desmorona ante el primer cuestionamiento. En este capítulo se recorren las siete etapas aplicándolas, una por una, al modelado del PIB de BRICS+ frente a G7, para mostrar que el proceso no es un adorno, sino la espina dorsal del trabajo.

## Etapa 1. Formulación del problema

El producto de esta etapa es una definición del problema con cinco dimensiones: decisión, horizonte, alcance, criterio de éxito y restricciones.

Para el caso: la decisión es si reasignar exposición de un bloque a otro y cuándo. El horizonte es de diez a quince años. El alcance son los agregados de PIB de los dos bloques, no los países individuales ni los sectores. El criterio de éxito es un MAPE fuera de muestra por debajo de un umbral razonable y un intervalo de predicción que no sea tan ancho que la recomendación pierda utilidad. Las restricciones son que solo se dispone de datos anuales públicos del Banco Mundial, con pocas décadas de historia. El Tema 4 está dedicado por completo a esta etapa, pues formular mal el problema arruina lo demás.

## Etapa 2. Recolección y análisis de datos

El producto es el análisis exploratorio, las estadísticas descriptivas y la identificación de valores atípicos.

```python
import wbgapi as wb
import numpy as np
import pandas as pd

brics = ["BRA", "RUS", "IND", "CHN", "ZAF", "EGY", "ETH", "IRN", "ARE"]
df = wb.data.DataFrame("NY.GDP.MKTP.CD", brics, time=range(1995, 2023))
df.columns = [int(c.replace("YR", "")) for c in df.columns]
pib_brics = (df.sum(axis=0) / 1e12).sort_index()

print(pib_brics.describe())
print("Años con crecimiento negativo:",
      (pib_brics.pct_change() < 0).sum())
```

En esta etapa se descubren los choques que deberán respetarse al modelar: la crisis de 2008, la pandemia de 2020 y, para Rusia, el efecto de las sanciones desde 2014 y 2022. No se eliminan como si fueran errores; son parte del fenómeno. El Tema 4 desarrolla el análisis exploratorio completo.

## Etapa 3. Construcción del modelo conceptual

El producto es el diagrama causal y los supuestos documentados, sin código todavía. Esta etapa es puramente conceptual, y la guía de estudio insiste en que nunca incluye código.

Para el caso, el modelo conceptual es: el PIB de cada bloque tiene un motor de crecimiento intrínseco, un freno estructural conforme madura, y está sujeto a choques externos. Los dos bloques compiten por una participación mundial que suma uno. Se documentan los supuestos críticos: que la composición de los bloques se mantiene, que no hay un colapso sistémico, y que el dólar corriente es la unidad de medida, con todas sus limitaciones.

## Etapa 4. Selección de la estructura matemática

El producto es el tipo de modelo elegido con justificación. Aquí se decide entre la ODE del Tema 1 y el ARIMA del Tema 2. El criterio, según el árbol de decisión de la guía, es el objetivo: si se busca comprender el mecanismo, ODE; si se busca pronosticar con incertidumbre, series de tiempo. Como el caso requiere ambas cosas, se utilizan las dos y se comparan, que es el contenido del Tema 5.

## Etapa 5. Estimación de parámetros

El producto son los parámetros calibrados con datos, por máxima verosimilitud o método de momentos. Para la ODE es el ajuste de r y K con `curve_fit` del capítulo 1.8 (tema-1-metodos-numericos/08-caso-pib-ode.md). Para el ARIMA es la estimación de los coeficientes que realiza `statsmodels` por máxima verosimilitud. En ambos casos los parámetros tienen interpretación, y leerlos es parte del Tema 6.

## Etapa 6. Implementación y verificación

El producto es el código que implementa correctamente el modelo. Verificar no es validar: verificar consiste en comprobar que el código hace lo que el modelo establece, sin errores de programación. Una técnica útil es comprobar identidades conocidas, como que el promedio de las simulaciones de un GBM coincide con su media teórica, la verificación que la guía de estudio señala para no omitir la corrección de Ito.

## Etapa 7. Validación y análisis de sensibilidad

El producto es la comparación con datos reales y el análisis de sensibilidad. La validación es el backtesting del capítulo 2.8 (tema-2-series-de-tiempo/08-validacion-metricas.md). El análisis de sensibilidad, con Morris y Sobol, es el contenido del capítulo 6.3 (tema-6-interpretacion/03-sensibilidad.md), y responde cuáles parámetros mueven de verdad la conclusión.

## El proceso es iterativo, no lineal

El punto que más cuesta interiorizar es que estas etapas no se recorren una sola vez en orden. Constituyen un ciclo. Si la validación de la etapa 7 falla, se regresa a la 4 para cambiar de estructura, o a la 3 para revisar un supuesto. La guía de estudio lo establece de forma explícita: el proceso es iterativo, y si la validación falla se vuelve a etapas anteriores. En el caso del PIB, ajustar un ARIMA sin estabilizar la varianza produjo una validación deficiente, lo que obligó a regresar a la etapa de transformaciones. Esa iteración no es un fracaso, sino el método funcionando.

## Bibliografía

Las siete etapas, formulación, datos, modelo conceptual, estructura, parámetros, implementación y validación, son el andamiaje que sostiene un modelo serio, y se recorren de forma iterativa. Aplicadas al PIB de los bloques, ordenan todo el trabajo. La siguiente página se enfoca en el producto estrella de este proceso para el negocio, el forecasting, y en cómo se convierte en insumo de decisión.

## Referencias

1. Banco Mundial. (2024). *World Development Indicators* [Conjunto de datos]. https://databank.worldbank.org/source/world-development-indicators
