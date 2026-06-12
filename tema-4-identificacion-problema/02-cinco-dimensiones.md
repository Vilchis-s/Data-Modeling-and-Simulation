# 4.2 Las cinco dimensiones de la definición del problema

La guía de estudio es tajante: una pregunta bien formulada especifica cinco dimensiones, decisión, horizonte, alcance, criterio de éxito y restricciones. La mayoría de los proyectos de ciencia de datos que fracasan no lo hacen por el modelo, sino porque nunca definieron bien el problema. Este capítulo desarrolla cada dimensión aplicándola al caso del PIB, y muestra cómo una mala formulación se diagnostica por la dimensión que le falta.

## El enunciado vago como punto de partida

La guía de estudio ofrece un ejemplo ilustrativo: alguien plantea quiero predecir ventas, y la tarea del analista consiste en advertir que a esa frase le falta especificidad en todas las dimensiones. El equivalente en el caso sería quiero saber si los BRICS+ van a superar al G7. Parece claro, pero no lo es. Superar en qué métrica. Para cuándo. Con qué países en cada bloque. Con qué precisión debe responderse. Con qué datos. Hasta que no se responde eso, no hay un problema modelable, sino una conversación informal.

## Dimensión 1. Decisión

Qué decisión se tomará con el modelo. Esta es la dimensión que da sentido a todo lo demás, pues la precisión requerida depende de qué está en juego. Para el caso, la decisión es la asignación de exposición de una cartera entre bloques a mediano plazo. No es una decisión académica de tener razón en un debate, sino una decisión con consecuencias monetarias, y eso fija el estándar de calidad.

## Dimensión 2. Horizonte

Cuánto tiempo abarca el análisis. El horizonte condiciona la técnica y la incertidumbre tolerable. Para el caso se fija un horizonte de diez años. Es suficiente para que la dinámica de la transición se manifieste, y lo bastante corto para que el intervalo de predicción no se vuelva inútilmente ancho. Como se advirtió en el capítulo 3.3 (tema-3-aplicacion-negocios/03-forecasting-decision.md), proyectar a treinta años con datos anuales de pocas décadas sería deshonesto, de modo que el horizonte no es un capricho, sino una restricción técnica disfrazada de decisión de negocio.

## Dimensión 3. Alcance

Qué entidades y variables están dentro del modelo. Aquí se decide que el alcance son los agregados de PIB de los dos bloques, no los países uno por uno, ni los sectores económicos, ni los flujos de comercio. También se decide la composición exacta de cada bloque, que para BRICS+ incluye los miembros recientes y para G7 los siete clásicos. Definir el alcance es, sobre todo, definir qué se deja fuera, y ser explícito al respecto evita reclamaciones posteriores por algo conscientemente excluido.

## Dimensión 4. Criterio de éxito

Cómo se medirá si el modelo es bueno, con una métrica y un umbral. Esta es la dimensión que más se omite y la que más perjudica omitir. Para el caso, el criterio es doble: un MAPE fuera de muestra por debajo de cierto umbral en el backtesting, y un intervalo de predicción cuyo ancho relativo no supere un límite que conserve la recomendación accionable. El criterio se fija de antemano, no a posteriori para que el modelo apruebe.

```python
# El criterio de éxito se fija ANTES de modelar, no después
criterio = {
    "mape_max": 0.08,           # error porcentual fuera de muestra aceptable
    "ancho_ic_relativo_max": 0.30,  # el IC al 90% no debe exceder 30% del valor
}
```

Sin un umbral definido de antemano, cualquier resultado parece aceptable a posteriori, que es la trampa clásica del analista que ajusta el criterio para que su modelo apruebe.

## Dimensión 5. Restricciones

Las limitaciones de datos, tiempo y recursos. Para el caso, la restricción dominante es el dato: solo se dispone de series anuales públicas del Banco Mundial, con poco más de dos décadas de historia comparable para el bloque ampliado, y con la complicación de que algunos miembros tienen huecos o están afectados por sanciones que distorsionan sus cifras. Reconocer esta restricción desde el inicio evita prometer una precisión que el dato no puede sostener.

## El test de sensibilidad informal de los supuestos

La guía de estudio propone una pregunta poderosa para cada supuesto: si este supuesto estuviera completamente equivocado, cambiaría sustancialmente la conclusión. Si la respuesta es sí, el supuesto es crítico y debe validarse con datos; si es no, es secundario y se documenta como simplificación.

El test se aplica a los supuestos del caso. El supuesto de la unidad de medida, dólares corrientes frente a PPP, es crítico: cambia la conclusión sobre el cruce, de modo que se trata con cuidado y se reportan ambas medidas. El supuesto de la composición exacta del bloque es secundario: agregar o quitar a Etiopía mueve poco el agregado dominado por China, de modo que se documenta y se prosigue. Distinguir críticos de secundarios es lo que indica dónde concentrar el esfuerzo de validación.

## Bibliografía

Las cinco dimensiones, decisión, horizonte, alcance, criterio de éxito y restricciones, convierten una pregunta vaga en un problema modelable, y la dimensión faltante es siempre el diagnóstico de una mala formulación. El test de sensibilidad informal separa los supuestos críticos de los secundarios y enfoca el esfuerzo. Con el problema bien definido, el siguiente paso es examinar los datos antes de modelar, el análisis exploratorio de una variable continua, que se desarrolla a continuación.

