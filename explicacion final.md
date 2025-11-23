# 📘 **Documento Unificado – Proyecto Final (Arquitectura + Stack + Flujo + HU)**

## 1. ✔ **Stack Tecnológico General**

### **Frontend**

* React + Vite
* Fetch API para consumo del API Gateway
* Deploy en Azure App Service (contenedores)

### **Backend / API Gateway**

* Node.js
* Express
* JWT (autenticación)
* node-fetch / global fetch (para enrutar peticiones)

### **Microservicios Backend**

* Node.js + Express
* Arquitectura REST
* Conexión a BD según servicio

### **Bases de Datos**

* **PostgreSQL (Azure Database for PostgreSQL)**

  * doctors
  * patients
  * appointments

* **MongoDB (Azure Cosmos DB for MongoDB API)**

  * pharmacy
  * sistema de autoincremento basado en colección `counters`

### **Infraestructura en Azure**

* Azure Container Registry (ACR)
* Azure App Service (Web App for Containers)
* Azure Database for PostgreSQL
* Azure Cosmos DB for MongoDB
* GitHub Actions (CI/CD, build & push de imágenes)

---

## 2. ✔ **Arquitectura General del Sistema**

### 🔐 **Flujo de Autenticación**

1. El **usuario** entra al módulo de **Login** (React).
2. Login envía correo/contraseña al **API Gateway**.
3. El Gateway valida credenciales contra su base de usuarios:

   * Usuarios de prueba
   * Usuarios cargados desde doctors y patients
4. Si son válidas → genera **JWT (access + refresh)**
5. Devuelve el token al frontend login
6. Login redirige al **Portal**

---

### 🏛️ **Flujo dentro del Portal**

1. El Portal recibe el **JWT**
2. Muestra **solo los módulos permitidos según el rol del usuario**:

   * admin → todo
   * doctor → doctors, appointments, pharmacy
   * patient → appointments, patients, pharmacy

---

### 🔁 **Flujo hacia cualquier microservicio**

Cuando se accede a un módulo:

1. El módulo (React) hace peticiones → **API Gateway**
2. El Gateway:

   * Valida el JWT
   * Identifica a qué servicio debe enviar
   * Reenvía la petición al microservicio correcto
3. El microservicio realiza:

   * GET, POST, PUT, DELETE
   * Validaciones necesarias
4. El Gateway responde al frontend con los datos de vuelta

---

### 🗄️ **Bases de Datos por Microservicio**

| Servicio         | BD               | Notas                                   |
| ---------------- | ---------------- | --------------------------------------- |
| doctors-api      | PostgreSQL       | Perfil del doctor, correo, especialidad |
| patients-api     | PostgreSQL       | Datos del paciente                      |
| appointments-api | PostgreSQL       | FKs a doctors y patients                |
| pharmacy-api     | MongoDB (Cosmos) | inventario, recetas, counters           |

---

### 🐳 **Contenedores y Despliegue**

* Cada microservicio se empaqueta como **imagen Docker completa**:

  * Backend Express
  * Frontend React/Vite (build incluido)
* GitHub Actions construye y empuja imágenes al ACR
* Azure Web App for Containers levanta cada imagen, una por microservicio
* Variables de entorno configuradas por App Service

---

## 3. ✔ **Diagrama de Arquitectura (Descripción para tu imagen generada)**

Ya generamos una imagen basada en este flujo, pero aquí está el texto definitivo:

```
Usuario → Login (React) → API Gateway → Valida credenciales → Genera JWT → Login redirige → Portal (React) → Selección de módulo

Portal → (JWT) → API Gateway

API Gateway → valida token → enruta → { Doctors | Patients | Appointments | Pharmacy }

Doctors → PostgreSQL (Azure)
Patients → PostgreSQL (Azure)
Appointments → PostgreSQL (Azure)
Pharmacy → MongoDB (Cosmos DB)

Todo se ejecuta en contenedores Docker → publicados vía GitHub Actions → Azure ACR → Azure App Service
```

---

## 4. ✔ **Historias de Usuario (10 para sustentación)**

| #  | Historia de Usuario                                                 | Criterios de Aceptación                        |
| -- | ------------------------------------------------------------------- | ---------------------------------------------- |
| 1  | Como paciente quiero registrarme para acceder a mis datos médicos.  | Registro guardado en patients-api.             |
| 2  | Como administrador quiero ver el inventario de medicamentos.        | pharmacy-api devuelve inventario actualizado.  |
| 3  | Como paciente quiero agendar citas médicas.                         | appointments-api crea la cita exitosamente.    |
| 4  | Como médico quiero ver mis citas del día.                           | appointments-api lista las citas asignadas.    |
| 5  | Como médico quiero emitir una receta digital.                       | pharmacy-api crea la receta y descuenta stock. |
| 6  | Como farmacéutico quiero actualizar el stock de medicamentos.       | pharmacy-api actualiza stock.                  |
| 7  | Como paciente quiero consultar mis recetas médicas.                 | pharmacy-api devuelve lista de recetas.        |
| 8  | Como administrador quiero gestionar la información de médicos.      | doctors-api permite CRUD completo.             |
| 9  | Como administrador quiero gestionar pacientes.                      | patients-api permite CRUD completo.            |
| 10 | Como usuario quiero acceder solo a módulos permitidos según mi rol. | Portal filtra módulos según el JWT.            |

---

## 5. ✔ **Funcionamiento Técnico Completo**

### 🔐 Login + Seguridad

* Expide **access token (corto)** y **refresh token (mediano)**
* refresh token permite obtener nuevos access tokens
* Los datos del usuario (rol, id, email) se incluyen dentro del access token

---

### 🧠 API Gateway

* Centraliza todo
* Verifica JWT
* Inyecta credenciales
* Rutea a microservicios
* Unifica errores y respuestas

---

### 🩺 Microservicios

#### Doctors / Patients / Appointments

* PostgreSQL
* AUTOINCREMENT
* Relaciones con FKs
* CRUD completo

#### Pharmacy

* MongoDB
* Sistema de autoincremento por colección
* Inventario
* Recetas digitales

---

### 🖥 Frontend — Portal y Módulos

* Cada módulo es una SPA independiente en React
* Todo está empaquetado en su contenedor
* El portal controla permisos con el JWT
* Los módulos siempre consultan *solo* por el API Gateway

---

### ⚙ CI/CD — GitHub Actions

Cada push a `main`:

1. Construye imagen de cada app
2. Setea variables VITE_*
3. Publica imágenes en ACR
4. Azure App Service hace pull automático
5. La plataforma arranca los contenedores

---

## 6. ✔ **Conclusión consolidada del proyecto**

Este proyecto implementa una arquitectura profesional basada en **microservicios**, desplegada en **Azure**, con un **API Gateway**, autenticación por **JWT**, sistemas distribuidos y **bases de datos híbridas** (PostgreSQL + MongoDB).
Toda la solución está contenedorizada con Docker, automatizada con GitHub Actions y con un diseño real utilizado en aplicaciones empresariales.

Esta arquitectura permite:

* Escalabilidad por servicio
* Mantenibilidad y desarrollo independiente
* Seguridad centralizada
* Observabilidad y despliegue continuo
* Experiencia de usuario moderna (SPA)

---
