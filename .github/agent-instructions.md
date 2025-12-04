# REPOSITORIO: `mycn18-infra-logic-telemetry`

## I. Descripción General del Proyecto (Contexto de MyCn18)

MyCn18 es una Plataforma de Desarrollo de Software No-Code/Low-Code universal y políglota, diseñada para soportar arquitecturas de microservicios de nivel empresarial. Su objetivo es permitir a los usuarios crear, versionar, probar y desplegar lógica de negocio en múltiples targets (Web, IoT, Data Science) a través de un editor visual. La arquitectura está diseñada para una escalabilidad masiva (millones de usuarios) y un ecosistema de Marketplace robusto.

-----

## II. Rol y Responsabilidad del Repositorio

  - Este es el **Servicio Central de Observabilidad**. Es la fuente única de verdad para todas las métricas, logs de acceso y traces de la plataforma MyCn18.
  - **Funciones Clave:** **Ingesta de Alto Rendimiento** (Manejo de millones de eventos por minuto), **Almacenamiento Columnar** (optimizado para consultas analíticas) y **API de Consulta** para dashboards y reportes.
  - **Tipo de repo:** `Logic` (Backend de Infraestructura).
  - **Stack Sugerido:** **Go** para la API de Ingesta (máxima concurrencia). **ClickHouse** para el almacenamiento de series de tiempo. ClickHouse supera a PostgreSQL/MongoDB en el rendimiento de consultas analíticas que el dashboard de administración requiere (ej., *latencia promedio de los últimos 7 días por región*).
  - **Meta Final:** Contenedor Docker de alta disponibilidad y escritura concurrente.

-----

## III. Licenciamiento y Dependencias

  * **Licencia del Código:** MIT
  * **Servicios que Consume (Dependencias Críticas):**
      * **ClickHouse Instance:** Almacenamiento de todos los datos brutos (logs y métricas).
  * **Servicios que Alimenta (Salidas):**
      * `mycn18-ui-admin`: Es el consumidor principal de la API de consulta para alimentar todos los gráficos y reportes del dashboard.
  * **Consumidores del Servicio (Fuentes de Datos):**
      * `mycn18-logic-api-gateway`: Logs de acceso y latencia de todas las peticiones externas.
      * `mycn18-core-logic-engine`: Métricas de ejecución (tiempo por nodo, errores, uso de recursos).
      * **Todos los demás microservicios:** Logs de aplicación para depuración y monitoreo.

-----

## IV. Plan de Desarrollo y Visión (Visión 1.0)

El objetivo de la Visión 1.0 es establecer una tubería de datos estable y rápida que pueda servir datos al dashboard de administración.

1.  **API de Ingesta:** Desarrollar un endpoint de **Go** optimizado para recibir grandes lotes de datos JSON/Protobuf y escribirlos en ClickHouse con baja latencia.
2.  **Esquemas de Datos:** Definir los esquemas de tabla en ClickHouse para logs (estructurados) y métricas (series de tiempo).
3.  **API de Consulta (Read):** Implementar una API de lectura que reciba filtros (fechas, servicio, tipo de error) y ejecute consultas analíticas rápidas en ClickHouse.

-----

## V. Plan de Iteraciones Iniciales (MVP)

| Task | Subtasks |
| :--- | :--- |
| **Configuración de Backend de Ingesta.** | \* Configurar el servidor Go (Echo) con un *handler* de alto rendimiento. \* Implementar la conexión de la capa de persistencia con ClickHouse. \* Definir el esquema de tabla inicial para los logs del `api-gateway` (hora, ruta, latencia, estado HTTP). |
| **Implementación de la API de Escritura Asíncrona.** | \* Crear el endpoint `POST /ingest/metrics` que acepta datos en lote. \* Utilizar **Goroutines** para manejar la ingesta concurrente sin bloquear la respuesta de la API. \* Implementar la lógica de *buffering* (buffer de logs antes de la escritura masiva). |
| **Desarrollo de la API de Consulta Básica.** | \* Crear el endpoint `GET /query/latency` para que `mycn18-admin` solicite la latencia promedio por servicio. \* Implementar la consulta optimizada de ClickHouse para datos de las últimas 24 horas. |
