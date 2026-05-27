# ADR-001: Elección de SQLite como base de datos

## Estado

**Aceptado**

## Contexto

La API de gestión de tareas necesita una base de datos para persistir las tareas
que los usuarios crean, consultan, actualizan y eliminan. El proyecto tiene las
siguientes características que condicionan la elección:

- Es una API de alcance reducido con un único recurso principal (tareas).
- El volumen de datos esperado es bajo a medio.
- El equipo de desarrollo es pequeño y necesita incorporar nuevos miembros con
  rapidez.
- Se prioriza la simplicidad de despliegue y la ausencia de infraestructura
  externa.
- Los tests deben ejecutarse de forma aislada y rápida, sin depender de
  servicios externos.

## Decisión

Se adopta **SQLite** como motor de base de datos, almacenando los datos en un
archivo local (`tareas.db`). El acceso se realiza a través de SQLAlchemy 2.0,
lo que permite cambiar de motor en el futuro sin reescribir la lógica de acceso
a datos.

### Razones principales

1. **Cero dependencias de infraestructura:** SQLite no requiere instalar ni
   configurar un servidor de base de datos. El archivo se crea automáticamente
   al arrancar la aplicación.
2. **Configuración inmediata:** Cualquier desarrollador puede clonar el
   repositorio y ejecutar la API sin pasos de configuración adicionales.
3. **Tests rápidos y aislados:** SQLite soporta bases de datos en memoria
   (`StaticPool`), lo que permite ejecutar los tests sin tocar el archivo de
   producción y sin necesidad de levantar contenedores.
4. **Portabilidad:** El archivo de la base de datos se puede copiar, respaldar
   o inspeccionar con herramientas estándar sin conocimientos especializados.
5. **Suficiente para el caso de uso:** El volumen de operaciones y la
   complejidad de las consultas no justifican un motor más pesado en esta fase.

## Alternativas consideradas

### PostgreSQL

| Aspecto | Detalle |
|---|---|
| **Ventajas** | Motor robusto y maduro para entornos de producción a gran escala. Soporte nativo de tipos avanzados (JSON, arrays, rangos). Excelente rendimiento en consultas complejas y escrituras concurrentes. Amplio ecosistema de extensiones. |
| **Inconvenientes** | Requiere instalar y mantener un servidor externo o un contenedor Docker. Añade complejidad a la configuración del entorno de desarrollo y al pipeline de tests. Sobredimensionado para el volumen y la complejidad actuales del proyecto. |

### MySQL

| Aspecto | Detalle |
|---|---|
| **Ventajas** | Amplia adopción y comunidad extensa. Buen rendimiento en lecturas intensivas. Disponibilidad de servicios gestionados en la mayoría de proveedores cloud. |
| **Inconvenientes** | Al igual que PostgreSQL, requiere un servidor dedicado. Menor riqueza de tipos y funcionalidades avanzadas frente a PostgreSQL. Historial de incompatibilidades entre versiones y modos de configuración. No aporta ventajas claras sobre PostgreSQL para este tipo de aplicación. |

## Consecuencias

### Positivas

- El tiempo de incorporación de nuevos desarrolladores se reduce al mínimo:
  basta con instalar las dependencias de Python.
- El ciclo de desarrollo local es rápido al no depender de servicios externos.
- Los tests se ejecutan en milisegundos gracias a SQLite en memoria.

### Negativas y riesgos a largo plazo

- **Concurrencia limitada:** SQLite utiliza bloqueo a nivel de archivo. Si la
  aplicación crece y recibe múltiples escrituras simultáneas, podrían aparecer
  cuellos de botella.
- **Sin soporte multiservidor:** SQLite no permite conexiones remotas. Si se
  necesita escalar horizontalmente la API (varias instancias), será necesario
  migrar a un motor cliente-servidor.
- **Funcionalidades SQL reducidas:** Algunas características avanzadas
  (migraciones complejas, replicación, particionado) no están disponibles en
  SQLite.
- **Migración futura:** Si el proyecto crece, será necesario migrar a
  PostgreSQL u otro motor. El uso de SQLAlchemy como capa de abstracción
  minimiza el impacto de esta migración, pero no lo elimina por completo
  (diferencias en tipos de datos, comportamiento de transacciones, etc.).

### Criterio de revisión

Esta decisión debería revisarse si se cumple alguna de las siguientes
condiciones:

- El número de usuarios concurrentes supera las capacidades de SQLite.
- Se requiere desplegar la API en múltiples instancias detrás de un balanceador
  de carga.
- Se necesitan funcionalidades SQL no soportadas por SQLite (búsqueda de texto
  completo avanzada, tipos JSON nativos con indexación, etc.).
