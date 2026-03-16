---
name: rovo-sprint-recap-scrum
status: backlog
created: 2026-03-16T00:00:00Z
progress: 0%
prd: .claude/prds/rovo-sprint-recap-scrum.md
github: https://github.com/Jcanastero/Rovo/issues/1
---

# Epic: ROVO Sprint Recap Scrum - Correccion de Acceso y Burndown Chart

## Overview

Este epic cubre las modificaciones al prompt del agente ROVO "Sprint Recap" y la revision de su configuracion en Atlassian para corregir dos defectos criticos:

1. **Acceso multi-owner en sprints cerrados**: El agente actualmente solo devuelve datos a la creadora (Jenny Canastero) cuando el sprint esta cerrado. Cualquier owner con permisos Jira debe poder ejecutarlo y obtener resultados reales.
2. **URL del burndown chart**: La URL generada contiene placeholders literales o IDs incorrectos. Debe resolverse automaticamente a partir del project key y sprint name, o notificar al usuario de forma accionable si no puede.

El trabajo es 100% de configuracion/prompt engineering sobre la plataforma ROVO de Atlassian. No hay codigo de aplicacion que desarrollar.

---

## Architecture Decisions

- **Sin desarrollo de codigo**: Todos los cambios son modificaciones al prompt del agente ROVO y, si aplica, a su configuracion en el panel de administracion de Atlassian Intelligence.
- **Prompt como fuente de verdad**: El archivo `ROVO/RovoSprintReportScrum.txt` es la fuente del prompt actual. Los cambios se documentan ahi y luego se copian al agente en Atlassian.
- **Validacion manual**: La plataforma ROVO no tiene entorno de CI/CD; la validacion se hace ejecutando el agente con el sprint de prueba definido en el PRD.
- **Cambio minimo**: Solo se modifican las secciones del prompt directamente relacionadas con los dos defectos. El resto del comportamiento (clasificacion por Work Category, formato del reporte, filtros de sub-tasks) se preserva sin cambios.

---

## Technical Approach

### Defecto 1: Acceso a sprints cerrados (JQL y configuracion ROVO)

**Causa raiz a investigar:**
- **Opcion A - Configuracion ROVO**: El agente puede estar configurado con "Run As Creator" en lugar de "Run As Caller". Verificar en el panel de administracion del agente en Atlassian y cambiar si aplica.
- **Opcion B - JQL fallback**: La Alternative Query 2 usa `sprint in openSprints()`, que silenciosamente excluye sprints cerrados cuando las queries anteriores fallan para ciertos usuarios. Eliminar esta query del flujo de fallback cuando el usuario provee un `sprintName` explicito.

**Cambio en el prompt:**
- Agregar instruccion explicita: cuando `sprintName` es provisto, las queries NUNCA deben usar `openSprints()` o `closedSprints()` como filtro; siempre usar `sprint = '{{sprintName}}'` para obtener cualquier sprint independientemente de su estado.
- Reordenar el flujo de fallback para que Alternative Query 2 (`openSprints()`) solo aplique cuando el usuario explicitamente no provee un sprint name.

### Defecto 2: Resolucion de boardId y sprintId para la URL del burndown

**Causa raiz**: El agente no tiene instrucciones claras sobre como buscar los IDs y que hacer cuando no puede encontrarlos.

**Cambio en el prompt - proceso de resolucion de IDs:**
1. Buscar el board Jira asociado al `projectKey` usando las herramientas disponibles
2. Dentro de ese board, buscar el sprint cuyo nombre coincida con `sprintName`
3. Extraer `boardId` y `sprintId` de los resultados
4. Construir la URL con los valores reales obtenidos
5. Si los IDs no pueden resolverse: incluir en el reporte una nota de error clara con instrucciones manuales (navegar a Jira > Proyecto > Board > Reports > Burndown Chart)
6. **Prohibicion explicita**: nunca escribir `{{boardId}}` o `{{sprintId}}` como texto literal en la URL final; nunca usar IDs de un proyecto distinto al solicitado

### Configuracion Atlassian (si aplica)
- Revisar en el panel del agente ROVO si existe opcion de contexto de ejecucion
- Cambiar a "Run As Caller" si esta en "Run As Creator"
- Verificar que los 10 proyectos configurados siguen accesibles tras el cambio

---

## Implementation Strategy

### Fase 1: Diagnostico (prerequisito)
Antes de modificar el prompt, confirmar la causa raiz del Defecto 1:
- Revisar configuracion del agente en Atlassian (opcion Run As)
- Ejecutar el agente como otro owner sobre el sprint de prueba y capturar el mensaje exacto de error/respuesta vacia
- Determinar si el problema es de configuracion, de JQL, o ambos

### Fase 2: Correccion del prompt
- Modificar `ROVO/RovoSprintReportScrum.txt` con los cambios identificados
- Aplicar los cambios en el agente ROVO en Atlassian

