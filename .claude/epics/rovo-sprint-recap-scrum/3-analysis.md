---
issue: 3
title: Fix JQL - Eliminar GLOBAL SCOPE PERMISSION y openSprints() del flujo de fallback
analyzed: 2026-03-16T00:00:00Z
estimated_hours: 0.5
parallelization_factor: 1.0
---

# Parallel Work Analysis: Issue #3

## Overview

Fix al prompt del agente ROVO para corregir la causa raiz confirmada en issue #2:
la instruccion `GLOBAL SCOPE PERMISSION` hace que ROVO use `searchSource: GLOBAL_SCOPE`
inyectando un filtro `project IN (...)` con IDs de proyectos no accesibles para todos
los usuarios, causando que todas las JQL queries fallen con 0 resultados.

Adicionalmente se elimina `openSprints()` del flujo de fallback cuando `sprintName` es provisto.

**Archivo a modificar**: `ROVO/RovoSprintReportScrum.txt`

## Parallel Streams

### Stream A: Fix completo del prompt JQL
**Scope**: Unico stream — modificar `RovoSprintReportScrum.txt` con los dos cambios necesarios:
1. Eliminar/restringir `GLOBAL SCOPE PERMISSION` para evitar busqueda global con project IDs
2. Eliminar `openSprints()` del fallback cuando `sprintName` es provisto
**Files**:
- `ROVO/RovoSprintReportScrum.txt`
**Agent Type**: general-purpose
**Can Start**: immediately
**Estimated Hours**: 0.5
**Dependencies**: none (causa raiz confirmada en issue #2)

## Coordination Points

### Shared Files
- `ROVO/RovoSprintReportScrum.txt` — este issue y el #4 modifican el mismo archivo
- Completar #3 antes de iniciar #4 para evitar conflictos

## Parallelization Strategy
**Recommended Approach**: sequential (tarea de archivo unico, no hay paralelizacion util)

## Expected Timeline
- Wall time: ~30 minutos
- Total work: 0.5 horas

## Notes
- El fix principal es eliminar `GLOBAL SCOPE PERMISSION` — esto resuelve el problema de los IDs invalidos
- El fix de `openSprints()` es secundario pero igualmente necesario para sprints cerrados
- Ambos fixes van en el mismo archivo en una sola operacion
