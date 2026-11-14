# Diccionario de datos: CONVALECER

Generado por `notebooks/convalecer_tcc.ipynb` (sección 6). No editar a mano.

## Fuente

- Dataset: *Casos positivos de COVID-19 en Colombia*, Instituto Nacional de Salud (INS).
- Portal: Datos Abiertos Colombia, recurso `gt2j-8ykr`: https://www.datos.gov.co/Salud-y-Protecci-n-Social/Casos-positivos-de-COVID-19-en-Colombia-/gt2j-8ykr
- Catálogo oficial de variables: https://www.ins.gov.co/BibliotecaDigital/dataset-casos.pdf
- Licencia: Creative Commons Atribución-CompartirIgual 4.0 Internacional (CC BY-SA 4.0).
- Filtro: `departamento_nom` igual a BOLIVAR o CARTAGENA.
- Archivo de partida: `data/covid19_bolivar_cartagena_raw.csv`, 203862 filas y 23 columnas, SHA-256 `f0c7a8ae91ecfa61c4ea60eb3a67d80f84ff5edb2ae65aafb3f17323f71b9741`.

## Archivo `data/convalecer_casos_preparado.csv`

Unidad de análisis: caso confirmado de COVID-19 anonimizado. 203862 filas y 34 columnas.

| Variable | Tipo | Descripción | Origen | Transformación |
|---|---|---|---|---|
| `departamento_cod_divipola` | Entero | Código DIVIPOLA: 13 Bolívar, 13001 Cartagena | departamento | Renombrada |
| `departamento_nom` | Texto | Bolivar o Cartagena | departamento_nom | Mayúscula inicial |
| `municipio_cod_divipola` | Entero | Código DIVIPOLA del municipio | ciudad_municipio | Renombrada |
| `municipio_nom` | Texto | Nombre del municipio | ciudad_municipio_nom | Renombrada, mayúscula inicial |
| `es_cartagena` | Entero 0/1 | 1 si el caso es del distrito de Cartagena | departamento_nom | Derivada |
| `edad` | Entero | Edad en la unidad indicada por unidad_medida | edad | Sin cambios |
| `unidad_medida` | Entero | 1 años, 2 meses, 3 días | unidad_medida | Sin cambios |
| `edad_anios` | Decimal | Edad en años | edad, unidad_medida | Derivada: meses / 12, días / 365 |
| `edad_atipica_iqr` | Booleano | Edad por encima de Q3 + 1,5 IQR | edad_anios | Derivada (bandera) |
| `grupo_edad` | Texto ordinal | 0 a 17, 18 a 29, 30 a 44, 45 a 59, 60 a 69, 70 o más | edad_anios | Derivada |
| `sexo` | Texto | F femenino, M masculino | sexo | Sin cambios |
| `sexo_cod` | Entero 0/1 | F = 0, M = 1 | sexo | Codificada |
| `pertenencia_etnica_cod` | Entero | 1 Indígena, 2 ROM, 3 Raizal, 4 Palenquero, 5 Negro, 6 Otro | per_etn_ | Renombrada; faltantes imputados con la moda |
| `etnia_imputada` | Booleano | Verdadero si la pertenencia étnica fue imputada | per_etn_ | Derivada (bandera) |
| `fuente_tipo_contagio` | Texto | Comunitaria, Relacionado o Importado | fuente_tipo_contagio | Sin cambios |
| `contagio_importado` | Entero 0/1 | 1 si el contagio es importado | fuente_tipo_contagio | One-hot (referencia Comunitaria) |
| `contagio_relacionado` | Entero 0/1 | 1 si el contagio es relacionado | fuente_tipo_contagio | One-hot (referencia Comunitaria) |
| `ubicacion` | Texto | Casa, Hospital, Hospital UCI, Fallecido o Fallecido no COVID | ubicacion | Unificada; 'N/A' de la fuente pasa a 'Fallecido no COVID' |
| `estado` | Texto | Leve, Moderado, Grave, Fallecido o Fallecido no COVID | estado | Unificada; 'N/A' de la fuente pasa a 'Fallecido no COVID' |
| `recuperado` | Texto | Recuperado, Fallecido, Fallecido no COVID o Activo | recuperado | Unificada; 'N/A' de la fuente pasa a 'Fallecido no COVID' |
| `tipo_recuperacion` | Texto | PCR (segunda prueba negativa) o Tiempo (criterio clínico) | tipo_recuperacion | Unificada |
| `fecha_inicio_sintomas` | Fecha | Inicio de síntomas; vacía en asintomáticos | fecha_inicio_sintomas | Convertida a fecha |
| `fecha_diagnostico` | Fecha | Confirmación por laboratorio | fecha_diagnostico | Convertida a fecha |
| `fecha_recuperado` | Fecha | Fecha de recuperación | fecha_recuperado | Convertida a fecha |
| `fecha_muerte` | Fecha | Fecha de muerte | fecha_muerte | Convertida a fecha |
| `asintomatico` | Booleano | Sin fecha de inicio de síntomas | fecha_inicio_sintomas | Derivada |
| `dias_sintomas_a_diagnostico` | Entero | Días entre inicio de síntomas y diagnóstico | fechas | Derivada |
| `dias_diagnostico_a_recuperacion` | Entero | Días entre diagnóstico y recuperación | fechas | Derivada |
| `anio_diagnostico` | Entero | Año del diagnóstico | fecha_diagnostico | Derivada |
| `meses_desde_inicio` | Entero | Meses desde marzo de 2020 hasta el mes del diagnóstico | fecha_diagnostico | Derivada |
| `alerta_muerte_antes_diagnostico` | Booleano | Fecha de muerte anterior al diagnóstico | fechas | Derivada (bandera) |
| `alerta_notificacion_posterior_reporte` | Booleano | Notificación posterior al reporte web | fechas | Derivada (bandera) |
| `en_poblacion_supervisada` | Booleano | Recuperado con días de recuperación calculables | recuperado, fechas | Derivada |
| `recuperacion_prolongada` | Booleano | Más de 21 días de diagnóstico a recuperación; vacío fuera de la población supervisada | dias_diagnostico_a_recuperacion | Derivada (variable objetivo) |

