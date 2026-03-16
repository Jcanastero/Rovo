---
issue: 5
title: Aplicar cambios en Atlassian - Prompt actualizado
analyzed: 2026-03-16T00:00:00Z
estimated_hours: 0.5
parallelization_factor: 1.0
---

# Parallel Work Analysis: Issue #5

## Overview

Tarea manual: copiar el contenido actualizado de `ROVO/RovoSprintReportScrum.txt`
al panel del agente "Sprint Recap ATL" en Atlassian Studio y ejecutar un smoke test.

La configuracion "Run As" fue descartada en issue #2 — no hay cambio de configuracion requerido.
El unico cambio es reemplazar las instrucciones del agente con el prompt actualizado.

## Parallel Streams

### Stream A: Aplicar prompt en Atlassian Studio
**Scope**: Unico stream — accion manual en el panel de Atlassian Intelligence
**Files**: No aplica (accion en UI de Atlassian)
**Agent Type**: manual-jenny
**Can Start**: immediately (issues #3 y #4 completados)
**Estimated Hours**: 0.5
**Dependencies**: issues #3 y #4 completados (ya lo estan)

## Parallelization Strategy
**Recommended Approach**: sequential (tarea manual unica)

## Expected Timeline
- Wall time: ~30 minutos

## Notes
- El prompt actualizado esta en: ROVO/RovoSprintReportScrum.txt
- No tocar "Usuarios y permisos" — configuracion correcta
- Smoke test: Jenny ejecuta el agente sobre DID / Sprint 15 y verifica que devuelve datos
