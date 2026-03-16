Tengo un agente en ROVO llamado Sprint Recap, el cual esta configurado para que genere un reporte al finalizar el sprint que contiene la siguiente estructura:
Use this subject:  [{Team}] Sprint {#} Recap – {StartDate}–{EndDate}

Include exactly four sections:
 
Key goals achieved (2–4 bullets; link demos if applicable) :  Goals are short comments coming from the impact the stories achieved in the sprint on Users and other stakeholders (product, users or similar)
Key technical goals achieved (2-4 quality/infra/tech‑debt)
Major struggles needing escalation (owner, ask, due date)
Burndown chart (embed PNG + link to Jira Sprint Report)

El prompt del agente esta en el archivo C:\Users\jenny.canastero\OneDrive - AppGate Inc\Documents\ClaudeCode_Proyectos\ROVO\RovoSprintReportScrum.txt
se configuro para que pueda leer informacion de los siguientes proyectos:
	1. OPSDEOXYS
	2. 360 Adaptive Authentication (Space key: DID)
	3. RIRBA.
	4. DTA
	5. 360 Brand Guardian key: BG
	6. DIDSDK2
	7. CID
	8. Vault Bank
	9. RBA One Console
	10. Risk-Orchestrator

si Jenny Canastero lo ejecuta, el recolecta informacion y la genera, pero si otro owner de proyecto lo hace, entonces no le trae informacion, adicional a que en la seccion de burndown chart no esta generando la URL de manera correcta:
Quiero revisar que esta ocurriendo y corregirlo, porque deberia poderse hacer:
que cualquier owner de los proyectos listados, pueda ejecutar el agente y que le genere reporte con informacion reale
que genere reporte para sprints cerrados o abiertos
que genere la URL del burndownchart de manera correcta