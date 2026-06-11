# 5.2 Criterios de información: AIC y BIC

El árbol de decisión me deja con varios candidatos plausibles. Para elegir entre ellos necesito una métrica que combine dos cosas en tensión: qué tan bien ajusta el modelo y qué tan complejo es. Esa es la función de los criterios de información, AIC y BIC. Son la forma cuantitativa de aplicar la navaja de Ockham que la guía pone como propiedad de parsimonia (Law, 2014).

## El problema que resuelven

Si juzgara los modelos solo por qué tan bien ajustan los datos de entrenamiento, siempre ganaría el más complejo, porque más parámetros siempre ajustan mejor el pasado. Pero ajustar mejor el pasado no es predecir mejor el futuro: a partir de cierto punto, los parámetros extra capturan ruido en lugar de señal, lo que se llama sobreajuste. Los criterios de información resuelven esto restando una penalización por cada parámetro, de modo que un modelo solo merece su complejidad si la paga con suficiente mejora en el ajuste.

## El AIC

El criterio de información de Akaike se define como

```
AIC = 2k - 2 * ln(L)
```

donde k es el número de parámetros y L es la verosimilitud máxima del modelo. El primer término penaliza la complejidad, el segundo premia el ajuste. Entre varios modelos, gana el de menor AIC. La intuición es directa: cada parámetro extra debe reducir el término de ajuste en más de lo que suma al término de penalización, o no vale la pena.

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

La tabla ordenada por AIC me da el ranking de modelos. El de arriba es el preferido por AIC, y casi nunca es el más complejo de la lista, que es exactamente lo que busco.

## El BIC

El criterio de información bayesiano es muy parecido pero penaliza más la complejidad:

```
BIC = k * ln(n) - 2 * ln(L)
```

La diferencia es que la penalización por parámetro es `ln(n)` en vez de 2, donde n es el número de observaciones. Como `ln(n)` crece con el tamaño de muestra, el BIC castiga la complejidad más fuerte cuanto más datos hay, y por eso tiende a elegir modelos más simples que el AIC. Esto no es un defecto, es una filosofía distinta: el BIC busca el modelo verdadero suponiendo que está entre los candidatos, mientras que el AIC busca el modelo que mejor predice sin suponer que el verdadero está en la lista.

## Cuándo prefiero uno u otro

Mi regla práctica, que sale de la teoría y de la experiencia, es esta. Si mi objetivo es predecir, me inclino por AIC, porque está diseñado para minimizar el error de predicción. Si mi objetivo es identificar la estructura más parsimoniosa y tengo bastantes datos, me inclino por BIC, porque su penalización más dura protege mejor contra el sobreajuste en muestras grandes. Para series económicas cortas, como el PIB anual, los dos suelen coincidir, y cuando difieren, prefiero el modelo más simple de los dos, fiel a la navaja de Ockham.

```python
mejor_aic = tabla.iloc[0]["orden"]
mejor_bic = tabla.sort_values("BIC").iloc[0]["orden"]
print(f"Mejor por AIC: {mejor_aic}")
print(f"Mejor por BIC: {mejor_bic}")
if mejor_aic != mejor_bic:
    print("Difieren: en duda, elijo el más parsimonioso.")
```

## Lo que AIC y BIC no hacen

Es importante saber qué no me dicen estos criterios, porque confiarse de ellos es un error que vi cometer seguido.

No miden el desempeño fuera de muestra de forma directa. Son una estimación teórica del error de predicción basada en la muestra de entrenamiento, no un backtesting real. Por eso nunca elijo un modelo solo por AIC: lo uso para preseleccionar candidatos y después los valido con la validación temporal del capítulo 5.4.

No comparan entre familias distintas de forma confiable. Comparar el AIC de un ARIMA con el de una ODE ajustada por mínimos cuadrados es resbaladizo, porque las verosimilitudes no son directamente comparables si los modelos no están sobre la misma escala y los mismos datos. Para comparar familias distintas uso el error de pronóstico fuera de muestra, que sí es una moneda común, tema del capítulo 5.3.

No capturan si un supuesto está roto. Un modelo puede tener el mejor AIC y aun así tener residuos autocorrelacionados, lo que invalida sus intervalos. Por eso el AIC va siempre acompañado del diagnóstico de residuos con Ljung-Box.

## Cierre

AIC y BIC operacionalizan la parsimonia restando una penalización por parámetro al ajuste, de modo que la complejidad solo se justifica si se paga con mejora real. AIC favorece la predicción, BIC favorece la simplicidad y castiga más en muestras grandes. Sirven para preseleccionar candidatos dentro de una familia, pero no reemplazan la validación fuera de muestra ni el diagnóstico de residuos. Para comparar familias tan distintas como una ODE y un ARIMA necesito una moneda común, el error de pronóstico real, y eso es lo que hago en la siguiente página.

## Referencias

Akaike, H. (1974). A new look at the statistical model identification. *IEEE Transactions on Automatic Control, 19*(6), 716-723. https://doi.org/10.1109/TAC.1974.1100705

Schwarz, G. (1978). Estimating the dimension of a model. *The Annals of Statistics, 6*(2), 461-464. https://doi.org/10.1214/aos/1176344136

Hyndman, R. J., & Athanasopoulos, G. (2021). *Forecasting: Principles and practice* (3a ed.). OTexts. https://otexts.com/fpp3/
