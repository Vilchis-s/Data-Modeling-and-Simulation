# Sistemas Continuos: Modelado y Simulación

Modelado y Simulación de Datos, sexto semestre. Licenciatura en Ciencia de Datos para Negocios.

Este libro desarrolla el bloque de sistemas continuos del curso. Lo escribí como lo que soy, un científico de datos junior que lleva dos años trabajando en NLP contemporáneo y AI engineering, acostumbrado a pipelines de embeddings, topic modeling y agentes, pero que para esta materia tuvo que bajar al terreno más clásico de las ecuaciones diferenciales, las series de tiempo y la simulación. El reto fue justamente ese: tomar herramientas que parecen de un curso de cálculo numérico y usarlas para responder preguntas de negocio y de economía política.

## El hilo conductor

Decidí no usar ejemplos genéricos de juguete. Todo el libro gira alrededor de una sola pregunta que me interesa de verdad: cómo evoluciona el Producto Interno Bruto del bloque BRICS+ frente al G7, y qué nos dicen los modelos continuos sobre la transición de poder económico que se está discutiendo en este momento.

La elección no es neutral y no pretendo que lo sea. El ascenso de los BRICS+ es uno de los procesos geopolíticos más relevantes de la década. Medir, modelar y proyectar ese ascenso es un ejercicio de ciencia de datos, pero también es un ejercicio de lectura del mundo. A lo largo del libro voy a tratar el PIB de estos bloques como una variable continua que crece, fluctúa y responde a choques, y voy a usar cada técnica del temario para entender una parte distinta de ese fenómeno.

Cuando un ejemplo necesita datos reales, los descargo de fuentes abiertas, principalmente la API del Banco Mundial. Cuando uso datos sintéticos lo digo de forma explícita y explico por qué los genero.

## Cómo está organizado

El libro sigue los siete temas del bloque de sistemas continuos:

1. Métodos numéricos para la simulación continua. Cómo resolver en la computadora las ecuaciones diferenciales que describen sistemas que cambian de forma continua.
2. Modelos de series de tiempo. Cómo modelar una variable continua observada a lo largo del tiempo cuando no tenemos una ecuación del sistema, solo el histórico.
3. Aplicación a la Ciencia de Datos para Negocios. Cómo se conecta todo esto con una decisión real.
4. Identificación de un problema de variable continua. Cómo reconocer y formular bien el problema antes de modelar.
5. Identificación del mejor modelo para el fenómeno de estudio. Cómo elegir con criterio entre las alternativas.
6. Interpretación de los resultados del modelo. Cómo leer lo que el modelo dice sin engañarse.
7. Proposición de soluciones basadas en el modelo. Cómo pasar de la predicción a la recomendación.

Cierro con las referencias en formato APA 7 y un apéndice de reproducibilidad con el entorno y las versiones de las librerías.

## Stack

Todo el código es Python. Uso `numpy`, `scipy`, `pandas`, `statsmodels`, `matplotlib`, `wbgapi` para el Banco Mundial y `SALib` para el análisis de sensibilidad. El apéndice tiene el `requirements.txt` completo.
