# 3.1 Del fenómeno continuo a la decisión de negocio

Los dos primeros temas fueron técnicos. Este tema responde la pregunta que un cliente o un jefe siempre termina haciendo: y esto para qué sirve. La Ciencia de Datos para Negocios no premia el modelo más elegante, premia el modelo que cambia una decisión. En este capítulo conecto los sistemas continuos con esa lógica, porque modelar el PIB de un bloque no es un fin en sí mismo, es el insumo de decisiones reales de inversión, política y estrategia.

## El modelo existe para una decisión, no al revés

La guía del curso lo pone como propiedad fundamental de todo modelo: el propósito. Todo modelo existe para un objetivo específico, y el mismo sistema puede tener modelos distintos según para qué se construyan (Law, 2014). Esto invierte el orden ingenuo. No empiezo eligiendo una técnica bonita y después busco dónde aplicarla. Empiezo por la decisión que hay que tomar y dejo que esa decisión determine qué modelo necesito y con qué precisión.

Para el caso del libro, la decisión de fondo no es académica. Si soy un fondo soberano, un banco de desarrollo o un ministerio de un país emergente, la pregunta sobre la trayectoria relativa de BRICS+ y G7 condiciona dónde coloco reservas, en qué moneda me endeudo y con qué bloque profundizo lazos comerciales. El modelo del PIB es la herramienta que convierte una intuición geopolítica en una cifra con incertidumbre acotada sobre la cual decidir.

## Las tres preguntas de negocio que un sistema continuo responde

En mi experiencia, casi toda aplicación de modelos continuos a negocios cae en una de tres preguntas.

La primera es la proyección: hacia dónde va la variable y con qué incertidumbre. Es lo que responden ARIMA y la ODE proyectada. Sirve para planeación, presupuesto y dimensionamiento.

La segunda es el escenario: qué pasaría si cambia un supuesto. Es lo que responde la simulación de la ODE bajo parámetros distintos, o un ARIMA con intervención. Sirve para evaluar políticas y riesgos.

La tercera es el umbral: cuándo se cruza una frontera relevante. Es lo que responde buscar el año del cruce entre dos trayectorias. Sirve para timing de decisiones, como cuándo reasignar una cartera.

El caso BRICS+ frente a G7 toca las tres: proyecto el producto de cada bloque, simulo escenarios de aceleración o sanciones, y busco el umbral del cruce. Cada técnica de los temas anteriores alimenta una de estas preguntas.

## La cadena que conecta el modelo con el valor

La guía describe un flujo que me parece la mejor síntesis de cómo se crea valor con estos modelos: el aprendizaje automático predice los parámetros del sistema, la simulación genera escenarios bajo incertidumbre, y la optimización encuentra la mejor política dentro de esos escenarios (Law, 2014). En ese flujo, los modelos continuos ocupan el centro, el de la simulación de escenarios.

Lo traduzco al caso. Un modelo de series de tiempo estima la tasa de crecimiento y su incertidumbre, que son los parámetros. Una simulación proyecta múltiples trayectorias del PIB del bloque bajo esos parámetros, que son los escenarios. Y una regla de decisión, por ejemplo qué fracción de reservas mover, se optimiza sobre la distribución de resultados. El modelo no decide solo, pero sin él la decisión es a ciegas.

## Lo que diferencia un entregable de negocio de un ejercicio académico

Vengo del mundo del AI engineering, donde aprendí por las malas que un modelo en un notebook no es un producto. Un entregable de negocio tiene tres cosas que un ejercicio académico suele omitir.

Tiene una métrica de decisión conectada al objetivo, no solo un RMSE técnico. Al cliente no le importa el RMSE en escala logarítmica, le importa si la recomendación cambia y cuánto puede perder si el modelo se equivoca.

Tiene la incertidumbre cuantificada y comunicada, porque una decisión bajo un punto estimado sin intervalo es irresponsable. Aquí los intervalos de predicción de ARIMA y la distribución de escenarios de la simulación son el corazón del entregable.

Tiene una recomendación accionable, una frase que dice qué hacer, no solo qué se observó. El Tema 7 está dedicado por completo a ese salto.

## Cierre

Un sistema continuo se aplica a negocios cuando responde una pregunta de proyección, de escenario o de umbral que cambia una decisión. El modelo existe para esa decisión, su valor se mide en la calidad de la decisión que habilita, y un buen entregable carga métrica de negocio, incertidumbre comunicada y recomendación accionable. La siguiente página aterriza esto en el proceso ordenado que sigo para construir un modelo, las siete etapas de la metodología del curso aplicadas a nuestro caso.

## Referencias

Law, A. M. (2014). *Simulation modeling and analysis* (5a ed.). McGraw-Hill.

Provost, F., & Fawcett, T. (2013). *Data science for business: What you need to know about data mining and data-analytic thinking*. O'Reilly Media.
