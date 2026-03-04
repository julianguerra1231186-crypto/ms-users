# Microservicio ms-users

## Descripción
El microservicio **ms-users** es responsable de la gestión de usuarios dentro del sistema **PulpApp - Sistema Distribuido de Venta Online de Pulpas Naturales**.
Este servicio permite realizar operaciones CRUD sobre los usuarios, incluyendo registro, consulta, actualización y eliminación.
La información se almacena en una base de datos **PostgreSQL** y el servicio se ejecuta dentro de contenedores **Docker**.
El microservicio expone una API REST que puede ser consumida por el **API Gateway** o directamente por el frontend.
---
# Arquitectura
Este microservicio hace parte de una arquitectura basada en microservicios compuesta por:
* Frontend (Interfaz web)
* API Gateway
* Microservicio de usuarios (ms-users)
* Base de datos PostgreSQL
* Contenedores Docker
El microservicio se encarga exclusivamente de la gestión de usuarios.
---
# Tecnologías utilizadas
* Java 17
* Spring Boot
* Spring Data JPA
* PostgreSQL
* Docker
* Maven
* Lombok
* Jakarta Validation
* Postman (pruebas de API)
---
# Estructura del proyecto
```
ms-users
│
├── controller
│   └── UserController.java
│
├── service
│   ├── IUserService.java
│   └── UserServiceImpl.java
│
├── repository
│   └── UserRepository.java
│
├── entity
│   └── User.java
│
├── dto
│   ├── UserRequestDTO.java
│   └── UserResponseDTO.java
│
└── MsUsersApplication.java
```
---
# Modelo de datos

Tabla: **users**

| Campo     | Tipo   | Descripción               |
| --------- | ------ | ------------------------- |
| id        | Long   | Identificador del usuario |
| cedula    | String | Documento de identidad    |
| name      | String | Nombre completo           |
| email     | String | Correo electrónico        |
| password  | String | Contraseña                |
| direccion | String | Dirección de entrega      |
---
# Endpoints disponibles
## Obtener todos los usuarios
GET
```
/users
```
Respuesta:
```
[
 {
  "id": 1,
  "name": "Juan Perez",
  "email": "juan@email.com"
 }
]
```
---
## Obtener usuario por ID
GET
```
/users/{id}
```
--
## Crear usuario
POST
```
/users
```
Body ejemplo:
```
{
 "cedula": "1075310263",
 "name": "Sebastian Guzman",
 "email": "sebastian@gmail.com",
 "password": "123456",
 "direccion": "Calle 79 #8-42"
}
```
---

## Actualizar usuario
PUT
```
/users/{id}
```
---
## Eliminar usuario
DELETE
```
/users/{id}
```
---

# Ejecución del proyecto
## 1 Clonar repositorio
```
git clone https://github.com/usuario/PulpApp.git
```
---
## 2 Compilar proyecto
```
mvn clean package
```
---
## 3 Ejecutar con Docker
```
docker compose up --build
```
---
# Pruebas de la API
Las pruebas se realizaron utilizando **Postman**.
```
http://localhost:8081/users
```
---

# Base de datos
Motor utilizado:
PostgreSQL
Puerto:
```
5433
```
Base de datos:
```
pulpapp_users
```
---

# Autor

Proyecto desarrollado como parte de la asignatura **Sistemas Distribuidos**.

Autor:

**Julian Guerra**
