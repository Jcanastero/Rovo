---
name: rovo-sprint-recap-scrum
description: Correccion del agente ROVO Sprint Recap para que cualquier owner pueda ejecutarlo con datos reales en sprints cerrados y abiertos, y genere correctamente la URL del burndown chart.
status: backlog
created: 2026-03-16T00:00:00Z
---

# PRD: ROVO Sprint Recap Scrum - Correccion de Acceso y Burndown Chart

## Executive Summary

El agente ROVO "Sprint Recap" genera reportes de cierre de sprint para los equipos de ingenieria de AppGate. Actualmente presenta dos problemas criticos: (1) solo el owner creador del agente obtiene datos reales al ejecutarlo sobre sprints cerrados, y (2) la URL del burndown chart se genera con placeholders literales o con IDs incorrectos. Este PRD define los cambios necesarios en el prompt del agente y, si aplica, en su configuracion en Atlassian, para que cualquier owner autorizado pueda ejecutarlo y obtener un reporte valido para cualquier sprint (abierto o cerrado).

---

## Problem Statement

### Problema 1 - Acceso a datos de sprints cerrados para otros owners

Cuando un owner distinto al creador del agente (Jenny Canastero) ejecuta el agente para un sprint **cerrado**, el agente responde que no encuentra issues y genera el reporte vacio. Sin embargo, el mismo owner puede ejecutarlo exitosamente sobre un sprint **abierto** del mismo proyecto.

Todos los owners afectados tienen permisos de lectura (Browse Projects) sobre los proyectos configurados. Esto descarta un problema de permisos Jira estandar y apunta a un problema en:

- La logica de fallback del JQL del agente (Alternative Query 2 usa `openSprints()`, lo que excluye sprints cerrados cuando el query principal falla para ciertos usuarios)
- O en la configuracion del agente ROVO que puede estar ejecutando consultas bajo el contexto del creador en lugar del caller para datos historicos

### Problema 2 - URL del burndown chart incorrecta

La URL generada del burndown chart presenta dos comportamientos erroneos:

- **Caso A**: Mantiene los placeholders literales `{{boardId}}` y `{{sprintId}}` sin resolver
- **Caso B**: Sustituye los placeholders con IDs incorrectos (de otro board o sprint), generando una URL que no lleva a ninguna pagina valida

Los usuarios invocan el agente unicamente con nombre del proyecto y nombre del sprint; nunca proveen IDs numericos. El agente debe resolver `boardId` y `sprintId` a partir de esos datos usando sus herramientas de busqueda en Jira, y cuando no puede hacerlo de manera confiable, actualmente falla silenciosamente.

### Por que es importante ahora

El reporte de sprint es un insumo critico para las reuniones de stakeholders de Engineering y Product Management. La incapacidad de otros owners de generar reportes propios crea una dependencia innecesaria en Jenny y reduce la adopcion de la herramienta en los 10 proyectos configurados.

---

## User Stories

### Personas

- **Owner de proyecto** (Dev Lead / ATL): Responsable de uno o mas de los 10 proyectos configurados. Ejecuta el agente al cerrar un sprint para generar el reporte de equipo.
- **Jenny Canastero (creadora del agente)**: Actualmente la unica usuaria que obtiene resultados correctos. Referencia de comportamiento esperado.

### Historia 1 - Owner ejecuta el agente para un sprint cerrado
**Como** owner de un proyecto Jira configurado en el agente,
**quiero** ejecutar el agente Sprint Recap para un sprint ya finalizado,
**para** obtener el reporte con la informacion real de ese sprint sin depender de la creadora del agente.

**Criterios de aceptacion:**
- El agente retorna issues reales del sprint cerrado cuando el owner tiene permisos Browse Projects
- El reporte incluye las 4 secciones completas: Key Goals, Key Technical Goals, Major Struggles, Burndown Chart
- El comportamiento es identico al que obtiene Jenny Canastero

### Historia 2 - Owner ejecuta el agente para un sprint abierto
**Como** owner de un proyecto,
**quiero** ejecutar el agente durante un sprint activo,
**para** obtener un reporte parcial del estado actual del sprint.

**Criterios de aceptacion:**
- El agente retorna los issues del sprint abierto correctamente
- Funciona para cualquier owner (comportamiento ya funcional, no debe romperse)

### Historia 3 - URL del burndown chart correcta y funcional
**Como** destinatario del reporte de sprint,
**quiero** que la URL del burndown chart del reporte sea valida y navegable,
**para** poder ver el grafico directamente sin buscar manualmente en Jira.

