# Sistemas Continuos: Modelado y Simulación

Modelado y Simulación de Datos, sexto semestre. Licenciatura en Ciencia de Datos para Negocios.

El presente trabajo de investigación desarrolla el bloque de sistemas continuos del curso. Parte de la redacción tiene influencia de mi reciente experiencia laboral, que me encuentro acumulando en ya más de un año y medio como Científico de Datos junior trabajando en NLP contemporáneo, pipelines de embeddings, topic modeling y agentes bajo la supervisión de seniors, ámbito desde el cual, para esta materia, se buscó enlazar todo lo anterior con el terreno más clásico de las ecuaciones diferenciales, las series de tiempo y la simulación, bajo el hilo conductor de temas de geopolítica: los BRICS+ countries.

## El hilo conductor

El trabajo no recurre a ejemplos genéricos de demostración. Todo el desarrollo gira en torno a una sola pregunta de relevancia contemporánea: cómo evoluciona el Producto Interno Bruto del bloque BRICS+ frente al G7, y qué aportan los modelos continuos a la comprensión de la transición de poder económico que se discute en la actualidad.

La elección no es neutral, y el trabajo no pretende presentarla como tal. El ascenso de los BRICS+ constituye uno de los procesos geopolíticos más relevantes de la década. Medir, modelar y proyectar ese ascenso es un ejercicio de ciencia de datos, pero también una lectura del orden internacional. A lo largo del documento, el PIB de estos bloques se trata como una variable continua que crece, fluctúa y responde a choques, y cada técnica del temario se emplea para comprender una faceta distinta de ese fenómeno.

Cuando un ejemplo requiere datos reales, estos se descargan de fuentes abiertas, principalmente la API del Banco Mundial. Cuando se utilizan datos sintéticos, se indica de forma explícita y se justifica su uso.

## Cómo está organizado

El trabajo de investigación sigue los siete temas del bloque de sistemas continuos:

1. Métodos numéricos para la simulación continua. Cómo resolver computacionalmente las ecuaciones diferenciales que describen sistemas que cambian de forma continua.
2. Modelos de series de tiempo. Cómo modelar una variable continua observada a lo largo del tiempo cuando no se dispone de una ecuación del sistema, solo del histórico.
3. Aplicación a la Ciencia de Datos para Negocios. Cómo se vincula todo lo anterior con una decisión real.
4. Identificación de un problema de variable continua. Cómo reconocer y formular el problema antes de modelar.
5. Identificación del mejor modelo para el fenómeno de estudio. Cómo elegir con criterio entre las alternativas.
6. Interpretación de los resultados del modelo. Cómo leer lo que el modelo indica sin incurrir en error.
7. Proposición de soluciones basadas en el modelo. Cómo pasar de la predicción a la recomendación.

El documento cierra con las referencias en formato APA 7 y un apéndice de reproducibilidad con el entorno y las versiones de las librerías.

## Stack

Todo el código está escrito en Python. Se emplean `numpy`, `scipy`, `pandas`, `statsmodels`, `matplotlib`, `wbgapi` para el Banco Mundial y `SALib` para el análisis de sensibilidad. El apéndice incluye el `requirements.txt` completo.
