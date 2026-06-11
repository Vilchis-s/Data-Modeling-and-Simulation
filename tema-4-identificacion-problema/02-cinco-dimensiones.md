# 4.2 Las cinco dimensiones de la definición del problema

La guía del curso es tajante: una pregunta bien formulada especifica cinco dimensiones, decisión, horizonte, alcance, criterio de éxito y restricciones (Law, 2014). En mi experiencia, la mayoría de los proyectos de ciencia de datos que fracasan no fracasan por el modelo, fracasan porque nunca definieron bien el problema. Este capítulo desarrolla cada dimensión aplicándola al caso del PIB, y muestra cómo una mala formulación se diagnostica por la dimensión que le falta.

## El enunciado vago como punto de partida

La guía da un ejemplo perfecto: alguien dice quiero predecir ventas, y la tarea del analista es ver que a esa frase le falta especificidad en todas las dimensiones (Law, 2014). El equivalente en nuestro caso sería quiero saber si los BRICS+ van a superar al G7. Suena claro, pero no lo es. Superar en qué métrica. Para cuándo. Con qué países en cada bloque. Qué tan precisa debe ser la respuesta. Con qué datos. Hasta que no respondo eso, no tengo un problema modelable, tengo una conversación de café.

## Dimensión 1. Decisión

Qué decisión se tomará con el modelo. Esta es la dimensión que le da sentido a todo lo demás, porque la precisión que necesito depende de qué está en juego. Para el caso, la decisión es la asignación de exposición de una cartera entre bloques a mediano plazo. No es una decisión académica de tener razón en un debate, es una decisión con consecuencias monetarias, y eso fija el estándar de calidad.

## Dimensión 2. Horizonte

Cuánto tiempo abarca el análisis. El horizonte condiciona la técnica y la incertidumbre tolerable. Para el caso fijo un horizonte de diez años. Es lo bastante largo para que la dinámica de la transición se manifieste, y lo bastante corto para que el intervalo de predicción no se vuelva inútilmente ancho. Como advertí en el capítulo 3.3, proyectar a treinta años con datos anuales de pocas décadas sería deshonesto, así que el horizonte no es un capricho, es una restricción técnica disfrazada de decisión de negocio.

## Dimensión 3. Alcance

Qué entidades y variables están dentro del modelo. Aquí decido que el alcance son los agregados de PIB de los dos bloques, no los países uno por uno, ni los sectores económicos, ni los flujos de comercio. También decido la composición exacta de cada bloque, que para BRICS+ incluye los miembros recientes y para G7 los siete clásicos. Definir el alcance es, sobre todo, definir qué dejo fuera, y ser explícito sobre eso evita que después me reclamen por algo que conscientemente excluí.

## Dimensión 4. Criterio de éxito

Cómo se medirá si el modelo es bueno, con una métrica y un umbral (Law, 2014). Esta es la dimensión que más se omite y la que más duele omitir. Para el caso, mi criterio es doble: un MAPE fuera de muestra por debajo de cierto umbral en el backtesting, y un intervalo de predicción cuyo ancho relativo no supere un límite que vuelva la recomendación accionable. Sin un umbral definido de antemano, cualquier resultado parece aceptable a posteriori, que es la trampa clásica del analista que ajusta el criterio para que su modelo apruebe.

```python
# El criterio de éxito se fija ANTES de modelar, no después
criterio = {
    "mape_max": 0.08,           # error porcentual fuera de muestra aceptable
    "ancho_ic_relativo_max": 0.30,  # el IC al 90% no debe exceder 30% del valor
}
```

## Dimensión 5. Restricciones

Las limitaciones de datos, tiempo y recursos. Para el caso, la restricción dominante es el dato: solo dispongo de series anuales públicas del Banco Mundial, con poco más de dos décadas de historia comparable para el bloque ampliado, y con la complicación de que algunos miembros tienen huecos o están afectados por sanciones que distorsionan sus cifras. Reconocer esta restricción desde el inicio evita prometer una precisión que el dato no puede sostener.

## El test de sensibilidad informal de los supuestos

La guía propone una pregunta poderosa para cada supuesto: si este supuesto estuviera completamente equivocado, cambiaría sustancialmente la conclusión (Law, 2014). Si la respuesta es sí, el supuesto es crítico y hay que validarlo con datos; si es no, es secundario y se documenta como simplificación.

Aplico el test a los supuestos del caso. El supuesto de la unidad de medida, dólares corrientes contra PPP, es crítico: cambia la conclusión sobre el cruce, así que lo trato con cuidado y reporto ambas medidas. El supuesto de la composición exacta del bloque es secundario: agregar o quitar a Etiopía mueve poco el agregado dominado por China, así que lo documento y sigo. Distinguir críticos de secundarios es lo que me dice dónde gastar mi esfuerzo de validación.

## Cierre

Las cinco dimensiones, decisión, horizonte, alcance, criterio de éxito y restricciones, convierten una pregunta vaga en un problema modelable, y la dimensión faltante es siempre el diagnóstico de una mala formulación. El test de sensibilidad informal separa los supuestos críticos de los secundarios y enfoca el esfuerzo. Con el problema bien definido, el siguiente paso es mirar de verdad los datos antes de modelar, el análisis exploratorio de una variable continua, que desarrollo a continuación.

## Referencias

Law, A. M. (2014). *Simulation modeling and analysis* (5a ed.). McGraw-Hill.

Provost, F., & Fawcett, T. (2013). *Data science for business: What you need to know about data mining and data-analytic thinking*. O'Reilly Media.