**Criterios de aceptacion:**
- La URL generada tiene el formato: `https://appgateinc.atlassian.net/jira/software/c/projects/{projectKey}/boards/{boardId}/reports/burndown-chart?sprint={sprintId}`
- `{projectKey}`, `{boardId}` y `{sprintId}` son valores numericos/textuales reales, nunca placeholders literales
- La URL es navegable y lleva directamente al burndown chart del sprint correcto
- Si el agente no puede resolver `boardId` o `sprintId`, indica explicitamente que no pudo obtener los IDs y pide al usuario que los provea, en lugar de generar una URL rota

---

## Requirements

### Functional Requirements

#### FR-1: JQL para sprints cerrados
- El agente debe usar explicitamente una query que incluya sprints en cualquier estado (abierto, cerrado, futuro):
  - Query recomendada: `project = '{{projectKey}}' AND sprint = '{{sprintName}}'`
  - La Alternative Query 2 actual (`sprint in openSprints()`) debe eliminarse o moverse al final del orden de fallback solo cuando se busca el sprint activo intencionalmente
  - El orden de fallback no debe nunca filtrar por `openSprints()` cuando el usuario provee un `sprintName` explicito

#### FR-2: Resolucion de boardId y sprintId
- El agente debe intentar resolver `boardId` y `sprintId` usando sus herramientas de busqueda (Jira API / ROVO tools) a partir del `projectKey` y `sprintName` provistos
- Proceso de resolucion:
  1. Buscar el board asociado al `projectKey`
  2. Buscar el sprint por nombre dentro de ese board
  3. Extraer los IDs numericos reales
- Si los IDs son encontrados: construir la URL con los valores reales
- Si los IDs **no** pueden ser resueltos confiablamente: incluir en el reporte una nota explicita indicando que la URL del burndown no pudo generarse automaticamente, e indicar al usuario como obtener los IDs manualmente en Jira (navegar al board > Reports > Burndown)
- **Nunca** generar una URL con placeholders literales ni con IDs de otro proyecto/sprint

#### FR-3: Consistencia de IDs en el reporte
- Si el agente encontro IDs mediante tools, esos valores reales deben usarse en la URL final
- Los IDs deben corresponder siempre al mismo proyecto y sprint que se esta reportando
- No debe existir discrepancia entre el `projectKey` reportado y el `boardId` usado en la URL

#### FR-4: Acceso universal para owners configurados
- Cualquier usuario Jira con permisos Browse Projects sobre alguno de los 10 proyectos configurados debe poder ejecutar el agente y obtener datos reales
- El agente no debe comportarse diferente segun quien lo invoque (salvo restricciones de permisos Jira del caller)
- Si hay una configuracion "Run As" o similar en ROVO que fija el contexto al creador, debe revisarse y ajustarse para usar el contexto del caller

#### FR-5: Manejo de "sin datos"
- Si el agente no encuentra issues tras ejecutar todas las queries de fallback, debe producir un reporte minimo que indique explicitamente la causa probable (permisos, nombre de sprint incorrecto, proyecto sin issues en ese sprint)
- Nunca fabricar datos

### Non-Functional Requirements

#### NFR-1: Sin regresion en sprints abiertos
- El comportamiento actual para sprints abiertos (que ya funciona para todos los owners) no debe verse afectado

#### NFR-2: Tiempo de generacion
- El reporte debe generarse en un tiempo comparable al actual (no introducir latencia significativa por la logica de resolucion de IDs)

#### NFR-3: Consistencia entre proyectos
- El fix debe funcionar de manera uniforme para los 10 proyectos configurados:
  1. OPSDEOXYS
  2. 360 Adaptive Authentication (DID)
  3. RIRBA
  4. DTA
  5. 360 Brand Guardian (BG)
  6. DIDSDK2
  7. CID
  8. Vault Bank
  9. RBA One Console
  10. Risk-Orchestrator

#### NFR-4: Idioma de salida
- El reporte final debe generarse siempre en ingles (comportamiento existente, no debe cambiar)

---

## Success Criteria

| Metrica | Condicion de exito |
|---|---|
| Acceso multi-owner sprints cerrados | Al menos 2 owners distintos a Jenny ejecutan el agente sobre `2026 - 11.0.0 - Sprint 13` (DID) y obtienen el reporte con datos reales |
| URL burndown correcta | La URL generada para el sprint de prueba es navegable y lleva al burndown chart correcto en Jira |
| Sin regresion sprints abiertos | El agente sigue funcionando correctamente para sprints abiertos para todos los owners |
| Sin placeholders en URL | En 10 ejecuciones consecutivas, la URL nunca contiene `{{boardId}}` o `{{sprintId}}` literales |
| Manejo de error claro | Cuando el agente no puede resolver IDs, el mensaje de error es explicito y accionable |

---

## Constraints & Assumptions

