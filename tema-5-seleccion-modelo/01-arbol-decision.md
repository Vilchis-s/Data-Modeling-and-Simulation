# 5.1 Árbol de decisión para seleccionar estructura

Con el problema definido y los datos comprendidos, llega la pregunta del Tema 5: cuál es el mejor modelo para este fenómeno. La guía de estudio ofrece un árbol de decisión que estructura esta elección, y este capítulo lo recorre aplicándolo al PIB de los bloques. La idea central es que la selección de modelo no es cuestión de gusto ni de moda, sino una consecuencia lógica de cómo se responden unas pocas preguntas sobre el problema.

## El árbol de la guía

La guía de estudio plantea el árbol de la siguiente forma:

```
¿Hay datos históricos suficientes?
├── NO  -> Modelos basados en teoría (ODE, dinámica de sistemas, MC con expertos)
└── SI
     ¿Objetivo: predecir o comprender el mecanismo?
     ├── PREDECIR -> ¿Relación lineal?
     │    ├── SI -> Regresión, modelos lineales
     │    └── NO -> Gradient Boosting, Redes Neuronales
     └── COMPRENDER -> ¿Sistema continuo o discreto?
          ├── CONTINUO -> ¿Hay ecuaciones del sistema?
          │    ├── SI -> ODE / PDE (scipy.integrate)
          │    └── NO -> Dinámica de sistemas
          └── DISCRETO -> ...
```

Se recorre con el caso. ¿Hay datos históricos suficientes? Sí, se dispone de series anuales desde 1995, aunque cortas. ¿El objetivo es predecir o comprender el mecanismo? Aquí está la bifurcación de interés, pues el caso del PIB requiere ambas cosas. Por ello se utilizan dos ramas del árbol en paralelo y se comparan, que es precisamente lo que hace el resto del tema.

## Cuando el objetivo es comprender: la rama de la ODE

Si se busca comprender por qué el bloque crece como crece, el camino conduce al sistema continuo. Como puede postularse una ecuación, el motor logístico del Tema 1, se cae en la rama de ODE con `scipy.integrate`. El valor de esta rama es la interpretabilidad: la ODE logística ofrece dos parámetros con significado económico, la tasa intrínseca y el techo estructural. Responde preguntas de mecanismo, no de precisión predictiva.

## Cuando el objetivo es predecir: la rama de las series de tiempo

Si se busca pronosticar el valor del próximo año con la menor incertidumbre, el árbol conduce a los modelos de predicción. Para una variable continua con dependencia temporal, eso son las series de tiempo del Tema 2. Aquí el árbol genérico de la guía de estudio se enriquece, pues entre los modelos predictivos para una serie continua debe elegirse, a su vez, entre ARIMA, suavizamiento exponencial, GBM y modelos de aprendizaje automático.

## El sub-árbol específico de series de tiempo

Para el caso continuo predictivo se construye un sub-árbol propio, pues la guía de estudio deja esa rama abierta:

```
¿La serie tiene estacionalidad fuerte?
├── SI -> SARIMA o Holt-Winters
└── NO -> ¿Se busca un mecanismo de tendencia con freno o solo proyectar?
     ├── mecanismo con freno -> ODE logística (Tema 1)
     ├── proyección con incertidumbre y memoria -> ARIMA (Tema 2)
     ├── proceso multiplicativo con ruido -> GBM (Tema 7)
     └── relaciones no lineales con muchas features -> ML / boosting
```

El PIB anual de un bloque no tiene estacionalidad, tiene tendencia con cierta saturación, y se busca proyectar con incertidumbre. Eso deja como candidatos principales la ODE logística, el ARIMA y el GBM, que son precisamente los tres que se comparan directamente en el capítulo 5.3 (tema-5-seleccion-modelo/03-comparacion-familias.md).

## La regla de la teoría frente a los datos

Conviene subrayar un principio del árbol: cuando no hay datos suficientes, la guía de estudio recomienda usar modelos basados en teoría. Esto explica por qué la ODE sigue siendo valiosa aun disponiendo de datos. Con solo dos décadas de observaciones anuales, un modelo puramente estadístico como un ARIMA de orden alto sobreajusta con facilidad. Un modelo teórico de dos parámetros, como la logística, incorpora conocimiento del dominio, la economía del crecimiento, que compensa la escasez de datos. La teoría es una forma de regularización cuando el dato escasea.

## El criterio de parsimonia atraviesa todo el árbol

Sea cual sea la rama, la guía de estudio impone una propiedad transversal: parsimonia, máxima fidelidad con el mínimo de parámetros, la navaja de Ockham. Esto significa que entre dos modelos que ajustan de forma similar, gana el más simple. No por elegancia, sino porque lo simple generaliza mejor y es más fácil de explicar a quien decide. Toda la maquinaria de los criterios de información del siguiente capítulo, AIC y BIC, es una forma de operacionalizar esta navaja.

## Bibliografía

El árbol de decisión convierte la elección de modelo en una consecuencia de pocas preguntas: hay datos, se busca comprender o predecir, el sistema es continuo, hay ecuaciones o no. Para el PIB de los bloques, el árbol justifica usar tanto la ODE, por su interpretabilidad, como las series de tiempo, por su poder predictivo, y compararlas. La teoría regulariza cuando el dato escasea, y la parsimonia decide entre empates. Para resolver esos empates con rigor se requiere una métrica que penalice la complejidad, y eso son los criterios de información AIC y BIC de la siguiente página.

## Referencias

1. Hyndman, R. J., & Athanasopoulos, G. (2021). *Forecasting: Principles and practice* (3a ed.). OTexts. https://otexts.com/fpp3/
