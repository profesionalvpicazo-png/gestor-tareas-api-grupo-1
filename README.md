# API de Gestión de Tareas

API REST para gestionar el ciclo de vida de tareas construida con **FastAPI** y **SQLAlchemy**. Permite crear, consultar, actualizar y eliminar tareas. Cada tarea cuenta con un identificador único, título, descripción opcional, estado (`pending`, `in_progress`, `done`) y fecha de creación asignada automáticamente.

---

## Requisitos previos

| Requisito | Versión mínima |
|-----------|---------------|
| Python    | 3.12+         |
| pip       | 23+           |

### Dependencias principales

| Paquete    | Versión  | Uso                                   |
|------------|----------|---------------------------------------|
| FastAPI    | 0.136.1  | Framework web asíncrono               |
| SQLAlchemy | 2.0.49   | ORM para acceso a base de datos       |
| Pydantic   | 2.13.4   | Validación de datos y serialización   |
| Uvicorn    | 0.46.0   | Servidor ASGI                         |
| pytest     | 9.0.3    | Framework de tests                    |
| httpx      | 0.28.1   | Cliente HTTP para tests de integración|
| anyio      | 4.13.0   | Compatibilidad asíncrona para tests   |

---

## Instalación paso a paso

1. **Clonar el repositorio:**

   ```bash
   git clone https://github.com/profesionalvpicazo-png/gestor-tareas-api-grupo-1.git
   cd gestor-tareas-api-grupo-1
   ```

2. **Crear y activar el entorno virtual:**

   ```bash
   python -m venv venv
   ```

   - **Windows:**

     ```bash
     venv\Scripts\activate
     ```

   - **macOS / Linux:**

     ```bash
     source venv/bin/activate
     ```

3. **Instalar las dependencias:**

   ```bash
   pip install -r requirements.txt
   ```

---

## Cómo arrancar la aplicación

```bash
uvicorn aplicacion.principal:app --reload
```

La API quedará disponible en `http://127.0.0.1:8000`.

La documentación interactiva (Swagger UI) estará en `http://127.0.0.1:8000/docs`.

---

## Endpoints

La API expone 5 endpoints bajo el prefijo `/tasks`.

### 1. Listar todas las tareas

| Campo  | Valor                    |
|--------|--------------------------|
| Método | `GET`                    |
| Ruta   | `/tasks/`                |
| Params | Ninguno                  |

**Ejemplo curl:**

```bash
curl -X GET http://127.0.0.1:8000/tasks/
```

**Ejemplo de respuesta** (`200 OK`):

```json
[
  {
    "id": 1,
    "title": "Comprar materiales",
    "description": "Ir a la ferretería",
    "status": "pending",
    "created_at": "2026-05-27T10:00:00"
  }
]
```

---

### 2. Obtener una tarea por ID

| Campo  | Valor                          |
|--------|--------------------------------|
| Método | `GET`                          |
| Ruta   | `/tasks/{task_id}`             |
| Params | `task_id` (int) — ruta, obligatorio |

**Ejemplo curl:**

```bash
curl -X GET http://127.0.0.1:8000/tasks/1
```

**Ejemplo de respuesta** (`200 OK`):

```json
{
  "id": 1,
  "title": "Comprar materiales",
  "description": "Ir a la ferretería",
  "status": "pending",
  "created_at": "2026-05-27T10:00:00"
}
```

**Respuesta de error** (`404 Not Found`):

```json
{
  "detail": "Task not found"
}
```

---

### 3. Crear una nueva tarea

| Campo  | Valor                                  |
|--------|----------------------------------------|
| Método | `POST`                                 |
| Ruta   | `/tasks/`                              |
| Body   | JSON con los campos descritos abajo    |

**Parámetros del cuerpo (JSON):**

| Campo         | Tipo   | Obligatorio | Valor por defecto | Descripción              |
|---------------|--------|-------------|-------------------|--------------------------|
| `title`       | string | Sí          | —                 | Título de la tarea       |
| `description` | string | No          | `null`            | Descripción opcional     |
| `status`      | string | No          | `"pending"`       | `pending`, `in_progress` o `done` |

