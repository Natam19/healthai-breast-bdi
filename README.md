# HealthAI Breast BDI

Sistema de Big Data para la predicción de riesgo de cáncer de mama y la analítica epidemiológica en Colombia.

Proyecto académico de la asignatura Big Data (2026), Programa de Ingeniería de Sistemas, Universidad de San Buenaventura, Bogotá.

**Autores:** Agustín Peralta Guarín y Natalia Moncaleano Montero.

**Aplicación:** https://bigdata.agustinynatalia.site

**Código fuente:** https://github.com/Agustin1231/cancer-risk-predictor

---

## Qué hace

Una aplicación web mobile-first que le entrega al usuario:

- **Mi riesgo:** evaluación paso a paso con 10 factores de riesgo y un nivel de riesgo (bajo, medio o alto).
- **Resultado:** factores que más pesan, recomendaciones preventivas y plan de tamizaje sugerido.
- **Autoexamen:** guía con temporizador.
- **Plan semanal:** lista de hábitos.
- **Panorama:** dashboard epidemiológico del cáncer de mama en Colombia, con descarga de datos.

La salida del modelo es una estimación estadística poblacional con fines educativos. No es un diagnóstico.

## Datos

| Fuente | Tipo | Uso |
|---|---|---|
| BCSC Risk Factor Dataset | Clínico (EE. UU.) | Entrenamiento del modelo: 1,5 M de filas agregadas que representan 6,7 M de mamografías (~400 MB en CSV) |
| GLOBOCAN 2022 (IARC/OMS) | Global | Incidencia, mortalidad y prevalencia por país |
| Cuenta de Alto Costo | Oficial Colombia | Casos por régimen, departamento y grupo de edad |
| SIVIGILA (INS) | Vigilancia pública | Casos notificados |

## Arquitectura

Patrón batch + streaming sobre un data lake. El procesamiento pesado está en AWS y la aplicación web en un VPS, lo que baja el costo sin perder capacidad de procesamiento.

```mermaid
flowchart LR
    A[Formulario React] -->|POST /predict| B[API FastAPI]
    B -->|cada predicción| C[AWS Kinesis Firehose]
    C --> D[(Amazon S3 - data lake)]
    D --> E[EC2 + Apache Spark / Jupyter]
    E -->|modelo .pkl| D
    D -->|carga del modelo| B
```

## Pipeline

1. Ingesta y limpieza del BCSC con Pandas (tratamiento de valores desconocidos y codificación de variables).
2. Exportación a Parquet.
3. Análisis exploratorio: variable objetivo, correlaciones ponderadas, tasas por edad, densidad mamaria y antecedentes.
4. Entrenamiento: regresión logística y Random Forest con `class_weight` para el desbalance; evaluación con AUC-ROC, precision y recall (partición 80/20 ponderada). Se prioriza el recall.
5. Procesamiento distribuido con PySpark leyendo directo desde S3.
6. Servicio del modelo con FastAPI; validación de entradas con Pydantic.
7. Captura streaming de cada predicción con Kinesis Firehose para reentrenamiento.

## Tecnologías

| Capa | Tecnologías |
|---|---|
| Big Data y ML | Apache Spark (PySpark), Jupyter, Pandas, NumPy, Scikit-learn, Parquet |
| Nube (AWS) | S3, Kinesis Firehose, EC2, IAM |
| Backend | FastAPI, Pydantic, Uvicorn, Boto3, Docker |
| Frontend | React 18, Vite, React Router, Recharts, Framer Motion, Nginx |
| Despliegue | Coolify, Docker Compose, Traefik, Let's Encrypt, Cloudflare |

## Limitaciones

- El BCSC es de EE. UU.: hay que calibrar el modelo con datos colombianos antes de usarlo con población local.
- No incluye factores genéticos (BRCA1/BRCA2).
- El reentrenamiento todavía es manual.
