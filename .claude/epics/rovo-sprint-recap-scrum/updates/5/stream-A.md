---
issue: 5
stream: Aplicar prompt actualizado en Atlassian Studio
agent: manual-jenny
started: 2026-03-16T00:00:00Z
status: in_progress
---

# Stream A: Aplicar prompt en Atlassian Studio

## Scope
Copiar el contenido de `ROVO/RovoSprintReportScrum.txt` al panel del agente en Atlassian y ejecutar smoke test.

## Files
- No aplica (accion manual en Atlassian Studio)
- Fuente: `ROVO/RovoSprintReportScrum.txt`

## Pasos a ejecutar

1. Abrir Atlassian Studio > Sprint Recap ATL > Escenarios > Sprint Recap ATL (Predeterminado)
2. En la seccion **Instrucciones**, seleccionar todo el texto actual y borrarlo
3. Copiar el contenido completo de `ROVO/RovoSprintReportScrum.txt` y pegarlo
4. Guardar los cambios
5. Hacer clic en **Prueba** (boton arriba a la derecha) y ejecutar con:
   - Project: DID
   - Sprint: 2026 - 11.0.0 - Sprint 15
6. Verificar que el reporte devuelve datos reales (no vacio)

## Progress

- [ ] Panel Atlassian Studio abierto
- [ ] Instrucciones del agente reemplazadas con el prompt actualizado
- [ ] Cambios guardados
- [ ] Smoke test ejecutado (Jenny, DID, Sprint 15)
- [ ] Resultado verificado: reporte con datos reales

## Completed
(pendiente)

## Blocked
None
