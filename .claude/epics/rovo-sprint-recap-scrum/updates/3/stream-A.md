---
issue: 3
stream: Fix JQL - GLOBAL SCOPE PERMISSION y openSprints()
agent: general-purpose
started: 2026-03-16T00:00:00Z
status: completed
---

# Stream A: Fix JQL del prompt ROVO

## Scope
Modificar `ROVO/RovoSprintReportScrum.txt` para:
1. Eliminar `GLOBAL SCOPE PERMISSION` — causa raiz confirmada
2. Eliminar `openSprints()` del fallback cuando `sprintName` es provisto

## Files
- `ROVO/RovoSprintReportScrum.txt`

## Progress
- [x] GLOBAL SCOPE PERMISSION reemplazado por STRICT PROJECT ONLY
- [x] openSprints() removido — reemplazado por query que funciona para sprints en cualquier estado
- [x] Nota IMPORTANT agregada prohibiendo openSprints() cuando sprintName es provisto
- [x] Descripcion del fallback en "No Results Handling" actualizada
- [x] Archivo RovoSprintReportScrum.txt guardado
