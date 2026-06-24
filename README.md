# Back Ventas — API REST

API REST para la gestión de **ventas / órdenes de compra**. Forma parte del sistema de despachos junto con el [frontend](../front_despacho) y la [API de Despachos](../back-Despachos_SpringBoot).

Desarrollada con **Spring Boot 3.4.4** y **Java 17**.

> El proyecto Maven se encuentra dentro de la carpeta [`Springboot-API-REST/`](Springboot-API-REST). En la raíz del repositorio están la configuración de despliegue (`k8s/`, `.github/`).

---

## 🛠️ Stack tecnológico

| Categoría        | Tecnología |
|------------------|------------|
| Lenguaje         | Java 17 |
| Framework        | Spring Boot 3.4.4 |
| Web              | Spring Web (REST) |
| Persistencia     | Spring Data JPA / Hibernate |
| Base de datos    | MySQL (producción) · H2 (perfil `test`) |
| Validación       | Spring Boot Starter Validation |
| Documentación    | springdoc OpenAPI (Swagger UI) |
| Utilidades       | Lombok |
| Build            | Maven (`mvnw` incluido) |

---

## 📦 Modelo de datos — `Venta`

| Campo               | Tipo        | Notas |
|---------------------|-------------|-------|
| `idVenta`           | `Long`      | PK, autogenerado |
| `direccionCompra`   | `String`    | Obligatorio (`@NotBlank`) |
| `valorCompra`       | `int`       | Valor de la compra |
| `fechaCompra`       | `LocalDate` | Obligatorio · formato `yyyy-MM-dd` |
| `despachoGenerado`  | `Boolean`   | Obligatorio · indica si ya tiene despacho (por defecto `false`) |

---

## 🔌 Endpoints

Base: `/api/v1/ventas` · CORS habilitado para cualquier origen (`*`).

| Método   | Ruta                     | Descripción                       | Respuesta |
|----------|--------------------------|-----------------------------------|-----------|
| `GET`    | `/api/v1/ventas`         | Lista todas las ventas            | `200 OK` |
| `GET`    | `/api/v1/ventas/{id}`    | Obtiene una venta por ID          | `200 OK` / `404` |
| `POST`   | `/api/v1/ventas`         | Crea una nueva venta              | `201 Created` |
| `PUT`    | `/api/v1/ventas/{id}`    | Actualiza una venta existente     | `200 OK` / `404` |
| `DELETE` | `/api/v1/ventas/{id}`    | Elimina una venta                 | `204 No Content` / `404` |

### Ejemplo de cuerpo (`POST` / `PUT`)

```json
{
  "direccionCompra": "Av. Siempre Viva 742",
  "valorCompra": 22990,
  "fechaCompra": "2024-02-02",
  "despachoGenerado": false
}
```

### 📖 Documentación interactiva (Swagger)

Con la aplicación corriendo:

```
http://localhost:8080/swagger-ui.html
```

---

## ⚙️ Configuración

La conexión a la base de datos se define con **variables de entorno** (ver `src/main/resources/application.properties`):

| Variable      | Descripción                  |
|---------------|------------------------------|
| `DB_ENDPOINT` | Host del servidor MySQL      |
| `DB_PORT`     | Puerto (ej. `3306`)          |
| `DB_NAME`     | Nombre de la base de datos   |
| `DB_USERNAME` | Usuario                      |
| `DB_PASSWORD` | Contraseña                   |

> Hibernate usa `ddl-auto=update`. El puerto del servicio por defecto es **8080**.

### Perfil de pruebas (`test`)

El perfil `test` usa una base de datos **H2 en memoria** (no requiere MySQL). Se activa con `SPRING_PROFILES_ACTIVE=test`.

---

## 🚀 Ejecución local

Requisitos: **JDK 17** y MySQL (o usar el perfil `test` con H2).

```bash
cd Springboot-API-REST

# Definir variables de entorno de la BD (ejemplo)
export DB_ENDPOINT=localhost DB_PORT=3306 DB_NAME=ventas \
       DB_USERNAME=root DB_PASSWORD=secret

# Ejecutar con el wrapper de Maven
./mvnw spring-boot:run          # Linux / macOS
mvnw.cmd spring-boot:run        # Windows
```

Para ejecutar con H2 (perfil de pruebas, sin MySQL):

```bash
./mvnw spring-boot:run -Dspring-boot.run.profiles=test
```

### Compilar y empaquetar

```bash
./mvnw clean package            # genera target/*.jar
java -jar target/*.jar
```

### Tests

```bash
./mvnw test
```

---

## 🐳 Docker

Build multi-etapa (Maven para compilar, JRE Alpine para ejecutar). El contenedor expone el puerto **8080**.

```bash
cd Springboot-API-REST
docker build -t back-ventas .
docker run -p 8080:8080 \
  -e DB_ENDPOINT=... -e DB_PORT=3306 -e DB_NAME=ventas \
  -e DB_USERNAME=... -e DB_PASSWORD=... \
  back-ventas
```

---

## ☸️ Despliegue (Kubernetes + CI/CD)

- **`k8s/deployment.yaml`**: Deployment (`back-ventas`, 2 réplicas, imagen `criss247/back-ventas:latest`, perfil `test`) + Service tipo `LoadBalancer` en el puerto 8080.

  ```bash
  kubectl apply -f k8s/deployment.yaml
  ```

- **`.github/workflows/deploy.yml`**: pipeline de CI/CD que, en cada *push* a `main`:
  1. Construye y publica la imagen Docker en Docker Hub.
  2. Configura credenciales AWS y actualiza el clúster **EKS** (`devops-eks`).
  3. Aplica los manifiestos y fuerza el *rollout* del deployment.

---

## 📂 Estructura

```
back-Ventas_SpringBoot/
├── Springboot-API-REST/
│   ├── src/main/java/com/citt/
│   │   ├── SpringbootApiRestApplication.java
│   │   ├── config/OpenApiConfing.java
│   │   ├── controller/VentaController.java
│   │   ├── exceptions/            # Manejo de errores (404, etc.)
│   │   └── persistence/
│   │       ├── entity/Venta.java
│   │       ├── repository/VentaRepository.java
│   │       └── services/          # VentaService + impl
│   ├── src/main/resources/
│   │   ├── application.properties
│   │   └── application-test.properties
│   ├── src/test/                  # Pruebas unitarias
│   ├── Dockerfile
│   └── pom.xml
├── k8s/deployment.yaml
└── .github/workflows/deploy.yml
```
