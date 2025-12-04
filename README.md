# `mycn18-infra-logic-telemetry`

## Servicio Central de Telemetría y Observabilidad (Go + ClickHouse)

[](https://golang.org)
[](https://clickhouse.com/)
[](https://www.google.com/search?q=http://internal.mycn18.com)

-----

## Descripción del Microservicio (Uso Interno)

El repositorio `mycn18-infra-logic-telemetry` es el servicio de infraestructura dedicado a la recolección, almacenamiento y consulta de métricas y logs de todos los microservicios de MyCn18. Es esencial para la operación, monitoreo de salud y toma de decisiones basadas en datos.

**Funciones Clave:**

  * **Ingesta Concurrente:** Servidor Go optimizado para recibir un flujo constante de logs y métricas de alto volumen.
  * **Almacenamiento Analítico:** Utiliza **ClickHouse** (base de datos columnar) para consultas rápidas de series de tiempo y agregación de datos.
  * **Fuente del Admin:** Suministra los datos brutos y agregados al dashboard de `mycn18-admin`.

**La combinación de Go y ClickHouse** está diseñada específicamente para soportar el requisito de ingesta masiva y consulta de baja latencia requerido por una plataforma a escala.

-----

## Especificaciones Operacionales

| Parámetro | Valor Óptimo | Notas |
| :--- | :--- | :--- |
| **Lenguaje Base** | Go | Eficiencia en I/O y manejo de Goroutines para ingesta. |
| **Base de Datos** | ClickHouse | La mejor opción para consultas analíticas de series de tiempo. |
| **Interfaz (Write)** | HTTP (Endpoint de Ingesta) | Diseñado para recibir lotes de datos (`/ingest/logs`). |
| **Interfaz (Read)** | REST | API para que el dashboard de administración obtenga los datos. |
| **Consumidor Crítico**| `mycn18-admin` | Único lugar donde se visualizan los datos. |

-----

## Despliegue y Acceso

El servicio requiere una conexión estable a una instancia de ClickHouse.

### 1\. Variables de Entorno Clave

| Variable | Propósito |
| :--- | :--- |
| `CLICKHOUSE_HOST` | Dirección de la instancia de ClickHouse. |
| `API_INGEST_PORT` | Puerto de escucha para la ingesta de datos. |
| `API_QUERY_PORT` | Puerto de escucha para las consultas del dashboard. |

### 2\. Puntos de Acceso Críticos

  * **Ingesta:** Los microservicios envían sus logs a `POST /ingest/logs` o `POST /ingest/metrics`.
  * **Consulta:** El dashboard de administración accede a `GET /query/aggregate`.
