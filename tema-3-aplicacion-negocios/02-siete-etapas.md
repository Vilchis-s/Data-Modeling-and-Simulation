# 3.2 Las siete etapas de modelado aplicadas

La guía del curso organiza la construcción de un modelo en siete etapas (Law, 2014). No son burocracia: son la diferencia entre un modelo que se sostiene y uno que se cae al primer cuestionamiento. En este capítulo recorro las siete etapas aplicándolas, una por una, al modelado del PIB de BRICS+ frente a G7, para mostrar que el proceso no es un adorno sino la espina dorsal del trabajo.

## Etapa 1. Formulación del problema

El producto de esta etapa es una definición del problema con cinco dimensiones: decisión, horizonte, alcance, criterio de éxito y restricciones (Law, 2014).

Para el caso: la decisión es si y cuándo reasignar exposición de un bloque a otro. El horizonte es de diez a quince años. El alcance son los agregados de PIB de los dos bloques, no los países individuales ni los sectores. El criterio de éxito es un MAPE fuera de muestra por debajo de un umbral razonable y un intervalo de predicción que no sea tan ancho que la recomendación se vuelva inútil. Las restricciones son que solo dispongo de datos anuales públicos del Banco Mundial, con pocas décadas de historia. Dedico todo el Tema 4 a esta etapa porque formular mal el problema arruina lo demás.

## Etapa 2. Recolección y análisis de datos

El producto es el análisis exploratorio, las estadísticas descriptivas y la identificación de valores atípicos (Law, 2014).

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

En esta etapa descubro los choques que tendré que respetar al modelar: la crisis de 2008, la pandemia de 2020 y, para Rusia, el efecto de las sanciones desde 2014 y 2022. No los borro como si fueran errores; son parte del fenómeno. El Tema 4 desarrolla el análisis exploratorio completo.

## Etapa 3. Construcción del modelo conceptual

El producto es el diagrama causal y los supuestos documentados, sin código todavía (Law, 2014). Esta etapa es puramente conceptual, y la guía insiste en que nunca incluye código.

Para el caso, mi modelo conceptual es: el PIB de cada bloque tiene un motor de crecimiento intrínseco, un freno estructural conforme madura, y está sujeto a choques externos. Los dos bloques compiten por una participación mundial que suma uno. Documento los supuestos críticos: que la composición de los bloques se mantiene, que no hay un colapso sistémico, y que el dólar corriente es la unidad de medida, con todas sus limitaciones.

## Etapa 4. Selección de la estructura matemática

El producto es el tipo de modelo elegido con justificación (Law, 2014). Aquí es donde se decide entre la ODE del Tema 1 y el ARIMA del Tema 2. El criterio, según el árbol de decisión de la guía, es el objetivo: si quiero entender el mecanismo, ODE; si quiero pronosticar con incertidumbre, series de tiempo. Como el caso necesita ambas cosas, uso las dos y las comparo, que es el contenido del Tema 5.

## Etapa 5. Estimación de parámetros

El producto son los parámetros calibrados con datos, por máxima verosimilitud o método de momentos (Law, 2014). Para la ODE es el ajuste de r y K con `curve_fit` del capítulo 1.8. Para el ARIMA es la estimación de los coeficientes que hace `statsmodels` por máxima verosimilitud. En ambos casos los parámetros tienen interpretación, y leerlos es parte del Tema 6.

## Etapa 6. Implementación y verificación

El producto es el código que implementa correctamente el modelo (Law, 2014). Verificar no es validar: verificar es comprobar que el código hace lo que el modelo dice, sin errores de programación. Un truco que uso es comprobar identidades conocidas, como que el promedio de las simulaciones de un GBM coincide con su media teórica, la verificación que la guía señala para no olvidar la corrección de Ito (Law, 2014).

## Etapa 7. Validación y análisis de sensibilidad

El producto es la comparación con datos reales y el análisis de sensibilidad (Law, 2014). La validación es el backtesting del capítulo 2.8. El análisis de sensibilidad, con Morris y Sobol, es el contenido del capítulo 6.3, y responde cuáles parámetros mueven de verdad la conclusión.

## El proceso es iterativo, no lineal

El punto que más me costó interiorizar es que estas etapas no se recorren una vez en orden. Son un ciclo. Si la validación de la etapa 7 falla, regreso a la 4 a cambiar de estructura, o a la 3 a revisar un supuesto. La guía lo dice explícitamente: el proceso es iterativo, y si la validación falla se vuelve a etapas anteriores (Law, 2014). En el caso del PIB, la primera vez que ajusté un ARIMA sin estabilizar la varianza, la validación fue mala, y eso me mandó de regreso a la etapa de transformaciones. Esa iteración no es un fracaso, es el método funcionando.

## Cierre

Las siete etapas, formulación, datos, modelo conceptual, estructura, parámetros, implementación y validación, son el andamiaje que sostiene un modelo serio, y se recorren de forma iterativa. Aplicadas al PIB de los bloques, ordenan todo el trabajo del libro. La siguiente página se enfoca en el producto estrella de este proceso para el negocio, el forecasting, y en cómo se convierte en insumo de decisión.

## Referencias

Law, A. M. (2014). *Simulation modeling and analysis* (5a ed.). McGraw-Hill.

Banco Mundial. (2024). *World Development Indicators* [Conjunto de datos]. https://databank.worldbank.org/source/world-development-indicators
