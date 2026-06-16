# BudgetWise Backend

API REST del proyecto **BudgetWise**, una aplicación Android de finanzas personales para registrar ingresos, gastos, consultar reportes y mantener el control financiero desde una interfaz sencilla.

Este repositorio contiene la parte backend del sistema: autenticación básica, gestión de usuarios y operaciones sobre transacciones financieras. El backend funciona como complemento de la app móvil, permitiendo conectar la experiencia local de BudgetWise con una API propia preparada para sincronización y evolución futura.

> Este backend forma parte del ecosistema BudgetWise junto con la app Android:  
> **App repository:** [Budget-Wise-App](https://github.com/AlejandroDehesa/Budget-Wise-App)

---

## Contexto del proyecto

BudgetWise está planteado como un producto móvil realista, no como una pantalla aislada.

La app Android permite al usuario registrar movimientos financieros, visualizar su saldo, consultar ingresos y gastos, revisar reportes y trabajar con una estructura preparada para evolucionar hacia funcionalidades más avanzadas.

El backend aporta la capa remota del proyecto:

- gestión básica de usuarios;
- registro e inicio de sesión;
- recuperación de contraseña en modo demo;
- persistencia de transacciones por usuario;
- operaciones CRUD sobre movimientos financieros;
- base preparada para sincronización entre app móvil y servidor.

La idea principal del proyecto es demostrar integración real entre:

```txt
Android App → Room / Repository → Retrofit → FastAPI → SQLAlchemy → Database
```

---

## Objetivo técnico

El objetivo de este backend es demostrar una base funcional y defendible para una app de finanzas personales:

- diseño de API REST;
- validación de datos con Pydantic;
- persistencia con SQLAlchemy;
- separación entre modelos, schemas y configuración de base de datos;
- endpoints consumidos desde una app Android real;
- soporte para SQLite en desarrollo y PostgreSQL en despliegue;
- despliegue preparado mediante Uvicorn y Procfile.

---

## Stack técnico

| Área | Tecnología |
|---|---|
| Lenguaje | Python |
| Framework API | FastAPI |
| ORM | SQLAlchemy 2.0 |
| Validación | Pydantic |
| Base de datos local | SQLite |
| Base de datos producción | PostgreSQL |
| Servidor ASGI | Uvicorn |
| Integración mobile | Retrofit / OkHttp desde Android |
| Despliegue | Railway / Procfile |

---

## Funcionalidades principales

### Autenticación

- Crear usuario.
- Iniciar sesión.
- Resetear contraseña en modo demo.
- Devolver un usuario remoto para vincularlo con la app Android.

### Transacciones

- Listar transacciones de un usuario.
- Crear una nueva transacción.
- Actualizar una transacción existente mediante flujo `upsert`.
- Eliminar una transacción concreta.
- Asociar cada transacción a un usuario remoto.

### Base de datos

- Modelo `User`.
- Modelo `Transaction`.
- Relación usuario → transacciones.
- Configuración compatible con SQLite y PostgreSQL.
- Creación automática de tablas en el arranque para una versión MVP.

---

## Relación con la app Android

La app BudgetWise usa este backend como API remota.

En la aplicación móvil, el usuario trabaja con una experiencia local basada en Room y repositorios. El backend permite añadir una segunda capa: usuarios remotos y transacciones almacenadas en servidor.

Flujo simplificado:

```txt
1. El usuario crea cuenta o inicia sesión en la app.
2. La app recibe un remoteUserId desde el backend.
3. La app guarda localUserId y remoteUserId en sesión.
4. El usuario registra ingresos o gastos.
5. La app puede persistir localmente y enviar movimientos al backend.
6. El backend guarda las transacciones asociadas al usuario remoto.
```

Esto convierte BudgetWise en un proyecto mobile + backend, no solo en una app visual.

---

## Arquitectura del backend

```txt
app/
├── main.py        # Aplicación FastAPI y endpoints principales
├── db.py          # Configuración de base de datos y sesión SQLAlchemy
├── models.py      # Modelos ORM: User y Transaction
└── schemas.py     # Schemas Pydantic de entrada y salida
```

### Responsabilidad de cada archivo

| Archivo | Responsabilidad |
|---|---|
| `main.py` | Define la app FastAPI, CORS, dependencias de base de datos y endpoints |
| `db.py` | Configura `DATABASE_URL`, engine, sesiones y base declarativa |
| `models.py` | Define las tablas y relaciones de la base de datos |
| `schemas.py` | Define los contratos de entrada/salida de la API |
| `requirements.txt` | Lista las dependencias necesarias |
| `Procfile` | Comando de arranque para despliegue |

---

## Endpoints principales

### Auth

| Método | Endpoint | Descripción |
|---|---|---|
| `POST` | `/auth/signup` | Crea un nuevo usuario |
| `POST` | `/auth/login` | Inicia sesión con usuario y contraseña |
| `POST` | `/auth/reset-password` | Actualiza la contraseña en modo demo |

### Transactions

| Método | Endpoint | Descripción |
|---|---|---|
| `GET` | `/users/{user_id}/transactions` | Lista las transacciones de un usuario |
| `POST` | `/users/{user_id}/transactions` | Crea o actualiza una transacción |
| `DELETE` | `/users/{user_id}/transactions/{tx_id}` | Elimina una transacción concreta |

---

## Modelo de datos

### User

```txt
id          string   UUID del usuario
username    string   Email / identificador único
password    string   Contraseña en modo demo
```

### Transaction

```txt
id          string   ID de la transacción
user_id     string   Usuario propietario de la transacción
type        string   Tipo de movimiento: ingreso o gasto
amount      float    Importe
category    string   Categoría
note        string   Nota opcional
date        string   Fecha del movimiento
created_at  integer  Timestamp de creación
updated_at  integer  Timestamp de última actualización
```

---

## Ejemplo de uso

### Crear usuario

```http
POST /auth/signup
Content-Type: application/json
```

```json
{
  "username": "demo@budgetwise.com",
  "password": "1234"
}
```

Respuesta:

```json
{
  "id": "user-uuid",
  "username": "demo@budgetwise.com"
}
```

---

### Iniciar sesión

```http
POST /auth/login
Content-Type: application/json
```

```json
{
  "username": "demo@budgetwise.com",
  "password": "1234"
}
```

Respuesta:

```json
{
  "id": "user-uuid",
  "username": "demo@budgetwise.com"
}
```

---

### Crear o actualizar una transacción

```http
POST /users/{user_id}/transactions
Content-Type: application/json
```

```json
{
  "id": "tx-001",
  "type": "GASTO",
  "amount": 24.99,
  "category": "Comida",
  "note": "Cena",
  "date": "16/06/2026"
}
```

Respuesta:

```json
{
  "id": "tx-001",
  "user_id": "user-uuid",
  "type": "GASTO",
  "amount": 24.99,
  "category": "Comida",
  "note": "Cena",
  "date": "16/06/2026",
  "created_at": 1781620000000,
  "updated_at": 1781620000000
}
```

---

## Instalación local

### 1. Clonar el repositorio

```bash
git clone https://github.com/AlejandroDehesa/BudgetWise-Backend.git
cd BudgetWise-Backend
```

### 2. Crear entorno virtual

```bash
python -m venv .venv
```

Activar en Windows:

```bash
.venv\Scripts\activate
```

Activar en macOS/Linux:

```bash
source .venv/bin/activate
```

### 3. Instalar dependencias

```bash
pip install -r requirements.txt
```

### 4. Ejecutar servidor

```bash
uvicorn app.main:app --reload
```

API local:

```txt
http://127.0.0.1:8000
```

Documentación interactiva:

```txt
http://127.0.0.1:8000/docs
```

---

## Configuración de base de datos

Por defecto, el backend usa SQLite:

```txt
sqlite:///./budgetwise.db
```

Para usar PostgreSQL, define la variable de entorno `DATABASE_URL`:

```bash
DATABASE_URL=postgresql://user:password@host:port/database
```

El proyecto adapta automáticamente la URL para usar el driver `psycopg` cuando detecta PostgreSQL.

---

## Despliegue

El repositorio incluye un `Procfile` preparado para ejecutar la app con Uvicorn:

```txt
web: uvicorn app.main:app --host 0.0.0.0 --port $PORT
```

Esto permite desplegar el backend en plataformas como Railway u otros entornos compatibles con procesos web basados en puerto dinámico.

---

## Estado actual

Este proyecto se encuentra en una versión **MVP funcional**.

La API ya permite conectar la app Android con un backend propio para usuarios y transacciones, pero todavía no debe presentarse como una solución final de producción.

Actualmente está pensado para demostrar:

- integración real entre app móvil y backend;
- diseño básico de API REST;
- persistencia de datos;
- modelado usuario/transacciones;
- base preparada para futuras mejoras.

---

## Limitaciones actuales

Esta versión mantiene algunas decisiones simples para priorizar claridad y avance del MVP:

- autenticación en modo demo;
- contraseñas todavía sin hashing;
- CORS abierto para facilitar pruebas;
- creación automática de tablas en arranque;
- ausencia de migraciones formales;
- sin JWT todavía;
- sin test suite automatizada completa.

Estas limitaciones están identificadas y forman parte del roadmap técnico del proyecto.

---

## Roadmap técnico

Próximas mejoras previstas:

- Añadir autenticación con JWT.
- Hashear contraseñas con un algoritmo seguro.
- Separar rutas en routers dedicados.
- Crear capa de servicios para lógica de negocio.
- Añadir migraciones con Alembic.
- Mejorar validaciones de transacciones.
- Añadir tests unitarios e integración.
- Configurar CORS por entorno.
- Añadir Docker.
- Documentar despliegue completo.
- Mejorar sincronización app/backend.
- Preparar base para reglas financieras inteligentes.
- Explorar futura integración con Open Banking.

---

## Qué demuestra este proyecto

BudgetWise Backend demuestra capacidad para construir una API real conectada a una app móvil:

- diseño de endpoints REST;
- modelado de entidades;
- uso de ORM;
- validación de datos;
- integración con Android mediante Retrofit;
- persistencia local/remota;
- separación básica de responsabilidades;
- despliegue de una API propia;
- criterio para construir una primera versión sin sobreingeniería.

Dentro del portfolio, BudgetWise refuerza el perfil mobile + backend: una app Android con producto, datos, reportes, persistencia local y una API propia preparada para evolucionar.

---

## Repositorios relacionados

- **BudgetWise App:** [Budget-Wise-App](https://github.com/AlejandroDehesa/Budget-Wise-App)
- **BudgetWise Backend:** [BudgetWise-Backend](https://github.com/AlejandroDehesa/BudgetWise-Backend)

---

## Autor

**Alejandro Dehesa Delgado**

Desarrollador Software Junior con foco en backend, Java, Android, datos, APIs REST e inteligencia artificial aplicada.

Portfolio orientado a proyectos con producto, arquitectura realista y evolución técnica defendible.
