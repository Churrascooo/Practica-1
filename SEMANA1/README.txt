Este repositorio/notebook construye un dataset consolidado (estructurado + no estructurado) desde MIMIC-IV para una cohorte geriátrica (60+ años), integra diagnósticos y fármacos al ingreso, obtiene Top 5 comorbilidades, vincula notas clínicas de las primeras 24h y genera curvas de supervivencia Kaplan–Meier para dichas comorbilidades.

Cómo ejecutar (IMPORTANTE)

Se deben correr todos los bloques en orden, sin saltarse pasos.
Esto ya que se utiliza la base de datos y las notas de MIMIC-IV, las cuáles son bastante pesadas, por lo que se debe primero ejecutar los bloques de la importación de MIMIC-IV.

Qué se hace dentro del código?

Dentro del notebook, hay 7 secciones, las cuales son:

- Importación de MIMIC-IV
- Análisis de MIMIC-IV
- Filtrado de la población geriátrica *FASE 1
- Integración del historial clínico *FASE 2
- Identificación de comorbilidades predominantes *FASE 3
- Fusión de notas clínicas (Raw data) *FASE 4
- Modelado de supervivencia (Kaplan-Meier) *FASE 5

A continuación la explicación de cada fase.

Fase 1 — Cohorte geriátrica (60+)
- Carga `patients`, `admissions`, `icustays`
- Convierte columnas de tiempo a datetime
- Une `patients + admissions`, calcula edad al ingreso y filtra 60+
- Une con `icustays` para cohorte con UCI
- Guarda:
  - `outputs/cohort_hosp_60.parquet`
  - `outputs/cohort_icu_60.parquet`

Fase 2 — Diagnósticos y medicamentos al ingreso
- Carga diagnósticos (`diagnoses_icd`) y diccionario (`d_icd_diagnoses`)
- Integra diagnósticos a la cohorte 60+
- Carga prescripciones (`prescriptions`)
- Filtra fármacos con `starttime` dentro de las primeras 24h desde `admittime`
- Guarda:
  - `outputs/dx_60.parquet`
  - `outputs/rx_60_ingreso.parquet`

Fase 3 — Top 5 comorbilidades
- Limpia diagnósticos válidos
- Cuenta diagnósticos por frecuencia
- Extrae Top 5 comorbilidades
- Guarda:
  - `outputs/top5_comorbidities.parquet`

Fase 4 — Notas clínicas primeras 24h
- Define ventana [0–24h] desde `admittime`
- Carga notas de radiología (texto libre)
- Vincula por `hadm_id` y filtra por ventana temporal
- Concatena texto por hospitalización
- Control de calidad: longitud del texto (`n_chars`)
- Guarda:
  - `outputs/notes_first24h_hosp.parquet`

Fase 5 — Supervivencia Kaplan–Meier (Top 5)
- Construye dataset de supervivencia:
  - evento = `hospital_expire_flag`
  - tiempo = admisión → muerte (si evento) o alta (si censura)
- Crea flags binarios por presencia de cada comorbilidad Top 5
- Genera curvas Kaplan–Meier (supervivencia y riesgo acumulado) para:
  - con comorbilidad vs sin comorbilidad


Outputs principales

En `/content/outputs/`:
- `cohort_hosp_60.parquet` → cohorte geriátrica hospitalaria
- `cohort_icu_60.parquet` → cohorte geriátrica con estancia UCI
- `dx_60.parquet` → diagnósticos asociados a la cohorte
- `rx_60_ingreso.parquet` → fármacos iniciados en primeras 24h
- `top5_comorbidities.parquet` → Top 5 comorbilidades más frecuentes
- `notes_first24h_hosp.parquet` → notas clínicas (radiología) primeras 24h


Notas y consideraciones

- Este análisis es descriptivo (Kaplan–Meier univariado por comorbilidad).
- Hacia el final del seguimiento pueden quedar pocos pacientes en riesgo, por lo que las curvas pueden ser menos estables en la cola.