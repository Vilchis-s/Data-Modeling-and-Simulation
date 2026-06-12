# 7.1 De la predicción a la recomendación

El último tema cierra el círculo y es, posiblemente, el que más distingue a un científico de datos de un analista que solo ejecuta modelos. Un pronóstico bien interpretado todavía no es una recomendación. La recomendación es la formulación que indica qué hacer, asumiendo el costo de equivocarse. Este capítulo trata cómo dar ese salto sin perder el rigor construido en los seis temas anteriores.

## La brecha entre predecir y recomendar

Predecir responde qué va a ocurrir. Recomendar responde qué debe hacerse dado lo que va a ocurrir y lo que cuesta equivocarse. Entre ambas media una función de pérdida que es responsabilidad de quien decide, no del modelo. El mismo pronóstico justifica recomendaciones opuestas según el apetito de riesgo: un fondo agresivo y un fondo de pensiones leen el mismo cono de incertidumbre y actúan distinto, y ambos pueden tener razón.

La guía de estudio lo enmarca con su flujo de tres pasos: el aprendizaje automático predice, la simulación genera escenarios, y la optimización encuentra la mejor política dentro de esos escenarios. La recomendación reside en el tercer paso. Hasta ahora el trabajo realizó los dos primeros; este tema realiza el tercero.

## La regla de oro de comunicar resultados

La guía de estudio incluye una tabla que sintetiza cómo no comunicar y cómo sí. Se traslada al caso.

```
Incorrecto                                  Correcto
"El PIB de BRICS+ será de X en 2035"        "El PIB central es X, IC90 [a, b]"
"BRICS+ supera al G7"                        "En PPP ya supera; en USD el cruce es incierto"
"El bloque crecerá 6% anual"                 "Crecimiento medio 6%, con años de crisis posibles"
```

Una recomendación que no carga la incertidumbre no es una recomendación profesional, sino una apuesta disfrazada de certeza. Toda recomendación de este tema lleva su intervalo y su probabilidad, fiel a esta regla.

## El formato de una recomendación accionable

Una buena recomendación tiene cuatro partes, que sirven de plantilla.

La acción: qué hacer concretamente, un verbo, no una observación. Aumentar, mantener, reducir, esperar.

La justificación cuantitativa: el resultado del modelo que la respalda, con su incertidumbre. No un número aislado.

La condición: bajo qué supuestos vale, y qué la invalidaría. Una recomendación sin condiciones es frágil.

El monitoreo: qué señal vigilar para saber si debe revisarse. Una recomendación viva, no de una sola vez.

```python
recomendacion = {
    "accion": "Aumentar de forma gradual la exposición al bloque BRICS+",
    "justificacion": "Crecimiento proyectado superior al G7 con alta probabilidad en PPP",
    "condicion": "Válida si no hay crisis sistémica ni cambio mayor de composición del bloque",
    "monitoreo": "Revisar si el dato anual cae fuera del IC90 dos años seguidos",
}
```

## La asimetría de los errores

Un punto que conviene tomar en serio: los errores no cuestan lo mismo en ambas direcciones. Recomendar aumentar exposición y equivocarse no cuesta lo mismo que recomendar no hacerlo y perder la oportunidad. Esta asimetría es la función de pérdida, y debe entrar de forma explícita en la recomendación. La guía de estudio lo ilustra con el ejemplo de los agentes de un call center, donde el valor marginal de un nivel de servicio adicional no justifica su costo. La misma lógica aplica a la cartera: el valor marginal de capturar el último punto de crecimiento del bloque puede no justificar el riesgo de concentración que implica.

## La humildad del modelo en la recomendación

Conviene ser claro sobre el alcance de la herramienta. El modelo informa la recomendación, no la dicta. La decisión final sobre una cartera, una política pública o una alianza pondera factores que ningún modelo de PIB captura: política, instituciones, riesgo geopolítico, valores. Al recomendar, la función del análisis es poner la mejor evidencia cuantitativa sobre la mesa con su incertidumbre, no pretender que el modelo reemplaza el juicio de quien carga la responsabilidad. Esa humildad no es debilidad técnica, sino honestidad sobre el alcance de la herramienta.

## Bibliografía

Pasar de la predicción a la recomendación es cruzar la brecha entre qué ocurrirá y qué hacer, una brecha que llena la función de pérdida de quien decide. Una recomendación accionable carga acción, justificación cuantitativa con incertidumbre, condición y monitoreo, respeta la asimetría de los errores y reconoce que el modelo informa pero no dicta. Para construir recomendaciones robustas se requiere observar cómo se comporta la decisión bajo distintos futuros, lo que exige simular escenarios, tema de la siguiente página.

