# CONVALECER

**Análisis de la recuperación prolongada por COVID-19 en Cartagena de Indias y Bolívar mediante modelos supervisados y no supervisados**

Trabajo Colaborativo Contextualizado (TCC) de la asignatura Ciencia de Datos II, Ingeniería de Software, semestre VI, Universidad de Cartagena, Programas a Distancia. CIPAS CodeReview: Jairo Alonso Osorio Cruz. Tutor: Mauricio Martínez.

## Objetivo

Queremos saber qué características conocidas el día del diagnóstico se asocian con una recuperación prolongada por COVID-19, es decir, de más de 21 días entre el diagnóstico y la recuperación, y qué perfiles de municipios aparecen al agrupar el territorio según el comportamiento de sus casos. Con esos resultados orientamos al Centro Tutorial CRT de la Universidad de Cartagena sobre qué grupos de edad, periodos y municipios conviene priorizar en acciones de comunicación preventiva y de acompañamiento académico.

Los datos describen a la población de Cartagena y Bolívar, no a los estudiantes del CRT. Las conclusiones son territoriales y agregadas, y no sirven para tomar decisiones sobre personas.

## Fuente de datos

- **Dataset:** *Casos positivos de COVID-19 en Colombia*, Instituto Nacional de Salud (INS).
- **Portal:** [Datos Abiertos Colombia, recurso gt2j-8ykr](https://www.datos.gov.co/Salud-y-Protecci-n-Social/Casos-positivos-de-COVID-19-en-Colombia-/gt2j-8ykr).
- **Catálogo de variables:** [Sobre el dataset de casos de COVID-19 en Colombia](https://www.ins.gov.co/BibliotecaDigital/dataset-casos.pdf), INS.
- **Licencia de los datos:** [Creative Commons Atribución-CompartirIgual 4.0 Internacional](https://creativecommons.org/licenses/by-sa/4.0/). Los archivos de `data/` son el subconjunto original y obras derivadas de ese dataset, y se comparten con la misma licencia y con atribución al INS.
- **Filtro:** casos con `departamento_nom` igual a BOLIVAR o CARTAGENA: 203.862 registros y 23 variables, diagnosticados entre el 11 de marzo de 2020 y el 14 de enero de 2024.
- **Descarga:** 12 de septiembre de 2026, por la API pública del portal. El 13 de noviembre de 2026 la volvimos a descargar y coincidió celda por celda.

## Estructura del repositorio

```
data/
  covid19_bolivar_cartagena_raw.csv        Subconjunto original descargado de la API (203.862 x 23)
  convalecer_casos_preparado.csv           Dataset preparado por el notebook (203.862 x 34)
  convalecer_indicadores_municipio.csv     Indicadores por municipio para el agrupamiento (46 x 13)
  diccionario_datos.md                     Diccionario de datos, generado por el notebook
notebooks/
  convalecer_tcc.ipynb                     Notebook con todo el proceso y sus salidas
figuras/                                   Figuras numeradas generadas por el notebook
requirements.txt                           Versiones exactas de las librerías
```

## Cómo ejecutar el proyecto (macOS)

Necesitamos Python 3.13. Lo trabajamos en macOS con PyCharm, pero también funciona desde la terminal.

### Desde la terminal

```zsh
git clone https://github.com/JOsorioCruz/convalecer-covid19-cartagena.git
cd convalecer-covid19-cartagena
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
jupyter notebook notebooks/convalecer_tcc.ipynb
```

En Jupyter usamos *Kernel > Restart & Run All*. Para ejecutarlo sin abrir el navegador:

```zsh
cd notebooks
jupyter nbconvert --to notebook --execute --inplace convalecer_tcc.ipynb
```

### Desde PyCharm

1. Abrimos la carpeta del proyecto en PyCharm.
2. En *Settings > Project > Python Interpreter > Add Interpreter > Virtualenv* creamos o seleccionamos el `.venv` de la carpeta del proyecto.
3. En la terminal de PyCharm ejecutamos `pip install -r requirements.txt`.
4. Abrimos `notebooks/convalecer_tcc.ipynb` y usamos *Run All*.

### Si falta el archivo de datos

El notebook revisa si existe `data/covid19_bolivar_cartagena_raw.csv`. Si no lo encuentra, lo descarga de la API de datos.gov.co con el mismo filtro y lo guarda con el mismo formato. La descarga tarda varios minutos. Al cargar el archivo, el notebook imprime su huella SHA-256 y la compara con la del archivo original:

```
f0c7a8ae91ecfa61c4ea60eb3a67d80f84ff5edb2ae65aafb3f17323f71b9741
```

### Reproducibilidad

- El notebook tarda unos 15 segundos con el CSV en `data/` y genera las 14 figuras de `figuras/`.
- El notebook usa una semilla fija (42), así que al ejecutarlo de nuevo da los mismos resultados.
- Lo ejecutamos completo dos veces desde cero y obtuvimos salidas y archivos idénticos.
- Una advertencia técnica: la fuente escribe el texto "N/A" para los fallecidos por causas distintas a COVID-19, y pandas lo convierte en celda vacía con sus opciones por defecto. Por eso el notebook lee el CSV original con `keep_default_na=False, na_values=['']`.

## Resultados principales

**Datos.** De los 199.274 casos recuperados con fechas completas, el 9,72 % tardó más de 21 días en recuperarse. La recuperación prolongada sube con la edad, del 8,51 % en menores de 18 años al 12,09 % en las personas de 70 años o más, y cambia mucho según el periodo: llegó al 41,64 % en el segundo trimestre de 2020 y bajó a valores entre el 3,31 % y el 7,98 % entre finales de 2020 y comienzos de 2022.

**Calidad.** La fuente escribe "N/A" para los fallecidos por causas distintas a COVID-19, y pandas lo lee como vacío. Al corregirlo, 1.090 casos dejaron de contarse como fallecidos por COVID-19.

**Modelo supervisado.** Comparamos una línea base, una regresión logística y un árbol de decisión, con y sin el periodo del diagnóstico (prueba de 39.855 casos):

| Modelo | F1 | ROC-AUC | Recall | Lift |
|---|---|---|---|---|
| Línea base | 0,000 | 0,500 | 0,000 | 0,00 |
| Regresión logística con periodo | 0,234 | 0,635 | 0,580 | 1,51 |
| Regresión logística sin periodo | 0,228 | 0,616 | 0,511 | 1,51 |
| Árbol de decisión con periodo | 0,413 | 0,737 | 0,479 | 3,73 |
| Árbol de decisión sin periodo | 0,260 | 0,656 | 0,544 | 1,75 |

El 94,6 % de la importancia del mejor árbol está en el periodo del diagnóstico, que mezcla la evolución de la pandemia con cambios en la forma de cerrar los casos. Sin él, el desempeño es moderado.

**Agrupamiento.** K-means con 3 grupos sobre los 46 municipios (silueta de 0,39, partición estable con 10 semillas): un perfil mayoritario de 38 municipios con Cartagena (9,69 % de recuperación prolongada), un grupo de 4 municipios pequeños con casos de mayor edad (22,32 %, pero solo 517 casos) y un grupo de 4 que se distingue por cómo registró las fechas.

**Conclusión.** Los resultados sugieren recomendaciones de planeación agregada para el CRT, como plazos de flexibilidad de al menos tres semanas y énfasis preventivo en las personas mayores. No sirven para decidir sobre personas concretas.

## Autoría

CIPAS CodeReview: Jairo Alonso Osorio Cruz. Universidad de Cartagena, Programas a Distancia.
