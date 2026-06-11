# 4.4 Caso: definición formal del problema del PIB

Cierro el Tema 4 escribiendo lo que la guía llama el documento de definición del problema, el producto de la etapa 1 (Law, 2014). Es el documento que, en un proyecto real, firmaría con el cliente antes de tocar una sola línea de modelado, porque fija el contrato de qué se va a responder y cómo se va a juzgar. Lo presento completo para el caso del PIB de BRICS+ frente a G7, integrando las cinco dimensiones y el EDA de los capítulos anteriores.

## Documento de definición del problema

### Título

Proyección de la trayectoria relativa del PIB de los bloques BRICS+ y G7 para informar una decisión de asignación de exposición a mediano plazo.

### Decisión

El modelo informa una decisión de asignación de cartera entre exposición a economías del bloque BRICS+ y del G7. En concreto, si la evidencia cuantitativa respalda aumentar, mantener o reducir el peso del bloque emergente en un horizonte de diez años. La decisión la toma un comité de estrategia; el modelo aporta la base cuantitativa, no la decisión final, que también pondera factores políticos no modelados.

### Horizonte

Diez años de proyección, con datos históricos desde 1995 para asegurar comparabilidad del bloque ampliado. El horizonte se justifica porque es suficiente para que la dinámica de la transición se exprese y porque excede el rango donde el intervalo de predicción se vuelve inutilizable con datos anuales.

### Alcance

Dentro del modelo están los agregados de PIB de los dos bloques. La composición de BRICS+ es Brasil, Rusia, India, China, Sudáfrica, Egipto, Etiopía, Irán y Emiratos Árabes Unidos. La de G7 es Estados Unidos, Japón, Alemania, Reino Unido, Francia, Italia y Canadá. Fuera del modelo quedan los países individuales, los sectores económicos, los flujos de comercio, y los factores geopolíticos cualitativos como conflictos o cambios de alianzas, que se consideran como contexto pero no se modelan.

### Criterio de éxito

El modelo se considera válido si en el backtesting con ventana temporal alcanza un MAPE fuera de muestra menor a 8 por ciento para cada bloque, y si el intervalo de predicción al 90 por ciento tiene un ancho relativo que mantenga la recomendación accionable. El criterio se fija antes de modelar y no se ajusta a posteriori para que el modelo apruebe.

### Restricciones

Solo se usan datos públicos anuales del Banco Mundial, lo que implica baja resolución temporal y pocas décadas de historia comparable. Algunos miembros del bloque tienen huecos en los datos o cifras afectadas por sanciones, en particular Rusia e Irán. La unidad de medida base es el dólar corriente, con la advertencia explícita de que la medición en paridad de poder adquisitivo arroja conclusiones distintas, por lo que ambas se reportarán.

## Supuestos y su criticidad

Aplico el test de sensibilidad informal a los supuestos principales (Law, 2014).

```
Supuesto                                  Criticidad   Tratamiento
Unidad de medida (USD corriente vs PPP)   CRÍTICO      Reportar ambas medidas
Composición fija de los bloques           Secundario   Documentar y proceder
No hay colapso sistémico en el horizonte  CRÍTICO      Acotar el horizonte; escenarios
Datos del Banco Mundial son fiables        Secundario   Cruzar con FMI si hay duda
La dinámica pasada informa la futura       CRÍTICO      Validar con backtesting
```

Los supuestos críticos son los que, de estar equivocados, cambiarían la conclusión, y por eso reciben tratamiento explícito. Los secundarios se documentan como simplificaciones.

## Resumen del EDA que respalda la definición

```python
import wbgapi as wb
import numpy as np

def perfil_bloque(paises, nombre):
    df = wb.data.DataFrame("NY.GDP.MKTP.CD", paises, time=range(1995, 2023))
    df.columns = [int(c.replace("YR", "")) for c in df.columns]
    pib = (df.sum(axis=0) / 1e12).sort_index()
    g = pib.pct_change().dropna()
    print(f"{nombre}: crecimiento medio {g.mean():.1%}, "
          f"volatilidad {g.std():.1%}, "
          f"años negativos {(g < 0).sum()}")

perfil_bloque(["BRA", "RUS", "IND", "CHN", "ZAF", "EGY", "ETH", "IRN", "ARE"], "BRICS+")
perfil_bloque(["USA", "JPN", "DEU", "GBR", "FRA", "ITA", "CAN"], "G7")
```

El perfil confirma lo que la definición supone: el bloque BRICS+ crece a un ritmo medio mayor y con más volatilidad que el G7, que crece despacio y de forma más estable. Esa diferencia de régimen, crecimiento alto y volátil contra crecimiento bajo y estable, es la base de toda la transición que el modelo va a proyectar.

## Por qué este documento vale el esfuerzo

En el mundo del AI engineering, donde vengo trabajando, la disciplina equivalente es escribir bien la especificación de un sistema antes de construirlo. He visto proyectos de modelos que se descarrilan no por mala técnica sino porque nadie acordó qué se estaba respondiendo. Este documento es el seguro contra eso. Cuando dentro de un año alguien cuestione por qué el modelo no anticipó algo que estaba fuera del alcance, el documento responde: porque conscientemente lo dejamos fuera, y aquí está escrito.

## Cierre del Tema 4

La definición formal del problema integra las cinco dimensiones, clasifica los supuestos por criticidad y se respalda en el EDA. Es el contrato que fija qué se responde y cómo se juzga, y protege el proyecto de los malentendidos que hunden más modelos que los errores técnicos. Con el problema definido con rigor, el Tema 5 ataca la pregunta que sigue: entre todas las estructuras posibles, cuál es el mejor modelo para este fenómeno.

## Referencias

Law, A. M. (2014). *Simulation modeling and analysis* (5a ed.). McGraw-Hill.

Banco Mundial. (2024). *World Development Indicators* [Conjunto de datos]. https://databank.worldbank.org/source/world-development-indicators

Fondo Monetario Internacional. (2024). *World Economic Outlook database*. https://www.imf.org/en/Publications/WEO