## Archivo `data/convalecer_indicadores_municipio.csv`

Unidad de análisis: municipio. 46 filas y 13 columnas.

| Variable | Tipo | Descripción | Origen | Transformación |
|---|---|---|---|---|
| `municipio_cod_divipola` | Entero | Código DIVIPOLA del municipio | Casos | Clave de agrupación |
| `municipio_nom` | Texto | Nombre del municipio | Casos | Clave de agrupación |
| `departamento_nom` | Texto | Bolivar o Cartagena | Casos | Clave de agrupación |
| `casos` | Entero | Número de casos confirmados | Casos | Conteo |
| `edad_mediana` | Decimal | Mediana de edad en años | edad_anios | Mediana |
| `pct_60_o_mas` | Decimal | Porcentaje de casos de 60 años o más | edad_anios | Porcentaje |
| `pct_hombres` | Decimal | Porcentaje de casos de sexo masculino | sexo_cod | Porcentaje |
| `pct_contagio_relacionado` | Decimal | Porcentaje de contagios relacionados | contagio_relacionado | Porcentaje |
| `mediana_dias_sintomas_diagnostico` | Decimal | Mediana de días de síntomas a diagnóstico | dias_sintomas_a_diagnostico | Mediana |
| `pct_fallecidos_covid` | Decimal | Porcentaje de casos con recuperado = 'Fallecido' | recuperado | Porcentaje |
| `recuperados_con_dias` | Entero | Casos de la población supervisada | en_poblacion_supervisada | Conteo |
| `pct_recuperacion_prolongada` | Decimal | Porcentaje de recuperación prolongada en la población supervisada | recuperacion_prolongada | Porcentaje |
| `menos_de_100_casos` | Booleano | Municipio con menos de 100 casos; sus porcentajes son poco estables | casos | Bandera |