**Ejemplo curl:**

```bash
curl -X POST http://127.0.0.1:8000/tasks/ \
  -H "Content-Type: application/json" \
  -d '{"title": "Revisar código", "description": "PR #42"}'
```

**Ejemplo de respuesta** (`201 Created`):

```json
{
  "id": 2,
  "title": "Revisar código",
  "description": "PR #42",
  "status": "pending",
  "created_at": "2026-05-27T12:30:00"
}
```

---

### 4. Actualizar parcialmente una tarea

| Campo  | Valor                                  |
|--------|----------------------------------------|
| Método | `PATCH`                                |
| Ruta   | `/tasks/{task_id}`                     |
| Params | `task_id` (int) — ruta, obligatorio    |
| Body   | JSON con los campos a modificar        |

**Parámetros del cuerpo (JSON):**

| Campo         | Tipo   | Obligatorio | Descripción                            |
|---------------|--------|-------------|----------------------------------------|
| `title`       | string | No          | Nuevo título                           |
| `description` | string | No          | Nueva descripción                      |
| `status`      | string | No          | Nuevo estado: `pending`, `in_progress` o `done` |

> **Nota:** No se pueden actualizar tareas con estado `done`. El servidor devolverá `400 Bad Request`.

**Ejemplo curl:**

```bash
curl -X PATCH http://127.0.0.1:8000/tasks/1 \
  -H "Content-Type: application/json" \
  -d '{"status": "in_progress"}'
```

**Ejemplo de respuesta** (`200 OK`):

```json
{
  "id": 1,
  "title": "Comprar materiales",
  "description": "Ir a la ferretería",
  "status": "in_progress",
  "created_at": "2026-05-27T10:00:00"
}
```

**Respuesta de error** (`400 Bad Request` — tarea completada):

```json
{
  "detail": "Cannot update a completed task"
}
```

**Respuesta de error** (`404 Not Found`):

```json
{
  "detail": "Task not found"
}
```

---

### 5. Eliminar una tarea

| Campo  | Valor                          |
|--------|--------------------------------|
| Método | `DELETE`                       |
| Ruta   | `/tasks/{task_id}`             |
| Params | `task_id` (int) — ruta, obligatorio |

**Ejemplo curl:**

```bash
curl -X DELETE http://127.0.0.1:8000/tasks/1
```

**Respuesta exitosa:** `204 No Content` (sin cuerpo).

**Respuesta de error** (`404 Not Found`):

```json
{
  "detail": "Task not found"
}
```

---

## Cómo ejecutar los tests

```bash
pytest tests/ -v
```

Los tests utilizan una base de datos SQLite independiente para garantizar el aislamiento entre casos. No afectan al archivo `tareas.db` de producción.

---

## Estructura del proyecto

```
gestor-tareas-api-grupo-1/
├── aplicacion/                 # Paquete principal de la aplicación
│   ├── __init__.py             # Marca el directorio como paquete Python
│   ├── principal.py            # Punto de entrada: crea la instancia FastAPI y registra routers
│   ├── base_de_datos.py        # Configuración del engine, sesión de SQLAlchemy y dependencia get_db
│   ├── modelos.py              # Modelos ORM (tabla tasks, enumeración TaskStatus)
│   ├── esquemas.py             # Esquemas Pydantic de entrada (TaskCreate, TaskUpdate) y respuesta (TaskResponse)
│   └── rutas/                  # Directorio de definiciones de endpoints
│       ├── __init__.py         # Marca el directorio como paquete Python
│       └── tareas.py           # Endpoints REST de tareas (GET, POST, PATCH, DELETE)
├── tests/                      # Suite de tests
│   ├── __init__.py             # Marca el directorio como paquete Python
│   └── test_tasks.py           # Tests funcionales con pytest y TestClient de FastAPI
├── requirements.txt            # Dependencias del proyecto (producción y desarrollo)
├── AGENTS.md                   # Instrucciones y convenciones para agentes de IA
├── .gitignore                  # Archivos y carpetas excluidos del control de versiones
└── README.md                   # Documentación del proyecto
```
