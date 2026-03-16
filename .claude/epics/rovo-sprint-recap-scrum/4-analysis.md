---
issue: 4
title: Fix burndown URL - Resolucion de boardId y sprintId
analyzed: 2026-03-16T00:00:00Z
estimated_hours: 1
parallelization_factor: 1.0
---

# Parallel Work Analysis: Issue #4

## Overview

El prompt tiene una contradiccion critica que causa el problema de la URL del burndown:

- La seccion "ID Consistency Rule" (lineas 26-29) dice que el agente DEBE usar los placeholders
  literales {{boardId}} y {{sprintId}} en la salida final.
- La seccion final de Burndown Chart dice que DEBE reemplazar esos valores con los reales.

El agente obedece la primera regla (mas arriba en el prompt) y escribe los placeholders
literalmente, o busca IDs en contexto incorrecto y genera valores erroneos.

El fix reemplaza el "ID Consistency Rule" contradictoio con un proceso claro de resolucion
de IDs, y actualiza las instrucciones del burndown chart para que sean coherentes.

## Parallel Streams

### Stream A: Fix del proceso de resolucion de IDs para burndown URL
**Scope**: Unico stream — modificar `ROVO/RovoSprintReportScrum.txt`:
1. Reemplazar "ID Consistency Rule" contradictorio con proceso activo de resolucion de IDs
2. Actualizar seccion "Locate Burndown Data" con pasos concretos de busqueda
3. Definir comportamiento de fallback claro cuando IDs no pueden resolverse
**Files**:
- `ROVO/RovoSprintReportScrum.txt`
**Agent Type**: general-purpose
**Can Start**: immediately
**Estimated Hours**: 1
**Dependencies**: issue #3 completado (mismo archivo, ya procesado)

## Coordination Points
- `ROVO/RovoSprintReportScrum.txt` modificado en issue #3 — leer version actualizada antes de editar

## Parallelization Strategy
**Recommended Approach**: sequential (archivo unico)

## Expected Timeline
- Wall time: ~1 hora
- Total work: 1 hora

## Notes
- La contradiccion entre "ID Consistency Rule" y la seccion de Burndown es la causa raiz del Defecto 2
- El fix debe ser coherente: un solo lugar en el prompt define como se obtienen y usan los IDs
