# 5.1 Árbol de decisión para seleccionar estructura

Con el problema definido y los datos comprendidos, llega la pregunta del Tema 5: cuál es el mejor modelo para este fenómeno. La guía del curso ofrece un árbol de decisión que estructura esta elección, y este capítulo lo recorre aplicándolo al PIB de los bloques (Law, 2014). La idea central es que la selección de modelo no es cuestión de gusto ni de moda, es una consecuencia lógica de cómo respondo unas pocas preguntas sobre el problema.

## El árbol de la guía

La guía plantea el árbol así (Law, 2014):

```
¿Tienes datos históricos suficientes?
├── NO  -> Modelos basados en teoría (ODE, dinámica de sistemas, MC con expertos)
└── SI
     ¿Objetivo: predecir o entender el mecanismo?
     ├── PREDECIR -> ¿Relación lineal?
     │    ├── SI -> Regresión, modelos lineales
     │    └── NO -> Gradient Boosting, Redes Neuronales
     └── ENTENDER -> ¿Sistema continuo o discreto?
          ├── CONTINUO -> ¿Tienes ecuaciones del sistema?
          │    ├── SI -> ODE / PDE (scipy.integrate)
          │    └── NO -> Dinámica de sistemas
          └── DISCRETO -> ...
```

Lo recorro con nuestro caso. ¿Tengo datos históricos suficientes? Sí, tengo series anuales desde 1995, aunque cortas. ¿Mi objetivo es predecir o entender el mecanismo? Aquí está la bifurcación interesante, porque el caso del PIB necesita las dos cosas. Por eso voy a usar dos ramas del árbol en paralelo y compararlas, que es justo lo que hace el resto del tema.

## Cuando el objetivo es entender: la rama de la ODE

Si quiero entender por qué el bloque crece como crece, voy hacia el sistema continuo. Como puedo postular una ecuación, el motor logístico del Tema 1, caigo en la rama de ODE con `scipy.integrate`. El valor de esta rama es la interpretabilidad: la ODE logística me da dos parámetros con significado económico, la tasa intrínseca y el techo estructural. Responde preguntas de mecanismo, no de precisión predictiva.

## Cuando el objetivo es predecir: la rama de las series de tiempo

Si quiero pronosticar el valor del próximo año con la menor incertidumbre, el árbol me empuja hacia los modelos de predicción. Para una variable continua con dependencia temporal, eso son las series de tiempo del Tema 2. Aquí el árbol genérico de la guía se enriquece, porque entre los modelos predictivos para una serie continua tengo que elegir a su vez entre ARIMA, suavizamiento exponencial, GBM y modelos de aprendizaje automático.

## El sub-árbol específico de series de tiempo

Para el caso continuo predictivo construyo un sub-árbol propio, porque la guía deja esa rama abierta:

```
¿La serie tiene estacionalidad fuerte?
├── SI -> SARIMA o Holt-Winters
└── NO -> ¿Quiero un mecanismo de tendencia con freno o solo proyectar?
     ├── mecanismo con freno -> ODE logística (Tema 1)
     ├── proyección con incertidumbre y memoria -> ARIMA (Tema 2)
     ├── proceso multiplicativo con ruido -> GBM (Tema 7)
     └── relaciones no lineales con muchas features -> ML / boosting
```

El PIB anual de un bloque no tiene estacionalidad, tiene tendencia con cierta saturación, y quiero proyectar con incertidumbre. Eso me deja como candidatos principales la ODE logística, el ARIMA y el GBM, que son justo los tres que comparo de frente en el capítulo 5.3.

## La regla de la teoría contra los datos

Hay un principio del árbol que vale subrayar: cuando no hay datos suficientes, la guía manda usar modelos basados en teoría (Law, 2014). Esto explica por qué la ODE sigue siendo valiosa aun teniendo datos. Con solo dos décadas de observaciones anuales, un modelo puramente estadístico como un ARIMA de orden alto sobreajusta con facilidad. Un modelo teórico de dos parámetros, como la logística, incorpora conocimiento del dominio, la economía del crecimiento, que compensa la escasez de datos. La teoría es una forma de regularización cuando el dato escasea.

## El criterio de parsimonia atraviesa todo el árbol

Sea cual sea la rama, la guía impone una propiedad transversal: parsimonia, máxima fidelidad con el mínimo de parámetros, la navaja de Ockham (Law, 2014). Esto significa que entre dos modelos que ajustan parecido, gana el más simple. No porque lo simple sea elegante, sino porque lo simple generaliza mejor y es más fácil de explicar a quien decide. Toda la maquinaria de los criterios de información del siguiente capítulo, AIC y BIC, es una forma de operacionalizar esta navaja.

## Cierre

El árbol de decisión convierte la elección de modelo en una consecuencia de pocas preguntas: tengo datos, quiero entender o predecir, el sistema es continuo, tengo ecuaciones o no. Para el PIB de los bloques el árbol justifica usar tanto la ODE, por su interpretabilidad, como las series de tiempo, por su poder predictivo, y compararlas. La teoría regulariza cuando el dato escasea, y la parsimonia decide entre empates. Para resolver esos empates con rigor necesito una métrica que penalice la complejidad, y eso son los criterios de información AIC y BIC de la siguiente página.

## Referencias

Law, A. M. (2014). *Simulation modeling and analysis* (5a ed.). McGraw-Hill.

Hyndman, R. J., & Athanasopoulos, G. (2021). *Forecasting: Principles and practice* (3a ed.). OTexts. https://otexts.com/fpp3/
