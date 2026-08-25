# wincc-rt-pro-opc-server

Base de conocimiento: exponer tags de **WinCC Runtime Professional V18** (PC Station) por **OPC DA** y **OPC UA**, validado con PLC **S7-315-2DP** vía **MPI**.

## Contenido

Todo el material está en [`conocimiento/`](conocimiento/):

| Archivo | Descripción |
|---|---|
| [OPC-UA-WinCC-RT-Professional-V18.md](conocimiento/OPC-UA-WinCC-RT-Professional-V18.md) | Informe técnico: cronología, desvíos, comandos PowerShell, procedimiento limpio |
| [OPC-PowerShell-Diagnostico.md](conocimiento/OPC-PowerShell-Diagnostico.md) | Comandos PowerShell en orden cronológico (reseña + acción esperada) para otra estación / agente |
| [GitHub-Chuleta.md](conocimiento/GitHub-Chuleta.md) | Chuleta Git: clone, pull, status, commit y push (PowerShell) |
| [Chat-Resumen-OPC.md](conocimiento/Chat-Resumen-OPC.md) | Hilo de chat condensado (sin consultas repetidas) |
| [AGENT-CONTEXT.md](conocimiento/AGENT-CONTEXT.md) | Contexto para un agente de IA |
| [AGENT-PROMPT.md](conocimiento/AGENT-PROMPT.md) | Prompt listo para otra estación de trabajo |

## Resultado de referencia (2026-08-18)

- **OPC DA:** ProgID `OPCServer.WinCC_SCADA.1` (OPC Expert)
- **OPC UA:** `opc.tcp://localhost:4861`, servicio `OpcUaServerWinCCPro` (UaExpert + Trust certificate)

## Nota

Este repositorio es documentación. El push a GitHub lo hace el usuario de forma manual. Comandos: [`conocimiento/GitHub-Chuleta.md`](conocimiento/GitHub-Chuleta.md).
