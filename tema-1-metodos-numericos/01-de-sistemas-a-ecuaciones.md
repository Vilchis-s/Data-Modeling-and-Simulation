# 1.1 De sistemas continuos a ecuaciones diferenciales

La pregunta con la cual inicia el Tema 1 es: si se tiene un sistema que cambia de forma continua, cómo se expresa en un lenguaje que la computadora pueda simular? La respuesta es la ecuación diferencial ordinaria, abreviada en adelante como ODE por sus siglas en inglés.

## La derivada como lenguaje del cambio

Un sistema continuo no se describe bien por su valor, sino por cómo cambia. Cuando se afirma que la economía de India crece al 6 por ciento anual, lo que se está dando es una tasa de cambio relativa: el incremento del PIB por unidad de tiempo, dividido entre el PIB actual. En notación de cálculo, esto es

```
(1/y) * dy/dt = 0.06
```

que, reacomodado, da la ecuación diferencial más simple e influyente de la economía del crecimiento:

```
dy/dt = r * y
```

Esta ecuación establece que la tasa de cambio del PIB es proporcional al PIB mismo. Es el modelo de crecimiento exponencial, y aparece siempre que un sistema se refuerza a sí mismo. La guía de estudio lo denomina lazo reforzador: el cambio alimenta más cambio, sin límite, y produce comportamiento exponencial. Una economía que reinvierte parte de lo que produce constituye un ejemplo de ello.

## El estado del sistema: qué describe la ecuación

Conviene detenerse en un concepto que recorre todo el tema, el de estado. El estado de un sistema es el conjunto mínimo de valores que se requiere conocer en un instante para predecir hacia dónde evoluciona a partir de ahí. En el modelo de crecimiento simple, el estado es un solo número, el PIB actual, pues la ecuación indica que con ese valor ya puede calcularse su tasa de cambio. En sistemas más ricos, el estado tiene varias componentes, por ejemplo el PIB de dos bloques que compiten, y entonces se requieren todos esos valores a la vez para determinar la evolución.

La razón por la que el estado importa es que una ODE es, en el fondo, una regla que conecta el estado presente con el cambio inmediato del estado. No considera el pasado, solo el presente. Esta idea se asemeja a la propiedad de Markov que la guía de estudio describe para los sistemas discretos, donde el siguiente paso depende solo del estado actual y no del historial. En los sistemas continuos, esa misma idea adopta otra forma: toda la historia que el sistema necesita recordar queda comprimida en su estado presente. Por ello, para iniciar una simulación basta con un punto de partida, sin necesidad de traer todo el camino anterior.

## Tres comportamientos típicos que conviene reconocer

Antes de abordar los métodos, conviene disponer de un mapa conceptual de cómo se comporta un sistema continuo según la forma de su ley, pues reconocer el comportamiento de antemano permite anticipar qué esperar y detectar cuándo un modelo produce un resultado absurdo.

El primer comportamiento es el crecimiento sin freno. Cuando la tasa de cambio es proporcional al valor y nada la limita, el sistema crece de forma exponencial, cada vez más rápido. Es el caso de `dy/dt = r*y`. En economía describe bien una fase de despegue, pero resulta irreal a largo plazo, pues ningún sistema crece de manera indefinida.

El segundo comportamiento es el crecimiento con saturación. Cuando al motor de crecimiento se suma un freno que se activa al aproximarse a un techo, el sistema describe una curva en forma de S: rápida al inicio, lenta al final. Es el comportamiento logístico que se utiliza en el caso del capítulo 1.8 (tema-1-metodos-numericos/08-caso-pib-ode.md), y constituye la forma natural de casi cualquier proceso real de crecimiento, desde una economía hasta la adopción de una tecnología.

El tercer comportamiento es el de equilibrio o decaimiento. Cuando el sistema tiende a regresar a un valor de reposo, su tasa de cambio empuja siempre hacia ese punto, y la variable se aproxima a él y se estabiliza. Es lo que la guía de estudio denomina un lazo balanceador dominante, que frena el cambio en lugar de amplificarlo. Aparece, por ejemplo, cuando una variable se estabiliza tras un choque.

