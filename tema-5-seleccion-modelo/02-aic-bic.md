# 5.2 Criterios de información: AIC y BIC

El árbol de decisión deja varios candidatos plausibles. Para elegir entre ellos se requiere una métrica que combine dos elementos en tensión: cuán bien ajusta el modelo y cuán complejo es. Esa es la función de los criterios de información, AIC y BIC. Constituyen la forma cuantitativa de aplicar la navaja de Ockham que la guía de estudio establece como propiedad de parsimonia.

## El problema que resuelven

Si los modelos se juzgaran solo por cuán bien ajustan los datos de entrenamiento, siempre ganaría el más complejo, pues más parámetros siempre ajustan mejor el pasado. Pero ajustar mejor el pasado no equivale a predecir mejor el futuro: a partir de cierto punto, los parámetros adicionales capturan ruido en lugar de señal, lo que se denomina sobreajuste. Los criterios de información resuelven esto restando una penalización por cada parámetro, de modo que un modelo solo merece su complejidad si la compensa con suficiente mejora en el ajuste.

## El AIC

El criterio de información de Akaike se define como

```
AIC = 2k - 2 * ln(L)
```

donde k es el número de parámetros y L es la verosimilitud máxima del modelo. El primer término penaliza la complejidad, el segundo premia el ajuste. Entre varios modelos, gana el de menor AIC. La intuición es directa: cada parámetro adicional debe reducir el término de ajuste en más de lo que suma al término de penalización, o no merece su inclusión.

```python
import numpy as np
import wbgapi as wb
import pandas as pd
from statsmodels.tsa.arima.model import ARIMA

pib = wb.data.DataFrame("NY.GDP.MKTP.CD", "IND", time=range(1970, 2023)).iloc[0]
pib = pib.sort_index().astype(float) / 1e12
pib.index = pd.period_range("1970", "2022", freq="Y")
log_pib = np.log(pib)

candidatos = [(0, 1, 0), (1, 1, 0), (0, 1, 1), (1, 1, 1), (2, 1, 1), (1, 1, 2)]
tabla = []
for orden in candidatos:
    m = ARIMA(log_pib, order=orden).fit()
    tabla.append({"orden": orden, "AIC": m.aic, "BIC": m.bic, "k": len(m.params)})

tabla = pd.DataFrame(tabla).sort_values("AIC")
print(tabla.round(2))
```

La tabla ordenada por AIC proporciona el ranking de modelos. El primero es el preferido por AIC, y casi nunca es el más complejo de la lista, que es justamente el objetivo.

## El BIC

El criterio de información bayesiano es muy similar, pero penaliza más la complejidad:

```
BIC = k * ln(n) - 2 * ln(L)
```

La diferencia es que la penalización por parámetro es `ln(n)` en lugar de 2, donde n es el número de observaciones. Como `ln(n)` crece con el tamaño de muestra, el BIC castiga la complejidad con mayor severidad cuanto más datos hay, y por ello tiende a elegir modelos más simples que el AIC. Esto no es un defecto, sino una filosofía distinta: el BIC busca el modelo verdadero suponiendo que está entre los candidatos, mientras que el AIC busca el modelo que mejor predice sin suponer que el verdadero está en la lista.

## Cuándo se prefiere uno u otro

La regla práctica, derivada de la teoría y de la experiencia, es la siguiente. Si el objetivo es predecir, se prefiere AIC, pues está diseñado para minimizar el error de predicción. Si el objetivo es identificar la estructura más parsimoniosa y hay bastantes datos, se prefiere BIC, pues su penalización más severa protege mejor contra el sobreajuste en muestras grandes. Para series económicas cortas, como el PIB anual, ambos suelen coincidir, y cuando difieren, se prefiere el modelo más simple de los dos, fiel a la navaja de Ockham.

```python
mejor_aic = tabla.iloc[0]["orden"]
mejor_bic = tabla.sort_values("BIC").iloc[0]["orden"]
print(f"Mejor por AIC: {mejor_aic}")
print(f"Mejor por BIC: {mejor_bic}")
if mejor_aic != mejor_bic:
    print("Difieren: en duda, se elige el más parsimonioso.")
```

## Lo que AIC y BIC no hacen

Conviene saber qué no indican estos criterios, pues confiarse de ellos es un error frecuente.

No miden el desempeño fuera de muestra de forma directa. Son una estimación teórica del error de predicción basada en la muestra de entrenamiento, no un backtesting real. Por ello nunca se elige un modelo solo por AIC: se utiliza para preseleccionar candidatos y después se validan con la validación temporal del capítulo 5.4 (tema-5-seleccion-modelo/04-validacion-cruzada-temporal.md).

No comparan entre familias distintas de forma confiable. Comparar el AIC de un ARIMA con el de una ODE ajustada por mínimos cuadrados es resbaladizo, pues las verosimilitudes no son directamente comparables si los modelos no están sobre la misma escala y los mismos datos. Para comparar familias distintas se utiliza el error de pronóstico fuera de muestra, que sí es una moneda común, tema del capítulo 5.3 (tema-5-seleccion-modelo/03-comparacion-familias.md).

No capturan si un supuesto está roto. Un modelo puede tener el mejor AIC y, aun así, tener residuos autocorrelacionados, lo que invalida sus intervalos. Por ello el AIC va siempre acompañado del diagnóstico de residuos con Ljung-Box.

## Bibliografía

AIC y BIC operacionalizan la parsimonia restando una penalización por parámetro al ajuste, de modo que la complejidad solo se justifica si se compensa con mejora real. AIC favorece la predicción, BIC favorece la simplicidad y castiga más en muestras grandes. Sirven para preseleccionar candidatos dentro de una familia, pero no reemplazan la validación fuera de muestra ni el diagnóstico de residuos. Para comparar familias tan distintas como una ODE y un ARIMA se requiere una moneda común, el error de pronóstico real, que es el objeto de la siguiente página.

## Referencias

1. Akaike, H. (1974). A new look at the statistical model identification. *IEEE Transactions on Automatic Control, 19*(6), 716-723. https://doi.org/10.1109/TAC.1974.1100705
2. Schwarz, G. (1978). Estimating the dimension of a model. *The Annals of Statistics, 6*(2), 461-464. https://doi.org/10.1214/aos/1176344136
3. Hyndman, R. J., & Athanasopoulos, G. (2021). *Forecasting: Principles and practice* (3a ed.). OTexts. https://otexts.com/fpp3/
