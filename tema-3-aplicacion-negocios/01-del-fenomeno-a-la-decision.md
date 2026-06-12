# 3.1 Del fenómeno continuo a la decisión de negocio

Los dos primeros temas fueron técnicos. Este tema responde la pregunta que un cliente o un responsable termina formulando: para qué sirve esto. La Ciencia de Datos para Negocios no premia el modelo más elegante, sino el modelo que modifica una decisión. En este capítulo se conectan los sistemas continuos con esa lógica, pues modelar el PIB de un bloque no es un fin en sí mismo, sino el insumo de decisiones reales de inversión, política y estrategia.

## El modelo existe para una decisión, no a la inversa

La guía de estudio lo establece como propiedad fundamental de todo modelo: el propósito. Todo modelo existe para un objetivo específico, y un mismo sistema puede admitir modelos distintos según para qué se construyan. Esto invierte el orden ingenuo. No se comienza eligiendo una técnica atractiva para después buscar dónde aplicarla. Se comienza por la decisión que debe tomarse, y esa decisión determina qué modelo se requiere y con qué precisión.

Para el caso del trabajo, la decisión de fondo no es académica. Para un fondo soberano, un banco de desarrollo o un ministerio de un país emergente, la pregunta sobre la trayectoria relativa de BRICS+ y G7 condiciona dónde se colocan reservas, en qué moneda se contrae deuda y con qué bloque se profundizan los lazos comerciales. El modelo del PIB es la herramienta que convierte una intuición geopolítica en una cifra con incertidumbre acotada sobre la cual decidir.

## Las tres preguntas de negocio que un sistema continuo responde

En la práctica, casi toda aplicación de modelos continuos a negocios corresponde a una de tres preguntas.

La primera es la proyección: hacia dónde se dirige la variable y con qué incertidumbre. Es lo que responden ARIMA y la ODE proyectada. Sirve para planeación, presupuesto y dimensionamiento.

La segunda es el escenario: qué ocurriría si cambia un supuesto. Es lo que responde la simulación de la ODE bajo parámetros distintos, o un ARIMA con intervención. Sirve para evaluar políticas y riesgos.

La tercera es el umbral: cuándo se cruza una frontera relevante. Es lo que responde la búsqueda del año del cruce entre dos trayectorias. Sirve para la oportunidad de las decisiones, como cuándo reasignar una cartera.

El caso BRICS+ frente a G7 toca las tres: se proyecta el producto de cada bloque, se simulan escenarios de aceleración o sanciones, y se busca el umbral del cruce. Cada técnica de los temas anteriores alimenta una de estas preguntas.

## La cadena que conecta el modelo con el valor

La guía de estudio describe un flujo que sintetiza bien cómo se crea valor con estos modelos: el aprendizaje automático predice los parámetros del sistema, la simulación genera escenarios bajo incertidumbre, y la optimización encuentra la mejor política dentro de esos escenarios. En ese flujo, los modelos continuos ocupan el centro, el de la simulación de escenarios.

Aplicado al caso: un modelo de series de tiempo estima la tasa de crecimiento y su incertidumbre, que son los parámetros. Una simulación proyecta múltiples trayectorias del PIB del bloque bajo esos parámetros, que son los escenarios. Y una regla de decisión, por ejemplo qué fracción de reservas mover, se optimiza sobre la distribución de resultados. El modelo no decide por sí solo, pero sin él la decisión se toma a ciegas.

## Lo que diferencia un entregable de negocio de un ejercicio académico

Un entregable de negocio tiene tres atributos que un ejercicio académico suele omitir.

Tiene una métrica de decisión conectada al objetivo, no solo un RMSE técnico. Al cliente no le interesa el RMSE en escala logarítmica, sino si la recomendación cambia y cuánto puede perder si el modelo se equivoca.

Tiene la incertidumbre cuantificada y comunicada, pues una decisión bajo un punto estimado sin intervalo es irresponsable. Aquí, los intervalos de predicción de ARIMA y la distribución de escenarios de la simulación son el núcleo del entregable.

Tiene una recomendación accionable, una formulación que indica qué hacer, no solo qué se observó. El Tema 7 está dedicado por completo a ese salto.

## Bibliografía

Un sistema continuo se aplica a negocios cuando responde una pregunta de proyección, de escenario o de umbral que modifica una decisión. El modelo existe para esa decisión, su valor se mide en la calidad de la decisión que habilita, y un buen entregable carga métrica de negocio, incertidumbre comunicada y recomendación accionable. La siguiente página aterriza esto en el proceso ordenado de construcción de un modelo, las siete etapas de la metodología del curso aplicadas al caso.

