---
issue: 2
stream: Reproduccion del error con otro owner
agent: manual-coordinacion
started: 2026-03-16T00:00:00Z
status: completed
---

# Stream B: Reproduccion del error con otro owner

## Scope
Coordinar con un owner de DID (distinto a Jenny) para ejecutar el agente sobre el sprint cerrado y capturar la respuesta exacta.

## Files
- No aplica (prueba manual en ROVO)

## Progress

- [x] Owner de DID contactado (Laura Nino - ATL de DID)
- [x] Agente ejecutado sobre Sprint 15 de DID
- [x] Respuesta del agente capturada
- [x] Log de ejecucion analizado
- [x] Causa raiz identificada

## Completed

- **Owner que ejecuto la prueba**: Laura Nino (ATL de DID)
- **Sprint usado**: 2026 - 11.0.0 - Sprint 15
- **Resultado**: Reporte vacio — "No issues found for project DID in sprint"

## Causa Raiz Real (CRITICA)

La instruccion `GLOBAL SCOPE PERMISSION` del prompt hace que ROVO use `searchSource: GLOBAL_SCOPE`, lo cual inyecta automaticamente un filtro:

```
project IN (10017,10065,10018,10067,10131,10036,10142,10831,10079,10111,10196)
```

Para Laura Nino, los project IDs `10017`, `10018` y `10036` **no existen en su contexto** de Jira, generando errores JQL que hacen que TODAS las queries devuelvan 0 resultados — incluso para DID (ID `10067`) al que si tiene acceso.

**La causa raiz NO es `openSprints()`** — es el `GLOBAL SCOPE PERMISSION` que triggerea la busqueda global con IDs no accesibles para todos los usuarios.

## Fix requerido en Task #3

Eliminar la instruccion `GLOBAL SCOPE PERMISSION` del prompt. El agente debe usar STRICT_PROJECT_SCOPING unicamente — consultar SOLO el `projectKey` provisto, nunca expandir a busqueda global con todos los IDs de proyectos configurados.

## Blocked
None
