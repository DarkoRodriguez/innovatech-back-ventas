# Backend - Microservicio de Ventas (Innovatech Chile)

Este repositorio contiene el microservicio de **Ventas** para Innovatech Chile, construido en **Java 17 con Spring Boot**. Se encarga de gestionar las órdenes de compra y exponer una API RESTful consumida por el Frontend.

## 🚀 Tecnologías Utilizadas
* **Framework:** Spring Boot 3
* **Lenguaje:** Java 17
* **ORM:** Hibernate / Spring Data JPA
* **Base de Datos:** MySQL
* **Documentación:** Swagger / OpenAPI
* **Contenedorización:** Docker

## 🏗️ Arquitectura y Contenedorización
El microservicio está paquetizado usando un **Dockerfile Multi-Stage**:
1. **Stage 1 (Build):** Usa una imagen oficial de Maven para descargar dependencias (`pom.xml`) y compilar el código fuente, generando el archivo `.jar`. Se aprovecha la caché de capas de Docker copiando las dependencias primero.
2. **Stage 2 (Producción):** Usa una imagen mínima de `eclipse-temurin:17-jre-alpine` para ejecutar el `.jar`.

**Buenas prácticas DevOps:**
* **Usuario No Root:** El contenedor crea un usuario específico (`innovatech`) y ejecuta la aplicación Java sin permisos de administrador.
* **Persistencia:** La base de datos MySQL (ejecutada en otro contenedor o RDS) utiliza volúmenes (`volumes`) para evitar la pérdida de datos ante reinicios.
* **Automatización de BD:** Se usa el enfoque *Code-First*. Spring Boot levanta automáticamente el esquema y las tablas al iniciar la aplicación (`ddl-auto=update`).

## ⚙️ Configuración y Ejecución Local

1. Asegúrate de tener la base de datos MySQL corriendo (puedes levantarla usando el `docker-compose.yml` general).
2. Construye y levanta el servicio:
   ```bash
   docker compose up -d --build backend-ventas
   ```
3. La API estará disponible internamente en el puerto `8080`.
4. Puedes acceder a la documentación de Swagger en: `http://localhost:8080/swagger-ui.html`

### Variables de Entorno Requeridas:
* `DB_ENDPOINT`: Host o nombre del contenedor MySQL.
* `DB_PORT`: Puerto de MySQL (3306).
* `DB_NAME`: Nombre de la base de datos.
* `DB_USERNAME` y `DB_PASSWORD`: Credenciales de acceso.

## 🔄 Pipeline CI/CD
El flujo de CI/CD automatiza la construcción de esta imagen Docker, publicándola en el registro y reiniciando el servicio en la instancia backend EC2 mediante comandos enviados por **AWS SSM** (sin exponer el puerto SSH). Este pipeline solo se dispara con modificaciones en la rama `deploy`.

## 📡 Comunicación entre aplicaciones
- **Cómo se conectan:** El microservicio de `ventas` expone una API REST que es consumida por el frontend y, cuando aplica, por el microservicio de `despachos`. En despliegues con Docker Compose, los servicios se comunican por la red interna usando los nombres de servicio como host.
- **Patrones de integración:** Se pueden usar llamadas HTTP directas (sincrónicas) o un bus de eventos/colas (mensajería) para desacoplar `ventas` de `despachos`. Elija el patrón según requisitos de consistencia y latencia.
- **Variables de entorno importantes:** `DB_ENDPOINT`, `DB_PORT`, `DB_NAME`, `DB_USERNAME`, `DB_PASSWORD`, `DESPACHOS_HOST` (si las llamadas a despachos son directas).

## 🔌 Endpoints de ejemplo
- `GET /api/v1/ventas` — Listar ventas.
- `POST /api/v1/ventas` — Crear una venta / orden de compra.
- `GET /api/v1/ventas/{id}` — Consultar una venta.

## 🧭 Ejecutar en el Monorepo (Docker Compose)
1. Desde la raíz del monorepo ejecutar:
   ```bash
   docker compose up -d --build
   ```
2. Esto levanta los servicios relacionados (front y back) y la base de datos. Use los nombres de servicio en las variables de entorno para enrutar llamadas internas.

## 🧾 Notas de integración
- Documente los contratos de API (OpenAPI/Swagger) para mantener consistencia entre consumidores y proveedores.
- Añada autenticación/autoridad (JWT, API Gateway) para proteger endpoints que modifican estados críticos.