### Restricciones tecnicas
- El agente es un agente ROVO en Atlassian Intelligence; los cambios se realizan modificando el prompt del agente y/o su configuracion en el panel de administracion de ROVO
- No hay acceso a codigo fuente del runtime de ROVO; los cambios son a nivel de instrucciones del agente
- Las herramientas disponibles para el agente (Jira search, board lookup) son las provistas por la plataforma ROVO; no se pueden agregar integraciones externas

### Supuestos
- Todos los owners afectados tienen permisos Browse Projects en Jira sobre sus respectivos proyectos
- La API de Jira accesible por el agente permite buscar boards por project key y sprints por nombre
- El entorno de prueba (`360 Adaptive Authentication`, sprint `2026 - 11.0.0 - Sprint 13`) es representativo del caso de uso general
- Si existe una opcion "Run As" en la configuracion del agente ROVO que esta fijada al creador, Jenny tiene acceso para modificarla

### Limitaciones conocidas
- Si un owner no tiene permisos sobre un proyecto especifico, el agente correctamente no devolvera datos para ese proyecto; esto no es un bug sino comportamiento esperado
- El problema de sub-tareas apareciendo como items principales ya fue corregido en una iteracion anterior; este PRD no lo cubre

---

## Out of Scope

- Agregar nuevos proyectos a la lista de proyectos configurados del agente
- Cambiar el formato o estructura del reporte de sprint
- Integracion con herramientas distintas a Jira/ROVO (Confluence, Slack, email)
- Automatizacion del trigger del agente (que se ejecute automaticamente al cerrar un sprint)
- Soporte para proyectos Jira que no esten en la lista de los 10 configurados
- Modificaciones a la seccion "Issues without Work Category"
- Cambios en la clasificacion de Work Categories

---

## Dependencies

### Dependencias externas
- **Plataforma ROVO / Atlassian Intelligence**: Los cambios al prompt y configuracion dependen de la interfaz de administracion de ROVO en Atlassian
- **Jira REST API**: La resolucion de boardId y sprintId depende de que las herramientas del agente puedan hacer lookup de boards y sprints por nombre
- **Permisos Jira**: Los owners deben tener Browse Projects activo sobre sus proyectos

### Dependencias internas
- Acceso de Jenny al panel de administracion del agente ROVO para aplicar cambios
- Disponibilidad de al menos un owner de proyecto distinto a Jenny para validacion del fix

---

## Test Plan

### Sprint de prueba
- **Proyecto**: 360 Adaptive Authentication (project key: DID)
- **Sprint**: 2026 - 11.0.0 - Sprint 13 (sprint cerrado)

### Casos de prueba

| # | Caso | Actor | Sprint | Resultado esperado |
|---|---|---|---|---|
| TC-01 | Sprint cerrado - owner distinto | Owner de DID (no Jenny) | 2026 - 11.0.0 - Sprint 13 | Reporte con datos reales, 4 secciones completas |
| TC-02 | Sprint cerrado - Jenny | Jenny | 2026 - 11.0.0 - Sprint 13 | Comportamiento igual al actual (sin regresion) |
| TC-03 | Sprint abierto - owner distinto | Owner de DID (no Jenny) | Sprint activo de DID | Reporte con datos reales (sin regresion) |
| TC-04 | URL burndown correcta | Cualquier owner | 2026 - 11.0.0 - Sprint 13 | URL navegable sin placeholders |
| TC-05 | Sprint inexistente | Cualquier owner | "Sprint Falso XYZ" | Mensaje de error claro, no reporte fabricado |
| TC-06 | Otro proyecto - owner distinto | Owner de OPSDEOXYS | Ultimo sprint cerrado de OPSDEOXYS | Reporte con datos reales |

---

## Implementation Notes

### Cambio prioritario en el prompt (FR-1)
Eliminar o reubicar la Alternative Query 2 (`sprint in openSprints()`) para que no se use como fallback cuando el usuario provee un `sprintName` explicito. Agregar instruccion explicita de que la busqueda de sprint debe incluir sprints en cualquier estado.

### Cambio en logica de burndown URL (FR-2)
Agregar una seccion de instrucciones al prompt que:
1. Especifique el proceso paso a paso para buscar el board del proyecto y el sprint por nombre
2. Indique que los IDs obtenidos de tools son los unicos autorizados para la URL
3. Defina el comportamiento cuando los IDs no pueden resolverse (mensaje de error + instrucciones manuales)
4. Prohiba explicitamente usar placeholders literales o IDs de proyectos distintos

### Revision de configuracion ROVO (FR-4)
Verificar en el panel de administracion del agente ROVO si existe una opcion de contexto de ejecucion ("Run As creator" vs "Run As caller") y cambiarla a "Run As caller" si aplica.
