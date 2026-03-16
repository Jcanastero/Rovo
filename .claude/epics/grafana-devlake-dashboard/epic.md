---
name: grafana-devlake-dashboard
status: backlog
created: 2026-03-12T00:00:00Z
updated: 2026-03-13T00:00:00Z
progress: 0%
prd: .claude/prds/grafana-devlake-dashboard.md
github: [Will be updated when synced to GitHub]
---

# Epic: Grafana DevLake Dashboard

## Overview

Implementar un dashboard de Grafana conectado al datasource MySQL `tfp-menv-mysql-devlake` con dos filtros (proyecto y version) y una tabla de issues. El entregable es un archivo JSON de dashboard importable en Grafana.

El alcance es intencionalmente minimo: dos template variables y un panel de tabla. No hay stats, pie charts ni time series en esta version.

---

## Architecture Decisions

- **Entregable como JSON exportado**: Dashboard creado en Grafana y exportado como `dashboard.json` versionable.
- **JOIN entre issues e issue_repo_commits**: El filtro de proyecto usa `repo_name` de `issue_repo_commits`, requiere JOIN con `issues` via `issue_id`.
- **Sin vistas SQL**: Toda la logica esta en la query del panel directamente.
- **Variable `$version` con Include All**: Grafana maneja el "sin filtro" con la opcion All value `%`, evitando logica condicional.
- **LIMIT 500**: Proteccion de performance en la tabla, sin afectar la query.

---

## Technical Approach

### Datasource
- `tfp-menv-mysql-devlake` (MySQL)
- Tablas: `issues` + `issue_repo_commits`

### Template Variables

| Variable | Tipo Grafana | Multi | Include All | Fuente |
|----------|--------------|-------|-------------|--------|
| `project` | Query (tabla `boards`) | No | No | IDs fijos de 9 boards en Jira |
| `version` | Query (tabla `issues`) | Si | Si (All value: `%`) | `fix_versions` filtrado por board seleccionado |

**Boards confirmados en DB:**

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

**Variable `project` — Query con label/value desde `boards`:**
```sql
SELECT name AS __text, id AS __value
FROM boards
WHERE id IN (
  'jira:JiraBoard:1:333',
  'jira:JiraBoard:1:99',
  'jira:JiraBoard:1:450',
  'jira:JiraBoard:1:431',
  'jira:JiraBoard:1:361',
  'jira:JiraBoard:1:356',
  'jira:JiraBoard:1:335',
  'jira:JiraBoard:1:87',
  'jira:JiraBoard:1:1618'
)
ORDER BY name
```
El dropdown muestra el `name` del board, pero el valor usado en las queries es el `id`.

**Variable `version` — Query (via JOIN con board_issues, filtrada por tipo):**
```sql
SELECT DISTINCT fix_versions
FROM issues i
INNER JOIN board_issues bi ON i.id = bi.issue_id
WHERE bi.board_id = '${project}'
  AND i.type IN ('REQUEST SUPPORT', 'BUG SUPPORT IT')
  AND fix_versions IS NOT NULL AND fix_versions != ''
ORDER BY fix_versions
```

### Panel: Issue Table

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

Notas:
- Campo Issue_key confirmado en Task 001: `issue_key` (varchar 255)
- JOIN confirmado: `board_issues` tiene columnas `board_id` e `issue_id`
- Valores de `type` confirmados en Task 001: `'REQUEST SUPPORT'`, `'BUG SUPPORT IT'` (todo mayúsculas)
- fix_versions confirmado: tipo TEXT, puede ser comma-separated; LIKE con All value `%` cubre ambos casos

### Layout

```
[Variables: project | version | (time range nativo)]
[Table: Issues - full width]
```

---

## Implementation Strategy

1. Validar schema (JOIN, nombre de campo Issue_key, formato de fix_versions)
2. Crear dashboard con variables
3. Crear tabla con query JOIN
4. Pulir y exportar JSON

---

## Task Breakdown

- [ ] **Task 1 - Schema Validation**: Inspeccionar `issues` e `issue_repo_commits`: confirmar JOIN key, nombre del campo Issue_key y formato de `fix_versions`.
- [ ] **Task 2 - Dashboard & Variables**: Crear dashboard con variables `project` y `version`.
- [ ] **Task 3 - Table Panel**: Crear el panel de tabla con JOIN y las 4 columnas (type, Issue_key, title, status).
- [ ] **Task 4 - Polish & Export**: Ajustar titulos, paginacion, layout y exportar el dashboard como JSON.

---

## Dependencies

- Grafana con acceso al datasource `tfp-menv-mysql-devlake`
- Permisos de lectura sobre `issues` e `issue_repo_commits`
- Permisos para crear dashboards en Grafana

---

## Success Criteria (Technical)

- La tabla carga datos reales con JOIN entre ambas tablas
- El filtro de proyecto es obligatorio y funcional
- El filtro de version es opcional y soporta multi-select
- Dashboard JSON importable en una instancia Grafana limpia

---

## Estimated Effort

- **Task 1 (Schema)**: 30 min
- **Task 2 (Variables)**: 30 min
- **Task 3 (Table)**: 45 min
- **Task 4 (Polish & Export)**: 45 min
- **Total estimado**: ~2.5 horas

**Riesgo principal**: Nombre del campo Issue_key y formato de `fix_versions` (se confirman en Task 1).

---

## Tasks Created
- [ ] 001.md - Schema Validation (parallel: false)
- [ ] 002.md - Dashboard Setup and Template Variables (parallel: false)
- [ ] 003.md - Table Panel (parallel: false)
- [ ] 004.md - Polish and Export (parallel: false)

Total tasks: 4
Parallel tasks: 0
Sequential tasks: 4
Estimated total effort: 2.5 hours
