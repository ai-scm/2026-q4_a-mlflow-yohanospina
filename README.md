# Despliegue de Modelos de Machine Learning con MLflow y Docker

Este repositorio contiene la arquitectura para la gestión del ciclo de vida y despliegue en producción de modelos predictivos. La solución permite registrar, versionar y poner a disposición un modelo de aprendizaje automático como un servicio en tiempo real.

## Arquitectura General

El flujo de trabajo abarca tres etapas principales:

1. Registro y Gobierno: Almacenamiento centralizado del modelo y sus métricas de calidad.
2. Contenedorización: Empaquetado del servicio mediante contenedores para garantizar portabilidad.
3. Consumo en Tiempo Real: Exposición de un punto de enlace de inferencia para recibir datos y responder solicitudes automáticamente.

## Requisitos Previos

- Docker y Docker Compose
- Python 3.10 o superior

## Despliegue Rápido

Para iniciar la plataforma completa (servidor de gobierno y servicio de inferencia):

```bash
docker compose -f docker-compose.yml up --build
```

El panel de administración estará disponible en: `http://localhost:8000`

## Ejecución Manual

### 1. Iniciar el Servidor de Registro y Gobierno (MLflow)

```bash
mlflow server \
  --backend-store-uri sqlite:///mlflow.db \
  --default-artifact-root ./mlflow-artifact-root \
  --host 0.0.0.0 \
  --port 8000
```

### 2. Entrenar y Registrar el Modelo

```bash
python clf-train-registry.py clf-model "http://localhost:8000" --outputTestData test.csv
```

### 3. Publicar el Servicio de Inferencia

```bash
export MLFLOW_TRACKING_URI=http://localhost:8000
mlflow models serve -m models:/clf-model@Staging -p 1234 -h 0.0.0.0 --env-manager local
```

## Pruebas de Consumo (Inferencia)

Para enviar datos de prueba y recibir predicciones en tiempo real:

```bash
./predict.sh test.csv
```

O mediante solicitud HTTP directa:

```bash
curl http://localhost:1234/invocations -H 'Content-Type: text/csv' --data-binary @test.csv
```

## Detención del Servicio

Para detener todos los servicios y liberar recursos:

```bash
docker compose down
```
