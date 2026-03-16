---
issue: 4
stream: Fix burndown URL - Resolucion de boardId y sprintId
agent: general-purpose
started: 2026-03-16T00:00:00Z
status: completed
---

# Stream A: Fix burndown URL

## Scope
Resolver la contradiccion en el prompt entre "ID Consistency Rule" (que forzaba placeholders literales)
y la seccion de Burndown Chart (que pedia valores reales). Fix aplicado en RovoSprintReportScrum.txt.

## Files
- `ROVO/RovoSprintReportScrum.txt`

## Progress
- [x] "ID Consistency Rule" contradictorio reemplazado por "BURNDOWN ID RESOLUTION" con pasos claros
- [x] Seccion "Locate Burndown Data" actualizada para usar REAL_BOARD_ID y REAL_SPRINT_ID
- [x] Seccion final "Burndown Chart" del reporte actualizada — elimina placeholders literales
- [x] Fallback claro cuando IDs no pueden resolverse (mensaje navegable, no URL rota)

## Completed

Tres secciones del prompt modificadas:
1. Reemplazado "ID Consistency Rule" (forzaba {{boardId}}/{{sprintId}} literales) por proceso activo de resolucion en 3 pasos
2. Seccion "Locate Burndown Data" actualiza para referenciar REAL_BOARD_ID y REAL_SPRINT_ID
3. Seccion final "Burndown Chart" del template del reporte usa [ACTUAL_BOARD_ID] y [ACTUAL_SPRINT_ID] como recordatorio de sustitucion real

## Blocked
None
