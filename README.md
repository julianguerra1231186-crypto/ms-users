
![](https://github.com/julianguerra1231186-crypto/ms-users/blob/main/ms-users/miroservicio1.png)
# ms-users — Microservicio de Usuarios, Autenticación y Reclutamiento

### Se deja evidencia de todas las Historias de Usuario en la mesa de trabajo:
   - [Mesa De Trabajo](https://julianguerra1231186-1773894024267.atlassian.net/?continue=https%3A%2F%2Fjulianguerra1231186-1773894024267.atlassian.net%2Fwelcome%2Fsoftware%3FprojectId%3D10000&atlOrigin=eyJpIjoiOTdhMWY4ZGU5N2YwNDQ0MDk3NTZjODkxYTU5ZWVlZWQiLCJwIjoiamlyYS1zb2Z0d2FyZSJ9)

## Descripción general

`ms-users` es el microservicio central de seguridad de PulpApp. Es responsable de tres dominios:

1. **Autenticación y autorización** — registro, login y generación de tokens JWT
2. **Gestión de usuarios** — CRUD completo con control de acceso por rol
3. **Reclutamiento** — recepción y almacenamiento de postulaciones laborales con CV adjunto

Todos los demás microservicios confían en los tokens que este servicio genera. Sin pasar por aquí, ningún usuario puede acceder a las rutas protegidas del sistema.

---

## Puerto

| Entorno | Puerto |
|---------|--------|
| Local | 8081 |
| Docker (interno) | 8081 |
| Acceso desde host | http://localhost:8081 |
| Acceso vía Gateway | http://localhost:8090 |

---

## Stack tecnológico

| Tecnología | Versión | Propósito |
|-----------|---------|-----------|
| Java | 17 | Lenguaje base |
| Spring Boot | 4.0.3 | Framework principal |
| Spring Security | Incluido | Cadena de filtros y autorización |
| JJWT | 0.12.6 | Generación y validación de tokens JWT |
| Spring Data JPA | Incluido | Persistencia con Hibernate |
| PostgreSQL | 15 | Base de datos relacional |
| Liquibase | Incluido | Versionado del esquema de BD |
| Lombok | 1.18.38 | Reducción de código boilerplate |
| MapStruct | 1.5.5.Final | Mapeo entre entidades y DTOs |
| BCrypt | Incluido | Encriptación de contraseñas |

---

## Estructura de paquetes

```
com.pulpapp.ms_users/
│
├── config/
│   ├── SecurityConfig.java
│   │     Define la cadena de filtros de Spring Security.
│   │     Configura qué rutas son públicas y cuáles requieren ROLE_ADMIN o ROLE_SELLER.
│   │     Registra el JwtAuthFilter antes del filtro estándar de Spring.
│   │     Declara los beans: PasswordEncoder (BCrypt), AuthenticationProvider,
│   │     AuthenticationManager.
│   │
│   ├── CorsConfig.java
│   │     Habilita CORS para todos los orígenes y métodos HTTP.
│   │     Permite que el frontend consuma la API sin bloqueos del navegador.
│   │
│   └── LiquibaseConfig.java
│         Configuración adicional de Liquibase si se requiere personalización.
│
├── controller/
│   ├── AuthController.java
│   │     Expone los endpoints públicos de autenticación:
│   │     POST /auth/login    → recibe email + password, retorna JWT
│   │     POST /auth/register → crea usuario y retorna JWT inmediatamente
│   │
│   ├── UserController.java
│   │     CRUD completo de usuarios:
│   │     GET    /users                        → lista todos (ADMIN)
│   │     GET    /users/{id}                   → busca por ID
│   │     GET    /users/cedula/{cedula}         → busca por cédula (público)
│   │     GET    /users/validar/{ced}/{tel}     → valida cédula + teléfono (público)
│   │     POST   /users                        → crea usuario (público)
│   │     PUT    /users/{id}                   → actualiza usuario (público)
│   │     DELETE /users/{id}                   → elimina usuario (ADMIN)
│   │
│   ├── PedidoController.java
│   │     CRUD de pedidos internos:
│   │     POST   /pedidos      → crea pedido
│   │     GET    /pedidos      → lista todos
│   │     GET    /pedidos/{id} → busca por ID
│   │     DELETE /pedidos/{id} → elimina pedido
│   │
│   └── JobApplicationController.java
│         Gestión de postulaciones laborales:
│         POST /job-applications              → recibe formulario + CV (multipart, público)
│         GET  /job-applications              → lista todas las postulaciones (ADMIN)
│         GET  /job-applications/{id}/download → descarga el CV adjunto (ADMIN)
│
├── service/
│   ├── AuthService.java
│   │     Lógica de login: delega en AuthenticationManager → DaoAuthenticationProvider
│   │     → BCrypt para verificar contraseña → JwtService para generar token.
│   │     Lógica de registro: valida email y cédula únicos, encripta contraseña,
│   │     persiste el usuario y genera el token en una sola operación.
│   │
│   ├── UserServiceImpl.java
│   │     Extiende BaseServiceImpl para heredar findAll, findById, delete.
│   │     Sobreescribe save() y update() para encriptar la contraseña con BCrypt
│   │     antes de persistir. Nunca se guarda texto plano en la base de datos.
│   │
│   ├── PedidoServiceImpl.java
│   │     Gestiona pedidos internos. Busca el usuario por ID antes de crear
│   │     el pedido para garantizar la integridad referencial.
│   │
│   └── JobApplicationService.java
│         Recibe los datos del formulario y el archivo CV.
│         Genera un nombre único con UUID para el archivo y lo guarda en disco
│         en el directorio configurado (uploads/cv/).
│         Solo guarda el nombre del archivo en la base de datos, no el binario.
│
├── security/
│   ├── JwtService.java
│   │     Genera tokens firmados con HMAC-SHA256 usando la clave secreta configurada.
│   │     El token incluye: subject (email), role (claim personalizado),
│   │     issuedAt y expiration.
│   │     Valida tokens verificando firma y fecha de expiración.
│   │     Extrae claims individuales (email, rol) del payload del token.
│   │
│   ├── JwtAuthFilter.java
│   │     Extiende OncePerRequestFilter — se ejecuta exactamente una vez por request.
│   │     Extrae el token del header Authorization: Bearer <token>.
│   │     Si el token es válido, establece el contexto de seguridad de Spring.
│   │     Si el token es inválido o expirado, limpia el contexto y Spring
│   │     devuelve 401 automáticamente en rutas protegidas.
│   │
│   ├── JwtAuthEntryPoint.java
│   │     Intercepta los errores 401 y los devuelve en formato JSON.
│   │     Evita que Spring Security redirija al formulario de login HTML.
│   │
│   ├── UserDetailsServiceImpl.java
│   │     Implementa UserDetailsService de Spring Security.
│   │     Carga el usuario desde la base de datos usando el email como identificador.
│   │     Spring Security lo invoca automáticamente durante la autenticación.
│   │
│   └── UserPrincipal.java
│         Adapter que envuelve la entidad User y la expone como UserDetails.
│         Mantiene la entidad de dominio limpia de dependencias de Spring Security.
│         getUsername() retorna el email. getAuthorities() retorna el rol con
│         el prefijo ROLE_ que Spring Security espera.
│
├── entity/
│   ├── User.java
│   │     Entidad JPA mapeada a la tabla users.
│   │     Campos: id, cedula (unique), telefono, name, email (unique),
│   │     password (BCrypt), direccion, role (enum).
│   │
│   ├── Role.java
│   │     Enum con dos valores: ROLE_ADMIN y ROLE_SELLER.
│   │     Se almacena como VARCHAR en la base de datos.
│   │
│   ├── Pedido.java
│   │     Entidad JPA mapeada a la tabla pedidos.
│   │     Relación ManyToOne con User (N pedidos → 1 usuario).
│   │
│   └── JobApplication.java
│         Entidad JPA mapeada a la tabla job_applications.
│         El campo cvFile guarda solo el nombre UUID del archivo en disco,
│         no el contenido binario.
│
├── dto/
│   ├── LoginRequestDTO.java         → {email, password} con validaciones @NotBlank
│   ├── RegisterRequestDTO.java      → {cedula, telefono, name, email, password, direccion, role}
│   ├── AuthResponseDTO.java         → {token, email, name, role} — respuesta del login/register
│   ├── UserRequestDTO.java          → entrada para crear/actualizar usuario
│   ├── UserResponseDTO.java         → salida con datos del usuario (sin password)
│   ├── PedidoDTO.java               → transferencia de datos de pedidos
│   ├── JobApplicationRequestDTO.java → {fullName, email, phone, position, message}
│   └── JobApplicationResponseDTO.java → {id, fullName, email, phone, position, message, cvFile, createdAt}
│
├── repository/
│   ├── UserRepository.java
│   │     Extiende JpaRepository<User, Long>.
│   │     Métodos adicionales: findByCedula, findByEmail,
│   │     findByCedulaAndTelefono, existsByEmail, existsByCedula.
│   │
│   ├── PedidoRepository.java
│   │     Extiende JpaRepository<Pedido, Long>. CRUD básico.
│   │
│   └── JobApplicationRepository.java
│         Extiende JpaRepository<JobApplication, Long>.
│         Método adicional: findAllByOrderByCreatedAtDesc()
│         para mostrar las postulaciones más recientes primero.
│
├── mapper/
│   └── UserMapper.java
│         Convierte User → UserResponseDTO y UserRequestDTO → User.
│         Asigna ROLE_SELLER por defecto si no se envía rol en el request.
│
├── exception/
│   ├── GlobalExceptionHandler.java
│   │     @RestControllerAdvice que captura excepciones y las convierte en JSON.
│   │     Maneja: ResourceNotFoundException (404), BadCredentialsException (401),
│   │     AccessDeniedException (403), MethodArgumentNotValidException (400),
│   │     IllegalArgumentException (400), RuntimeException (404), Exception (500).
│   │
│   ├── BadCredentialsException.java  → lanzada cuando email o contraseña son incorrectos
│   └── ResourceNotFoundException.java → lanzada cuando no se encuentra un recurso por ID
│
└── core/
    ├── BaseServiceImpl.java
    │     Clase abstracta genérica que implementa findAll, findById, save, update, delete.
    │     Los servicios concretos la extienden y solo implementan los mappers específicos.
    │
    └── IBaseService.java
          Interfaz genérica que define el contrato CRUD base.
```

---

## Seguridad JWT — Flujo completo

### ¿Qué es un JWT?

Un JSON Web Token es una cadena de texto dividida en 3 partes separadas por puntos:

```
eyJhbGciOiJIUzI1NiJ9.eyJyb2xlIjoiUk9MRV9BRE1JTiIsInN1YiI6ImFkbWluQHB1bHBhcHAuY29tIn0.firma
     HEADER                              PAYLOAD                                    SIGNATURE
```

- **Header** → algoritmo de firma (HMAC-SHA256)
- **Payload** → datos del usuario (email, rol, fechas)
- **Signature** → garantiza que el token no fue modificado

### Flujo de autenticación

```
1. POST /auth/login {email, password}
         ↓
2. AuthService → AuthenticationManager.authenticate()
         ↓
3. DaoAuthenticationProvider carga el usuario por email
         ↓
4. BCrypt compara la contraseña enviada con el hash guardado en BD
         ↓
5. Si coincide → JwtService.generateToken() firma el token con HMAC-SHA256
         ↓
6. Respuesta: {token, email, name, role}

En cada request posterior:
7. JwtAuthFilter lee el header: Authorization: Bearer eyJ...
         ↓
8. JwtService.extractUsername() decodifica el email del payload
         ↓
9. UserDetailsServiceImpl carga el usuario desde la BD
         ↓
10. JwtService.isTokenValid() verifica firma + expiración
         ↓
11. SecurityContextHolder.setAuthentication() → Spring conoce quién es el usuario
         ↓
12. SecurityConfig evalúa si el rol tiene permiso para la ruta solicitada
```

### Payload del token

```json
{
  "role": "ROLE_ADMIN",
  "sub": "admin@pulpapp.com",
  "iat": 1712800000,
  "exp": 1712886400
}
```

### Tabla de permisos por ruta

| Ruta | Método | Público | ROLE_SELLER | ROLE_ADMIN |
|------|--------|---------|-------------|------------|
| /auth/** | cualquiera | ✅ | ✅ | ✅ |
| /users | POST | ✅ | ✅ | ✅ |
| /users/cedula/** | GET | ✅ | ✅ | ✅ |
| /users/validar/** | GET | ✅ | ✅ | ✅ |
| /users | GET | ❌ | ❌ | ✅ |
| /users/{id} | DELETE | ❌ | ❌ | ✅ |
| /products | GET | ✅ | ✅ | ✅ |
| /products | POST/PUT/DELETE | ❌ | ❌ | ✅ |
| /orders | POST | ✅ | ✅ | ✅ |
| /orders | GET | ❌ | ✅ | ✅ |
| /job-applications | POST | ✅ | ✅ | ✅ |
| /job-applications | GET | ❌ | ❌ | ✅ |
| /job-applications/{id}/download | GET | ❌ | ❌ | ✅ |

---

## Liquibase — Versionado de base de datos

### ¿Qué es Liquibase?

Liquibase es una herramienta que gestiona los cambios en la estructura de la base de datos de forma controlada y reproducible. En lugar de ejecutar scripts SQL manualmente, se definen **changesets** (conjuntos de cambios) en archivos YAML. Liquibase los ejecuta automáticamente al arrancar la aplicación y lleva un registro de cuáles ya fueron aplicados.

### ¿Por qué se usa?

- Garantiza que la base de datos siempre tenga la estructura correcta
- Permite trabajar en equipo sin conflictos de esquema
- Si la tabla ya existe, el changeset se marca como ejecutado sin error (`onFail: MARK_RAN`)
- Es reproducible: el mismo archivo genera la misma base de datos en cualquier entorno

### Changesets de ms-users

| ID | Descripción | Tabla afectada |
|----|-------------|----------------|
| 1-create-users-table | Crea la tabla users con cedula y email únicos | users |
| 2-create-pedidos-table | Crea la tabla pedidos con FK hacia users | pedidos |
| 3-add-fk-pedidos-to-users | Agrega FK pedidos.user_id → users.id con RESTRICT | pedidos |
| 4-add-role-to-users | Agrega columna role con default ROLE_SELLER | users |
| 5-create-job-applications-table | Crea la tabla de postulaciones laborales | job_applications |

---

## Modelo de datos

### Tabla `users`

| Columna | Tipo | Restricción | Descripción |
|---------|------|-------------|-------------|
| id | BIGINT | PK, autoincrement | Identificador único |
| cedula | VARCHAR(20) | NOT NULL, UNIQUE | Documento de identidad |
| telefono | VARCHAR(20) | nullable | Teléfono de contacto |
| name | VARCHAR(150) | NOT NULL | Nombre completo |
| email | VARCHAR(150) | NOT NULL, UNIQUE | Correo electrónico |
| password | VARCHAR(255) | NOT NULL | Hash BCrypt de la contraseña |
| direccion | VARCHAR(255) | NOT NULL | Dirección de entrega |
| role | VARCHAR(20) | NOT NULL, default ROLE_SELLER | Rol del usuario |

### Tabla `pedidos`

| Columna | Tipo | Restricción | Descripción |
|---------|------|-------------|-------------|
| id | BIGINT | PK, autoincrement | Identificador único |
| descripcion | VARCHAR(255) | NOT NULL | Descripción del pedido |
| total | DOUBLE PRECISION | NOT NULL | Valor total |
| user_id | BIGINT | FK → users (RESTRICT) | Usuario propietario |

### Tabla `job_applications`

| Columna | Tipo | Restricción | Descripción |
|---------|------|-------------|-------------|
| id | BIGINT | PK, autoincrement | Identificador único |
| full_name | VARCHAR(150) | NOT NULL | Nombre del candidato |
| email | VARCHAR(150) | NOT NULL | Correo del candidato |
| phone | VARCHAR(50) | NOT NULL | Teléfono del candidato |
| position | VARCHAR(150) | nullable | Cargo de interés |
| message | TEXT | nullable | Presentación personal |
| cv_file | VARCHAR(255) | nullable | Nombre UUID del archivo CV en disco |
| created_at | TIMESTAMP | NOT NULL, default NOW() | Fecha de postulación |

---

## Endpoints con ejemplos

### POST /auth/register

```json
// Request
{
  "cedula": "123456789",
  "telefono": "3001234567",
  "name": "Ana Admin",
  "email": "admin@pulpapp.com",
  "password": "admin123",
  "direccion": "Calle 1 # 2-3",
  "role": "ROLE_ADMIN"
}

// Response 201
{
  "token": "eyJhbGciOiJIUzI1NiJ9...",
  "email": "admin@pulpapp.com",
  "name": "Ana Admin",
  "role": "ROLE_ADMIN"
}
```

### POST /auth/login

```json
// Request
{
  "email": "admin@pulpapp.com",
  "password": "admin123"
}

// Response 200
{
  "token": "eyJhbGciOiJIUzI1NiJ9...",
  "email": "admin@pulpapp.com",
  "name": "Ana Admin",
  "role": "ROLE_ADMIN"
}
```

### POST /job-applications (multipart/form-data)

```
fullName = Juan Pérez
email    = juan@ejemplo.com
phone    = 3001234567
position = Operario de producción
message  = Me interesa trabajar con ustedes
file     = [archivo PDF]
```

---

## Manejo de errores

Todos los errores retornan JSON estructurado:

```json
{
  "status": 401,
  "error": "Unauthorized",
  "message": "Token JWT ausente, inválido o expirado"
}
```

| Excepción | HTTP | Cuándo ocurre |
|-----------|------|---------------|
| BadCredentialsException | 401 | Email o contraseña incorrectos |
| AccessDeniedException | 403 | Rol insuficiente para la operación |
| ResourceNotFoundException | 404 | Usuario o recurso no encontrado |
| MethodArgumentNotValidException | 400 | Campos inválidos en el request |
| IllegalArgumentException | 400 | Email o cédula ya registrados |
| Exception (fallback) | 500 | Error interno no controlado |

---

## Configuración

```yaml
server:
  port: ${SERVER_PORT:8081}

spring:
  datasource:
    url: ${SPRING_DATASOURCE_URL:jdbc:postgresql://localhost:5434/pulpapp_db}
    username: ${SPRING_DATASOURCE_USERNAME:postgres}
    password: ${SPRING_DATASOURCE_PASSWORD:1234}
  jpa:
    hibernate:
      ddl-auto: none        # Liquibase gestiona el esquema, Hibernate no toca las tablas
    show-sql: true
  liquibase:
    change-log: classpath:db/changelog/changelog-master.yml
  servlet:
    multipart:
      max-file-size: 5MB
      max-request-size: 10MB

jwt:
  secret: ${JWT_SECRET:cGVwcGVyTWludFB1bHBBcHBTZWNyZXRLZXkyMDI2IVhZWg==}
  expiration-ms: ${JWT_EXPIRATION_MS:86400000}   # 24 horas

app:
  uploads-dir: ${UPLOADS_DIR:uploads/cv}
```

---

## Levantar el servicio

```bash
# Reconstruir y levantar solo ms-users
docker-compose up --build ms-users

# Ver logs en tiempo real
docker-compose logs -f ms-users
```

Autor:

**Julian Guerra**
