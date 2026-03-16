---
name: grafana-devlake-dashboard
description: Dashboard de Grafana para visualizar issues de DevLake filtrados por proyecto y version, con tabla de resultados
status: backlog
created: 2026-03-12T00:00:00Z
updated: 2026-03-13T00:00:00Z
---

# PRD: Grafana DevLake Dashboard

## Executive Summary

Crear un dashboard en Grafana conectado al datasource MySQL `tfp-menv-mysql-devlake` que permita al equipo consultar los issues de un proyecto especifico. El dashboard ofrece dos filtros (proyecto y version) y muestra los resultados en una tabla con tipo, clave, titulo y estado del issue.

---

## Problem Statement

El equipo necesita una forma rapida de consultar el estado de los issues de un proyecto en DevLake sin necesidad de ejecutar SQL manualmente. El dashboard centraliza esta informacion con filtros sencillos y una vista tabular clara.

---

## User Stories

### Persona Principal: Engineering Manager / Tech Lead / Developer

**Historia 1 - Seleccion de proyecto**
Como usuario, quiero seleccionar el proyecto a consultar desde una lista desplegable para ver solo los issues que me interesan.

Criterios de aceptacion:
- Dropdown con lista fija de 9 proyectos definidos (no consulta dinamica al DB)
- Al seleccionar un proyecto, la tabla de issues se filtra
- El filtro es obligatorio (sin proyecto no se muestran datos)

**Historia 2 - Filtro por version**
Como product owner, quiero filtrar por version de producto para ver los issues de una release especifica.

Criterios de aceptacion:
- Dropdown con valores de `fix_versions` de la tabla `issues`
- Seleccion opcional (sin version seleccionada muestra todos los issues del proyecto)
- Soporta seleccion multiple

**Historia 3 - Tabla de issues**
Como usuario, quiero ver la lista de issues del proyecto/version seleccionado en una tabla clara con la informacion clave de cada issue.

Criterios de aceptacion:
- Tabla con columnas: type, Issue_key, title, status
- Resultado filtrado por el proyecto y/o version seleccionados
- Solo muestra issues de tipo **"Request Support"** y **"Bug Support IT"**
- Tabla paginada y ordenable

---

## Requirements

### Functional Requirements

#### Filtros / Variables de Grafana

| Variable | Tipo Grafana | Tabla | Obligatorio |
|----------|--------------|-------|-------------|
| `$project` | Query | `boards` (filtrada por IDs) | Si |
| `$version` | Query | `issues` via `board_issues` | No |

#### Panel: Tabla de Issues

Columnas visibles:
- `type` — Tipo de issue (Bug, Story, Task, etc.)
- `Issue_key` — Identificador unico del issue (campo a confirmar en Task 001)
- `title` — Titulo del issue
- `status` — Estado actual del issue

La tabla filtra por board via JOIN: `issues` → `board_issues` → `board_id = '${project}'`

#### Proyectos (boards por ID)

| Board ID | Nombre en DB | Producto |
|----------|--------------|---------|
| jira:JiraBoard:1:333 | RIRBA Board | 360 Risk Control Sensors |
| jira:JiraBoard:1:99 | Mobile SDK Board | Mobile SDKs |
| jira:JiraBoard:1:450 | CID board | AI/ML Services |
| jira:JiraBoard:1:431 | BG General | 360 Brand Guardian |
| jira:JiraBoard:1:361 | Vault Bank | Demos |
| jira:JiraBoard:1:356 | ROC board | OneConsole Core |
| jira:JiraBoard:1:335 | DTA board | 360 Risk Control Base Package |
| jira:JiraBoard:1:87 | DetectID Server | 360 Adaptive Authentication |
| jira:JiraBoard:1:1618 | BG board | 360 Bran Guardian |

Query de referencia:
```sql
SELECT
  i.type        AS "Type",
  i.issue_key   AS "Issue Key",
  i.title       AS "Title",
  i.status      AS "Status"
FROM issues i
INNER JOIN board_issues bi ON i.id = bi.issue_id
WHERE bi.board_id = '${project}'
  AND i.type IN ('REQUEST SUPPORT', 'BUG SUPPORT IT')
  AND (i.fix_versions LIKE '${version}' OR '${version}' = '%')
ORDER BY i.created_date DESC
LIMIT 500
```

#### Variables de Template

**Variable `project` — Query desde `boards` (muestra nombre, filtra por ID):**
```sql
SELECT name AS __text, id AS __value
FROM boards
WHERE id IN (
  'jira:JiraBoard:1:333', 'jira:JiraBoard:1:99', 'jira:JiraBoard:1:450',
  'jira:JiraBoard:1:431', 'jira:JiraBoard:1:361', 'jira:JiraBoard:1:356',
  'jira:JiraBoard:1:335', 'jira:JiraBoard:1:1618', 'jira:JiraBoard:1:87'
)
ORDER BY name
```

**Variable `version` — Query via `board_issues` (filtrada por tipo):**
```sql
SELECT DISTINCT fix_versions
FROM issues i
INNER JOIN board_issues bi ON i.id = bi.issue_id
WHERE bi.board_id = '${project}'
  AND i.type IN ('REQUEST SUPPORT', 'BUG SUPPORT IT')
  AND fix_versions IS NOT NULL AND fix_versions != ''
ORDER BY fix_versions;
```

### Non-Functional Requirements

- Tiempo de carga de la tabla < 5 segundos con filtros aplicados
- Solo lectura sobre el datasource `tfp-menv-mysql-devlake`
- Tabla con maximo 500 filas (LIMIT) para proteger performance

---

## Success Criteria

| Metrica | Objetivo |
|---------|----------|
| Carga de la tabla | < 5 segundos |
| Proyectos visibles | Los 9 namespaces del listado definido |
| Precision de datos | 0 discrepancias vs SQL directo |

---

## Constraints & Assumptions

**Restricciones**
- El datasource `tfp-menv-mysql-devlake` ya existe y esta configurado en Grafana
- No se crean vistas ni stored procedures
- El campo `fix_versions` puede tener multiples valores (formato a confirmar)

**Supuestos**
- La tabla `issues` tiene un campo `name_space` (o equivalente) con los valores de proyecto
- La tabla `issues` tiene los campos: `id`, `number` (o similar para Issue_key), `title`, `type`, `status`, `fix_versions`, `created_date`, `name_space`
- Si `name_space` no existe en `issues`, se usara JOIN con `issue_repo_commits` via `repo_name`
- Grafana tiene permisos de lectura sobre el datasource

---

## Out of Scope

- Paneles de metricas (stat panels, pie charts, time series)
- Alertas o notificaciones
- Exportacion a CSV/Excel
- Modificacion del esquema de base de datos
- Proyectos fuera del listado de 9 namespaces definidos

---

## Dependencies

- Grafana con datasource `tfp-menv-mysql-devlake` configurado
- Acceso de lectura a tabla `issues` (y posiblemente `issue_repo_commits`)
- Confirmacion del nombre del campo de proyecto (`name_space` u otro) en Task 001
- Confirmacion del nombre del campo Issue_key en Task 001

---

## Revision Log

| Fecha | Cambio |
|-------|--------|
| 2026-03-12 | Version inicial |
| 2026-03-13 | Simplificado: solo tabla como panel; eliminados stats, pie charts y time series |
| 2026-03-13 | Proyecto: lista fija de 9 namespaces en lugar de query dinamico; campo name_space a confirmar |
