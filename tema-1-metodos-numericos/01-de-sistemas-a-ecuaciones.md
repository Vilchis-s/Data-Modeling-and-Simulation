# 1.1 De sistemas continuos a ecuaciones diferenciales

La pregunta con la que arranca todo el Tema 1 es sencilla de enunciar y difícil de responder bien: si tengo un sistema que cambia de forma continua, cómo lo escribo en un lenguaje que la computadora pueda simular. La respuesta es la ecuación diferencial ordinaria, que de aquí en adelante voy a abreviar como ODE por sus siglas en inglés.

## La derivada como lenguaje del cambio

Un sistema continuo no se describe bien por su valor, sino por cómo cambia. Cuando leo que la economía de India crece al 6 por ciento anual, lo que me están dando es una tasa de cambio relativa: el incremento del PIB por unidad de tiempo, dividido entre el PIB actual. Eso, en notación de cálculo, es

```
(1/y) * dy/dt = 0.06
```

que reacomodado da la ecuación diferencial más simple y más importante de la economía del crecimiento:

```
dy/dt = r * y
```

Esta ecuación dice que la tasa de cambio del PIB es proporcional al PIB mismo. Es el modelo de crecimiento exponencial, y aparece siempre que un sistema se refuerza a sí mismo. La guía del curso lo llama un lazo reforzador: el cambio alimenta más cambio, sin límite, y produce comportamiento exponencial (Law, 2014). Una economía que reinvierte parte de lo que produce es exactamente eso.

## El estado del sistema: qué describe la ecuación

Conviene detenerse en un concepto que recorre todo el tema, el de estado. El estado de un sistema es el conjunto mínimo de números que necesito conocer en un instante para poder predecir hacia dónde va a partir de ahí. En el modelo de crecimiento simple, el estado es un solo número, el PIB actual, porque la ecuación me dice que con saber ese valor ya puedo calcular su tasa de cambio. En sistemas más ricos el estado tiene varias componentes, por ejemplo el PIB de dos bloques que compiten, y entonces necesito todos esos números a la vez para saber qué pasa después.

La razón por la que el estado importa es que una ODE es, en el fondo, una regla que conecta el estado de ahora con el cambio inmediato del estado. No mira el pasado, solo el presente. Esto se parece muchísimo a la propiedad de Markov que la guía describe para los sistemas discretos, donde el siguiente paso depende solo del estado actual y no del historial (Law, 2014). En los sistemas continuos esa misma idea aparece bajo otra forma: toda la historia que el sistema necesita recordar está comprimida en su estado presente. Por eso para arrancar una simulación me basta con un punto de partida, sin tener que cargar con todo el camino anterior.

## Tres comportamientos típicos que conviene reconocer

Antes de meternos en los métodos, vale tener un mapa mental de cómo se comporta un sistema continuo según la forma de su ley, porque reconocer el comportamiento de antemano me dice qué esperar y me sirve para detectar cuándo un modelo está dando algo absurdo.

El primer comportamiento es el crecimiento sin freno. Cuando la tasa de cambio es proporcional al valor y nada la limita, el sistema crece de forma exponencial, cada vez más rápido. Es el caso de `dy/dt = r*y`. En economía describe bien una fase de despegue, pero es irreal a largo plazo porque ningún sistema crece para siempre.

El segundo comportamiento es el crecimiento con saturación. Cuando al motor de crecimiento se le suma un freno que se activa al acercarse a un techo, el sistema dibuja una curva en forma de S: rápido al inicio, lento al final. Es el comportamiento logístico que uso en el caso del capítulo 1.8, y es la forma natural de casi cualquier proceso real de crecimiento, desde una economía hasta la adopción de una tecnología.

El tercer comportamiento es el de equilibrio o decaimiento. Cuando el sistema tiende a regresar a un valor de reposo, su tasa de cambio empuja siempre hacia ese punto, y la variable se acerca a él y se queda. Es lo que la guía llama un lazo balanceador dominante, que frena el cambio en lugar de amplificarlo (Law, 2014). Aparece, por ejemplo, cuando una variable se estabiliza tras un choque.

Casi todo lo que voy a modelar es una combinación de estos tres patrones, y tener clara la diferencia entre ellos es la mejor defensa contra confundir un artefacto numérico con un resultado real.

## De la ley local a la trayectoria global

Lo interesante de una ODE es que es una afirmación local. Solo me dice qué pasa en el instante siguiente, dado dónde estoy ahora. No me dice cuál será el PIB en veinte años. Para saber eso tengo que integrar la ecuación, es decir, encadenar infinitos pasos locales hasta cubrir el horizonte completo.

En algunos casos puedo integrar a mano. La ecuación `dy/dt = r*y` con condición inicial `y(0) = y0` tiene solución cerrada conocida:

```
y(t) = y0 * exp(r * t)
```

Pero esto es la excepción. En cuanto la ley del sistema se vuelve un poco más realista, la solución cerrada desaparece y no queda más remedio que aproximar la integración con un método numérico. Esa es la razón de ser de todo este tema.

## Un primer ejemplo concreto con datos reales

Quiero anclar la idea desde la primera página con el fenómeno que recorre el libro. Voy a descargar el PIB de China, ajustar la tasa de crecimiento promedio implícita en sus primeros años de despegue y comparar el modelo exponencial puro contra lo que de verdad pasó. El punto no es que el modelo exponencial sea bueno, de hecho es malo a largo plazo, sino mostrar cómo una ODE elemental ya captura el motor del fenómeno.

```python
import wbgapi as wb
import numpy as np
import matplotlib.pyplot as plt

# PIB a precios actuales en USD, indicador NY.GDP.MKTP.CD del Banco Mundial
serie = wb.data.DataFrame("NY.GDP.MKTP.CD", "CHN", time=range(1990, 2023))
pib = serie.iloc[0].sort_index()
pib.index = [int(c.replace("YR", "")) for c in pib.index]
pib = pib / 1e12  # billones de USD para leer cómodo

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

El gráfico muestra dos cosas a la vez. Por un lado, la ODE exponencial reproduce sorprendentemente bien la fase de despegue chino, porque durante ese periodo el lazo reforzador domina. Por el otro, hacia el final el modelo se dispara por encima de los datos, porque el crecimiento exponencial no tiene techo y la economía real sí lo tiene. Ese desajuste es la pista de que falta un lazo balanceador, y es justo lo que voy a agregar cuando llegue a la ecuación logística en el caso del capítulo 1.8.

## Qué necesito para simular cualquier ODE

Toda ODE de primer orden se puede escribir en la forma estándar

```
dy/dt = f(t, y),    y(t0) = y0
```

donde `f` es la función que codifica la ley del sistema y `y0` es la condición inicial. Esta forma es la entrada universal de todos los métodos numéricos que vienen. Si soy capaz de escribir mi fenómeno como una `f(t, y)` y conozco un punto de partida, ya puedo simularlo. El resto del tema trata de cómo convertir esa `f` en una trayectoria, paso a paso, controlando el error.

## Cierre

Lo que hay que llevarse de aquí es que modelar un sistema continuo es modelar su tasa de cambio, que la tasa de cambio se escribe como una ODE en forma estándar `dy/dt = f(t, y)`, y que salvo casos triviales esa ecuación no se resuelve a mano. La siguiente página formaliza el problema que los métodos numéricos vienen a resolver: el problema de valor inicial.

## Referencias

Law, A. M. (2014). *Simulation modeling and analysis* (5a ed.). McGraw-Hill.

Strogatz, S. H. (2018). *Nonlinear dynamics and chaos: With applications to physics, biology, chemistry, and engineering* (2a ed.). CRC Press.
