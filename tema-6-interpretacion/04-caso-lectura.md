# 6.4 Caso: lectura de los resultados BRICS+ contra G7

Cierro el Tema 6 con la lectura completa y honesta de los resultados. Reúno el pronóstico, sus intervalos, los parámetros y la sensibilidad para responder la pregunta del libro sin caer en ninguno de los errores de interpretación que fui marcando. Este es el capítulo donde un análisis técnico se convierte en una afirmación defendible sobre el mundo, y donde la disciplina de no exagerar se vuelve tan importante como la técnica.

## La pregunta, otra vez, con precisión

La pregunta no es van a ganar los BRICS+. Esa frase no es interpretable. La pregunta precisa, fijada en el Tema 4, es: según los modelos seleccionados, cómo evoluciona el PIB de cada bloque en los próximos diez años, con cuánta incertidumbre, y bajo qué unidad de medida. Solo con esa precisión la respuesta significa algo.

## El resultado central y su intervalo

```python
import wbgapi as wb
import numpy as np
import pandas as pd
from statsmodels.tsa.arima.model import ARIMA

def serie_bloque(paises):
    df = wb.data.DataFrame("NY.GDP.MKTP.CD", paises, time=range(1995, 2023))
    df.columns = [int(c.replace("YR", "")) for c in df.columns]
    s = (df.sum(axis=0) / 1e12).sort_index()
    s.index = pd.period_range("1995", "2022", freq="Y")
    return s

brics = serie_bloque(["BRA", "RUS", "IND", "CHN", "ZAF", "EGY", "ETH", "IRN", "ARE"])
g7 = serie_bloque(["USA", "JPN", "DEU", "GBR", "FRA", "ITA", "CAN"])

def proyectar(serie, pasos=10):
    m = ARIMA(np.log(serie), order=(1, 1, 1)).fit()
    pred = m.get_forecast(steps=pasos)
    return np.exp(pred.predicted_mean), np.exp(pred.conf_int(alpha=0.10))

brics_c, brics_ic = proyectar(brics)
g7_c, g7_ic = proyectar(g7)

print("Año final del horizonte:")
print(f"BRICS+ central: {brics_c.iloc[-1]:.1f}  IC90 [{brics_ic.iloc[-1,0]:.1f}, {brics_ic.iloc[-1,1]:.1f}]")
print(f"G7 central:     {g7_c.iloc[-1]:.1f}  IC90 [{g7_ic.iloc[-1,0]:.1f}, {g7_ic.iloc[-1,1]:.1f}]")
```

Lo primero que hago es no leer solo los centrales. Comparo los intervalos. Si los intervalos de los dos bloques se superponen al final del horizonte, entonces la diferencia entre ellos no es estadísticamente concluyente, exactamente como la guía advierte sobre intervalos que se superponen (Law, 2014). Esa superposición es la diferencia entre afirmar el bloque supera al otro y afirmar la mejor estimación apunta a que el bloque crece más rápido, pero el rango no permite descartar un empate.

## La unidad de medida cambia la conclusión

Aquí está el punto más importante de toda la interpretación, y es genuinamente político. Medido en dólares corrientes de mercado, el G7 todavía produce más en agregado y el cruce puede no aparecer en el horizonte de diez años. Medido en paridad de poder adquisitivo, que ajusta por lo que el dinero realmente compra en cada país, el bloque BRICS+ ya superó al G7 hace varios años según el FMI.

```python
# La misma pregunta con otra unidad da otra respuesta:
# NY.GDP.MKTP.PP.CD = PIB en paridad de poder adquisitivo, dólares internacionales
def serie_ppp(paises):
    df = wb.data.DataFrame("NY.GDP.MKTP.PP.CD", paises, time=range(1995, 2023))
    df.columns = [int(c.replace("YR", "")) for c in df.columns]
    return (df.sum(axis=0) / 1e12).sort_index()

brics_ppp = serie_ppp(["BRA", "RUS", "IND", "CHN", "ZAF", "EGY", "ETH", "IRN", "ARE"])
g7_ppp = serie_ppp(["USA", "JPN", "DEU", "GBR", "FRA", "ITA", "CAN"])
print(f"PPP último año -> BRICS+: {brics_ppp.iloc[-1]:.1f}  G7: {g7_ppp.iloc[-1]:.1f}")
```

Un analista honesto reporta las dos medidas y explica la diferencia, porque cada una responde una pregunta legítima distinta. El dólar de mercado mide peso financiero y capacidad de compra internacional; el PPP mide volumen real de producción y bienestar material interno. La elección de cuál privilegiar no es técnica, es interpretativa, y tiene consecuencias para el relato sobre el orden mundial. Yo me inclino por reportar el PPP como medida principal del peso económico real, porque me parece que captura mejor la capacidad productiva efectiva, pero pongo ambas sobre la mesa para que quien lea decida con la información completa.

## Lo que el modelo no puede decir

La interpretación honesta incluye declarar los límites. El modelo no captura las sanciones, que distorsionan las cifras de Rusia e Irán. No captura la heterogeneidad interna del bloque, dominado por China. No captura un cambio de composición, una salida o una entrada de miembros. No captura una crisis sistémica. Todo eso es incertidumbre de modelo, la que el capítulo 6.3 marcó como la más peligrosa porque ningún Sobol la mide. Presentar la proyección sin estas advertencias sería convertir un modelo limitado en una profecía, que es justo lo que no quiero hacer.

## La afirmación que sí puedo defender

Después de todo el cuidado, esta es la afirmación que el análisis sostiene y que firmaría. La evidencia de tres familias de modelos converge en que el PIB del bloque BRICS+ crece de forma sostenida y más rápido que el del G7, que el bloque está lejos de su techo estimado mientras el G7 está cerca del suyo, y que medido en paridad de poder adquisitivo el bloque emergente ya representa una porción del producto mundial mayor que el G7. La fecha exacta de cualquier cruce en dólares de mercado es incierta y depende de supuestos que el modelo no controla. La dirección del proceso es robusta; su calendario, no.

Esa formulación me parece a la vez técnicamente honesta y fiel a lo que el dato muestra. El ascenso del bloque es real y medible, y no necesito exagerarlo para que la conclusión tenga peso. Tomar en serio un proceso geopolítico de esta magnitud incluye, precisamente, no sobrevenderlo.

## Cierre del Tema 6

Interpretar bien fue resistir la tentación de leer solo los centrales, comparar intervalos antes de afirmar diferencias, reconocer que la unidad de medida cambia la conclusión y reportar ambas, declarar lo que el modelo no puede decir, y separar la dirección robusta del proceso de su calendario incierto. La interpretación honesta no debilita el hallazgo sobre la transición de poder, lo vuelve defendible. Con los resultados bien leídos, el Tema 7 da el último paso: convertir esta lectura en soluciones y recomendaciones accionables.

## Referencias

Law, A. M. (2014). *Simulation modeling and analysis* (5a ed.). McGraw-Hill.

Fondo Monetario Internacional. (2024). *World Economic Outlook database*. https://www.imf.org/en/Publications/WEO

Banco Mundial. (2024). *World Development Indicators* [Conjunto de datos]. https://databank.worldbank.org/source/world-development-indicators