Casi todos los fenómenos por modelar combinan estos tres patrones, y distinguirlos con claridad es la mejor defensa contra confundir un artefacto numérico con un resultado real.

## De la ley local a la trayectoria global

Lo característico de una ODE es que constituye una afirmación local. Solo indica qué ocurre en el instante siguiente, dado el punto actual. No indica cuál será el PIB dentro de veinte años. Para determinarlo es preciso integrar la ecuación, es decir, encadenar infinitos pasos locales hasta cubrir el horizonte completo.

En algunos casos la integración es analítica. La ecuación `dy/dt = r*y` con condición inicial `y(0) = y0` tiene solución cerrada conocida:

```
y(t) = y0 * exp(r * t)
```

Pero esto es la excepción. En cuanto la ley del sistema se vuelve un poco más realista, la solución cerrada desaparece y no queda alternativa a aproximar la integración con un método numérico. Esa es la razón de ser de todo el tema.

## Un primer ejemplo concreto con datos reales

Conviene anclar la idea desde la primera página con el fenómeno que recorre el trabajo. Se descarga el PIB de China, se ajusta la tasa de crecimiento promedio implícita en sus primeros años de despegue y se compara el modelo exponencial puro con lo efectivamente observado. El propósito no es sostener que el modelo exponencial sea adecuado, pues es deficiente a largo plazo, sino mostrar cómo una ODE elemental ya captura el motor del fenómeno.

```python
import wbgapi as wb
import numpy as np
import matplotlib.pyplot as plt

# PIB a precios actuales en USD, indicador NY.GDP.MKTP.CD del Banco Mundial
serie = wb.data.DataFrame("NY.GDP.MKTP.CD", "CHN", time=range(1990, 2023))
pib = serie.iloc[0].sort_index()
pib.index = [int(c.replace("YR", "")) for c in pib.index]
pib = pib / 1e12  # billones de USD para lectura cómoda

# Tasa de crecimiento promedio implícita entre 1990 y 2010 (fase de despegue)
y0 = pib.loc[1990]
y_ref = pib.loc[2010]
r = np.log(y_ref / y0) / (2010 - 1990)

t = np.arange(1990, 2023)
modelo = y0 * np.exp(r * (t - 1990))

plt.plot(pib.index, pib.values, "o", label="PIB observado (China)")
plt.plot(t, modelo, label=f"Modelo exponencial r={r:.3f}")
plt.xlabel("Año")
plt.ylabel("PIB (billones USD corrientes)")
plt.legend()
plt.title("Crecimiento exponencial: motor correcto, freno ausente")
plt.show()
```

El gráfico muestra dos cosas a la vez. Por un lado, la ODE exponencial reproduce con notable fidelidad la fase de despegue chino, pues durante ese periodo el lazo reforzador domina. Por el otro, hacia el final el modelo se dispara por encima de los datos, pues el crecimiento exponencial carece de techo y la economía real lo tiene. Ese desajuste es la señal de que falta un lazo balanceador, y es precisamente lo que se incorpora al llegar a la ecuación logística en el caso del capítulo 1.8 (tema-1-metodos-numericos/08-caso-pib-ode.md).

## Qué se requiere para simular cualquier ODE

Toda ODE de primer orden admite la forma estándar

```
dy/dt = f(t, y),    y(t0) = y0
```

donde `f` es la función que codifica la ley del sistema y `y0` es la condición inicial. Esta forma es la entrada universal de todos los métodos numéricos que siguen. Si el fenómeno puede expresarse como una `f(t, y)` y se conoce un punto de partida, ya es posible simularlo. El resto del tema trata de cómo convertir esa `f` en una trayectoria, paso a paso, controlando el error.

## Bibliografía

Lo esencial de este capítulo es que modelar un sistema continuo equivale a modelar su tasa de cambio, que la tasa de cambio se expresa como una ODE en forma estándar `dy/dt = f(t, y)`, y que, salvo casos triviales, esa ecuación no admite solución analítica. La siguiente página formaliza el problema que los métodos numéricos vienen a resolver: el problema de valor inicial.
