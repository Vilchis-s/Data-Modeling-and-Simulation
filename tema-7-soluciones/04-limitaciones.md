# 7.4 Limitaciones y trabajo futuro

Cierro el libro con el capítulo que más respeto, porque declarar las limitaciones es lo que separa un trabajo serio de una pieza de propaganda. Un modelo cuyo autor no reconoce sus límites no es confiable, por bueno que sea su código. Aquí enumero con honestidad lo que este trabajo no puede hacer y trazo cómo se mejoraría, conectándolo con los tres tipos de incertidumbre que recorrieron el libro.

## Limitaciones de los datos

La restricción más dura, fijada desde el Tema 4, es el dato. Las series anuales del Banco Mundial tienen baja resolución temporal y pocas décadas de historia comparable para el bloque ampliado. Con dos décadas de observaciones anuales, cualquier modelo está estimando sus parámetros sobre una muestra pequeña, lo que vuelve las estimaciones frágiles y los intervalos amplios. Un trabajo futuro usaría datos trimestrales donde existan, integraría fuentes como el FMI y bases nacionales, y trataría los huecos de miembros como Rusia e Irán con métodos de imputación en lugar de dejarlos absorber por el agregado.

## Limitaciones de la unidad de medida

Todo el análisis depende de una elección que no es técnica sino interpretativa: medir en dólares corrientes o en paridad de poder adquisitivo, como discutí en el capítulo 6.4. Las dos medidas dan conclusiones distintas sobre el cruce, y ninguna es la verdadera, porque responden preguntas distintas. Reporté ambas, pero reconozco que la conclusión geopolítica es sensible a esta elección, y que un lector con otra postura sobre qué mide mejor el poder económico podría leer los mismos datos de otra forma. Un trabajo futuro construiría una medida compuesta que pondere ambas según el propósito de la decisión.

## Limitaciones de los modelos

Cada familia que usé carga supuestos que el fenómeno real viola.

La ODE logística es determinista e ignora todos los choques, las crisis y la política. Su techo K se estima con enorme incertidumbre cuando el sistema aún no satura, como mostró el análisis de sensibilidad del capítulo 6.3.

El ARIMA supone que la estructura de dependencia temporal del pasado se mantiene en el futuro, lo cual falla justamente cuando hay un cambio de régimen, que es lo más interesante de predecir.

El GBM supone drift y volatilidad constantes, lo que subestima la posibilidad de saltos abruptos como los de una crisis o una sanción.

Ninguno de los tres captura la heterogeneidad interna del bloque, dominado por una economía, ni un cambio de composición, ni una crisis sistémica. Esto es la incertidumbre de modelo que la guía marca como la más peligrosa porque es invisible a cualquier análisis de sensibilidad (Law, 2014).

## El triángulo de la simulación, revisitado

Vale cerrar volviendo al triángulo de la guía, sistema real, modelo, simulación (Law, 2014). El libro recorrió el triángulo entero: del sistema real, la economía de los bloques, al modelo, ODE y series de tiempo, a la simulación, el código de Python, y de vuelta a validar contra el dato. Pero el triángulo nunca se cierra del todo, porque el modelo siempre es una abstracción que deja fuera parte del sistema. Reconocer ese hueco no es un defecto del trabajo, es entender qué es un modelo: una representación útil y deliberadamente incompleta, no una copia de la realidad.

## Trabajo futuro concreto

Si continuara este proyecto, en orden de prioridad, haría lo siguiente.

Incorporar modelos que admitan cambios de régimen, como los modelos de Markov-switching, que combinan la cadena de Markov del temario discreto con la dinámica continua, para capturar las transiciones entre régimen de crecimiento alto y de crisis que un ARIMA fijo no puede.

Modelar los bloques como un sistema acoplado de ecuaciones diferenciales, no como dos series independientes, retomando el modelo de dos bloques que competían por un techo común del capítulo 1.7, para capturar que el ascenso de uno presiona al otro.

Añadir incertidumbre bayesiana sobre los parámetros con MCMC, el método del temario, para que la incertidumbre de los parámetros entre de forma explícita en los intervalos de predicción, en lugar de tratarlos como conocidos.

Conectar el modelo de PIB con variables explicativas que vienen de mi terreno, el NLP: construir un índice de tensión geopolítica a partir de noticias y discursos con embeddings y topic modeling, y probar si anticipa los choques que los modelos puramente temporales no ven venir. Esa es la línea donde mi experiencia en AI engineering se cruzaría de verdad con este bloque del curso, y la que más me gustaría explorar.

## Cierre del libro

Este libro recorrió los siete temas de los sistemas continuos tomando como hilo una sola pregunta que me importa: la transición de poder económico entre el bloque BRICS+ y el G7. Construí la maquinaria numérica para resolver ecuaciones diferenciales, modelé series de tiempo, conecté todo con decisiones de negocio, formulé el problema con rigor, seleccioné modelos con criterio, interpreté los resultados sin exagerar y propuse recomendaciones que respetan la incertidumbre y la función de pérdida de quien decide.

Si algo me llevo del ejercicio es que el rigor técnico y el interés por el mundo no se estorban. Tomar en serio un proceso geopolítico de esta magnitud significó, precisamente, medirlo con honestidad, reportar lo que no sé y resistir que el modelo dijera lo que yo quería oír. Un modelo es una herramienta para entender mejor, no para confirmar lo que ya creía, y esa disciplina es lo que intenté practicar en cada página.

## Referencias

Law, A. M. (2014). *Simulation modeling and analysis* (5a ed.). McGraw-Hill.

Hamilton, J. D. (1989). A new approach to the economic analysis of nonstationary time series and the business cycle. *Econometrica, 57*(2), 357-384. https://doi.org/10.2307/1912559

Box, G. E. P. (1976). Science and statistics. *Journal of the American Statistical Association, 71*(356), 791-799. https://doi.org/10.1080/01621459.1976.10480949
