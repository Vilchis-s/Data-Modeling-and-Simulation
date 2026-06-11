# Apéndice: reproducibilidad

Para que cualquiera pueda correr el código del libro y obtener los mismos resultados, dejo aquí el entorno, las versiones, las fuentes de datos y las notas prácticas que aprendí al ejecutar todos los ejemplos. La reproducibilidad no es un trámite: es parte de la honestidad científica, y la guía del curso la pone como una ventaja central de la pseudoaleatoriedad, que la misma semilla genera la misma secuencia (Law, 2014).

## Entorno

El código se escribió y probó con Python 3.11. Recomiendo un entorno virtual aislado para no contaminar la instalación global.

```bash
python -m venv venv
# Windows
venv\Scripts\activate
# instalar dependencias
pip install -r requirements.txt
```

## requirements.txt

```
numpy>=1.26
scipy>=1.11
pandas>=2.1
statsmodels>=0.14
matplotlib>=3.8
wbgapi>=1.0.12
pmdarima>=2.0
SALib>=1.4
```

Una nota sobre `pmdarima`: depende de versiones específicas de `numpy` y `statsmodels`, y a veces da conflictos con las versiones más nuevas. Si la instalación falla, fijo `numpy` a una versión compatible o sustituyo `auto_arima` por una búsqueda manual de órdenes con `statsmodels`, que no tiene esa dependencia.

## Fuentes de datos y acceso

Todos los datos reales del libro vienen de la API del Banco Mundial, accedida con la librería `wbgapi`. Es abierta y no requiere clave de API. Los indicadores que uso son:

```
NY.GDP.MKTP.CD       PIB a precios actuales, dólares corrientes
NY.GDP.MKTP.KD.ZG    Crecimiento anual del PIB, porcentaje
NY.GDP.MKTP.PP.CD    PIB en paridad de poder adquisitivo, dólares internacionales
```

Los códigos de país siguen el estándar ISO de tres letras. La composición de bloques que uso en el libro es:

```
BRICS+ : BRA, RUS, IND, CHN, ZAF, EGY, ETH, IRN, ARE
G7     : USA, JPN, DEU, GBR, FRA, ITA, CAN
```

## Reproducibilidad de la aleatoriedad

Donde hay simulación, fijo la semilla con el generador moderno de NumPy, no con el antiguo. La guía es explícita en preferir `default_rng` sobre `np.random.seed`, porque el primero usa un generador de mejor calidad, Xoshiro256, mientras que el segundo usa el Mersenne Twister legacy (Law, 2014).

```python
import numpy as np
rng = np.random.default_rng(7)   # semilla fija para reproducibilidad
```

Toda simulación del libro que use aleatoriedad pasa su `rng` de forma explícita, en lugar de depender de un estado global, lo que evita resultados que cambian según el orden de ejecución de las celdas.

## Notas prácticas que aprendí ejecutando los ejemplos

Los datos del Banco Mundial tienen huecos, sobre todo en años tempranos y para países con cifras afectadas por sanciones. Al sumar bloques, esos huecos se propagan como valores faltantes; conviene revisar con `isna` antes de agregar y decidir el tratamiento de forma consciente, no dejar que `sum` los ignore en silencio.

Las series de PIB anual son cortas, pocas décadas. Los modelos ARIMA de orden alto sobreajustan con facilidad sobre tan pocos datos, así que mantengo los órdenes bajos y desconfío de cualquier mejora marginal de AIC que venga de agregar parámetros.

El ajuste de la logística con `curve_fit` es sensible a los valores iniciales `p0`. Si la convergencia falla o el techo K sale absurdo, conviene dar mejores valores iniciales basados en el EDA, por ejemplo un K del orden de dos veces el último valor observado.

Al exponenciar pronósticos hechos en escala logarítmica, los intervalos dejan de ser simétricos, lo cual es correcto y deseable para una variable positiva. No hay que forzarlos a simétricos.

## Estructura del repositorio

El libro está organizado como un proyecto de GitBook con sincronización Git. La estructura de carpetas refleja los siete temas, y `SUMMARY.md` define el orden del menú lateral. Cada archivo `.md` es una página del libro publicado.

```
README.md                     portada
SUMMARY.md                    índice del menú lateral
.gitbook.yaml                 configuración de GitBook
sistemas-continuos/           marco conceptual
tema-1-metodos-numericos/     ocho páginas
tema-2-series-de-tiempo/      nueve páginas
tema-3-aplicacion-negocios/   cuatro páginas
tema-4-identificacion-problema/ cuatro páginas
tema-5-seleccion-modelo/      cinco páginas
tema-6-interpretacion/        cuatro páginas
tema-7-soluciones/            cuatro páginas
referencias.md                referencias APA 7
apendice-reproducibilidad.md  este apéndice
```

## Referencias

Law, A. M. (2014). *Simulation modeling and analysis* (5a ed.). McGraw-Hill.

Banco Mundial. (2024). *World Development Indicators* [Conjunto de datos]. https://databank.worldbank.org/source/world-development-indicators
