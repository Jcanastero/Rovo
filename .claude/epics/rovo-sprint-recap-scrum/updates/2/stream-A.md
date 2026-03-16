---
issue: 2
stream: Verificacion de configuracion ROVO
agent: manual-jenny
started: 2026-03-16T00:00:00Z
status: completed
---

# Stream A: Verificacion de configuracion ROVO

## Scope
Revisar el panel de administracion del agente ROVO en Atlassian Intelligence y documentar si existe opcion "Run As" y su valor actual.

## Files
- No aplica (accion manual en panel Atlassian)

## Progress

- [x] Panel ROVO accedido (Atlassian Studio > Sprint Recap ATL)
- [x] Opcion "Run As" revisada - **NO EXISTE** en ROVO Studio
- [x] Seccion "Usuarios y permisos" revisada
- [x] Resultado final documentado

## Completed

- Panel del agente accedido: "Sprint Recap ATL" en Atlassian Studio
- Confirmado: ROVO Studio NO tiene opcion "Run As" / "Execute As"
- **Hipotesis A DESCARTADA**: El agente ejecuta con credenciales del caller por diseno de la plataforma
- Prompt visible confirma que `{{boardId}}` y `{{sprintId}}` son variables que deben resolverse en tiempo de ejecucion

## Working On
Ninguno - stream completado.

## Conclusion Final

- "Open to all users" = ON: cualquier usuario del sitio Atlassian puede invocar el agente
- Roles Manager/Editor son para administrar el agente, no para usarlo
- **Hipotesis A DESCARTADA DEFINITIVAMENTE**: el problema NO es de permisos ni de ejecucion con credenciales del creador
- **Causa raiz confirmada: Hipotesis B** — el problema esta en el JQL del prompt (fallback a `openSprints()`) y en la resolucion de IDs para el burndown chart
- No se requiere ningun cambio en la seccion "Usuarios y permisos"

## Blocked
None
