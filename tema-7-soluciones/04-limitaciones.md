# 7.4 Limitaciones y trabajo futuro

El trabajo cierra con el capítulo de mayor importancia metodológica, pues declarar las limitaciones es lo que distingue un trabajo serio de una pieza de propaganda. Un modelo cuyo autor no reconoce sus límites no es confiable, por bueno que sea su código. A continuación se enumeran con honestidad las cosas que este trabajo no puede hacer y se traza cómo se mejoraría, vinculándolo con los tres tipos de incertidumbre que recorrieron el documento.

## Limitaciones de los datos

La restricción más severa, fijada desde el Tema 4, es el dato. Las series anuales del Banco Mundial tienen baja resolución temporal y pocas décadas de historia comparable para el bloque ampliado. Con dos décadas de observaciones anuales, cualquier modelo estima sus parámetros sobre una muestra pequeña, lo que vuelve las estimaciones frágiles y los intervalos amplios. Un trabajo futuro usaría datos trimestrales donde existan, integraría fuentes como el FMI y bases nacionales, y trataría los huecos de miembros como Rusia e Irán con métodos de imputación en lugar de dejarlos absorber por el agregado.

## Limitaciones de la unidad de medida

Todo el análisis depende de una elección que no es técnica, sino interpretativa: medir en dólares corrientes o en paridad de poder adquisitivo, como se discutió en el capítulo 6.4 (tema-6-interpretacion/04-caso-lectura.md). Las dos medidas dan conclusiones distintas sobre el cruce, y ninguna es la verdadera, pues responden preguntas distintas. Se reportaron ambas, pero conviene reconocer que la conclusión geopolítica es sensible a esta elección, y que un lector con otra postura sobre qué mide mejor el poder económico podría leer los mismos datos de otra forma. Un trabajo futuro construiría una medida compuesta que pondere ambas según el propósito de la decisión.

## Limitaciones de los modelos

Cada familia utilizada carga supuestos que el fenómeno real viola.

La ODE logística es determinista e ignora todos los choques, las crisis y la política. Su techo K se estima con enorme incertidumbre cuando el sistema aún no satura, como mostró el análisis de sensibilidad del capítulo 6.3 (tema-6-interpretacion/03-sensibilidad.md).

El ARIMA supone que la estructura de dependencia temporal del pasado se mantiene en el futuro, lo cual falla justamente cuando hay un cambio de régimen, que es lo más interesante de predecir.

El GBM supone drift y volatilidad constantes, lo que subestima la posibilidad de saltos abruptos como los de una crisis o una sanción.

Ninguno de los tres captura la heterogeneidad interna del bloque, dominado por una economía, ni un cambio de composición, ni una crisis sistémica. Esto es la incertidumbre de modelo que la guía de estudio señala como la más peligrosa por ser invisible a cualquier análisis de sensibilidad.

## El triángulo de la simulación, revisitado

Conviene cerrar volviendo al triángulo de la guía de estudio, sistema real, modelo, simulación. El trabajo recorrió el triángulo entero: del sistema real, la economía de los bloques, al modelo, ODE y series de tiempo, a la simulación, el código de Python, y de vuelta a validar contra el dato. Pero el triángulo nunca se cierra del todo, pues el modelo siempre es una abstracción que deja fuera parte del sistema. Reconocer ese hueco no es un defecto del trabajo, sino comprender qué es un modelo: una representación útil y deliberadamente incompleta, no una copia de la realidad.

## Trabajo futuro concreto

De continuar este proyecto, en orden de prioridad, correspondería lo siguiente.

Incorporar modelos que admitan cambios de régimen, como los modelos de Markov-switching, que combinan la cadena de Markov del temario discreto con la dinámica continua, para capturar las transiciones entre régimen de crecimiento alto y de crisis que un ARIMA fijo no puede.

Modelar los bloques como un sistema acoplado de ecuaciones diferenciales, no como dos series independientes, retomando el modelo de dos bloques que competían por un techo común del capítulo 1.7 (tema-1-metodos-numericos/07-solve-ivp.md), para capturar que el ascenso de uno presiona al otro.

Añadir incertidumbre bayesiana sobre los parámetros con MCMC, el método del temario, para que la incertidumbre de los parámetros entre de forma explícita en los intervalos de predicción, en lugar de tratarlos como conocidos.

Conectar el modelo de PIB con variables explicativas provenientes del NLP: construir un índice de tensión geopolítica a partir de noticias y discursos con embeddings y topic modeling, y probar si anticipa los choques que los modelos puramente temporales no ven venir. Esta es la línea en la que mi experiencia en AI engineering se cruzaría de forma directa con este bloque del curso, y la que considero más prometedora para una continuación.

## Bibliografía del trabajo

Este trabajo de investigación recorrió los siete temas de los sistemas continuos tomando como hilo una sola pregunta de relevancia contemporánea: la transición de poder económico entre el bloque BRICS+ y el G7. Se construyó la maquinaria numérica para resolver ecuaciones diferenciales, se modelaron series de tiempo, se conectó todo con decisiones de negocio, se formuló el problema con rigor, se seleccionaron modelos con criterio, se interpretaron los resultados sin exagerar y se propusieron recomendaciones que respetan la incertidumbre y la función de pérdida de quien decide.

La conclusión metodológica de fondo es que el rigor técnico y el interés por el mundo no se estorban. Tomar en serio un proceso geopolítico de esta magnitud significó, precisamente, medirlo con honestidad, reportar lo que se desconoce y resistir que el modelo confirmara lo previamente creído. Un modelo es una herramienta para comprender mejor, no para validar una creencia previa, y esa disciplina es la que se procuró practicar en cada página.

## Referencias

1. Hamilton, J. D. (1989). A new approach to the economic analysis of nonstationary time series and the business cycle. *Econometrica, 57*(2), 357-384. https://doi.org/10.2307/1912559
2. Box, G. E. P. (1976). Science and statistics. *Journal of the American Statistical Association, 71*(356), 791-799. https://doi.org/10.1080/01621459.1976.10480949