### Fase 3: Correccion de configuracion (si aplica)
- Cambiar opcion "Run As" en el panel de administracion si corresponde

### Fase 4: Validacion
- Ejecutar los 6 casos de prueba del PRD
- Validar con al menos un owner distinto a Jenny

### Estrategia de testing
- Sprint de prueba: `360 Adaptive Authentication` / `2026 - 11.0.0 - Sprint 13`
- Actor primario de validacion: owner de DID distinto a Jenny
- Validacion secundaria: al menos un owner de otro proyecto (ej. OPSDEOXYS)

---

## Task Breakdown

- [ ] **Task 1 - Diagnostico**: Verificar configuracion "Run As" del agente ROVO en Atlassian y reproducir el error del Defecto 1 con otro owner sobre el sprint de prueba. Documentar causa raiz confirmada.
- [ ] **Task 2 - Fix JQL fallback**: Modificar el prompt para eliminar `openSprints()` del flujo de fallback cuando `sprintName` es provisto. Actualizar archivo `RovoSprintReportScrum.txt`.
- [ ] **Task 3 - Fix resolucion de IDs para burndown URL**: Agregar al prompt el proceso paso a paso de busqueda de `boardId` y `sprintId`, el comportamiento de error cuando no se pueden resolver, y la prohibicion de placeholders literales. Actualizar archivo `RovoSprintReportScrum.txt`.
- [ ] **Task 4 - Aplicar cambios en Atlassian**: Copiar el prompt actualizado al agente ROVO en el panel de Atlassian Intelligence. Ajustar configuracion "Run As" si aplica.
- [ ] **Task 5 - Validacion**: Ejecutar los 6 casos de prueba del PRD. Confirmar URL navegable y datos reales para owners distintos a Jenny.

## Tasks Created

- [ ] [#2](https://github.com/Jcanastero/Rovo/issues/2) - Diagnostico - Causa raiz de acceso a sprints cerrados (parallel: false)
- [ ] [#3](https://github.com/Jcanastero/Rovo/issues/3) - Fix JQL - Eliminar openSprints() del flujo de fallback (parallel: false, depends: #2)
- [ ] [#4](https://github.com/Jcanastero/Rovo/issues/4) - Fix burndown URL - Resolucion de boardId y sprintId (parallel: false, depends: #2)
- [ ] [#5](https://github.com/Jcanastero/Rovo/issues/5) - Aplicar cambios en Atlassian - Prompt y configuracion Run As (parallel: false, depends: #3, #4)
- [ ] [#6](https://github.com/Jcanastero/Rovo/issues/6) - Validacion - Casos de prueba con owners reales (parallel: false, depends: #5)

Total tasks: 5
Parallel tasks: 0
Sequential tasks: 5
Estimated total effort: 4-6 horas

---

## Dependencies

| Dependencia | Tipo | Detalle |
|---|---|---|
| Panel de administracion ROVO | Acceso | Jenny necesita acceso de admin al agente en Atlassian Intelligence |
| Otro owner de DID | Validacion | Necesario para TC-01 y TC-03 del plan de pruebas |
| Sprint de prueba activo o cerrado | Datos | `2026 - 11.0.0 - Sprint 13` debe existir en Jira con issues reales |
| Herramientas Jira del agente ROVO | Funcional | El agente debe poder hacer lookup de boards y sprints por nombre via sus tools |

---

## Success Criteria (Technical)

- El prompt actualizado en `RovoSprintReportScrum.txt` no contiene la query `openSprints()` en el flujo de fallback cuando `sprintName` es provisto
- El prompt incluye instrucciones explicitas de resolucion de boardId/sprintId y manejo de error para URL del burndown
- TC-01: Owner distinto a Jenny obtiene reporte con datos reales para `2026 - 11.0.0 - Sprint 13` (DID)
- TC-04: URL generada para ese sprint es navegable y llega al burndown chart correcto
- TC-02 y TC-03: Sin regresion para Jenny ni para sprints abiertos
- TC-05: Mensaje de error claro cuando el sprint no existe

---

## Estimated Effort

| Tarea | Esfuerzo estimado |
|---|---|
| Diagnostico y reproduccion | 1-2 horas |
| Fix JQL fallback en prompt | 30 minutos |
| Fix logica de burndown URL en prompt | 1 hora |
| Aplicar cambios en Atlassian | 30 minutos |
| Validacion con casos de prueba | 1-2 horas |
| **Total** | **4-6 horas** |

**Riesgo principal**: Si la causa raiz del Defecto 1 es la opcion "Run As" de ROVO (no el JQL), el fix es solo de configuracion y toma minutos. Si requiere cambios adicionales en permisos Jira o en la arquitectura del agente, puede escalar.
