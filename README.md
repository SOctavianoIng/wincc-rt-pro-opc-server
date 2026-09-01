# wincc-rt-pro-opc-server

Base de conocimiento: exponer tags de **WinCC Runtime Professional V18** (PC Station) por **OPC DA** y **OPC UA**, validado con PLC **S7-315-2DP** vía **MPI**.

**Estado:** trabajo **cerrado y entregado al cliente** (agosto 2026).  
**Versión entregada:** [`Informe-Tecnico-Comunicacion-OPC-UA-WinCC-RT (Rev.1).docx`](conocimiento/entregable/Informe-Tecnico-Comunicacion-OPC-UA-WinCC-RT%20(Rev.1).docx) (revisión manual sobre la plantilla IndustrialLAB).

## Estructura

```
.rules/                  Reglas y prompt para agentes de IA
.cursor/rules/           Regla Cursor (carga el contexto de .rules/)
conocimiento/
  entregable/            Informe al cliente (Word Rev.1, fuente Markdown, plantilla)
  imagenes/              Diagramas y capturas de HMI
  resolucion-tecnica/    Markdown de diagnóstico, procedimiento, chat e inventario TIA
  variables/             Listados y notas de tags PLC / Runtime
```

| Carpeta | Contenido |
|---|---|
| [.rules/](.rules/) | [Contexto del agente](.rules/AGENT-CONTEXT.md) · [prompt de sesión](.rules/AGENT-PROMPT.md) |
| [entregable/](conocimiento/entregable/) | **[Word Rev.1 — entregado al cliente](conocimiento/entregable/Informe-Tecnico-Comunicacion-OPC-UA-WinCC-RT%20(Rev.1).docx)** · [fuente Markdown](conocimiento/entregable/Informe-Tecnico-Comunicacion-OPC-UA-WinCC-RT.md) · [Word generado (pre-revisión)](conocimiento/entregable/Informe-Tecnico-Comunicacion-OPC-UA-WinCC-RT.docx) · [plantilla IndustrialLAB](conocimiento/entregable/IndustrialLAB-plantilla-informe.docx) |
| [imagenes/](conocimiento/imagenes/) | Arquitectura OPC UA WinCC–Ignition · listado de alarmas HMI |
| [resolucion-tecnica/](conocimiento/resolucion-tecnica/) | [Informe de implementación](conocimiento/resolucion-tecnica/OPC-UA-WinCC-RT-Professional-V18.md) · [diagnóstico PowerShell](conocimiento/resolucion-tecnica/OPC-PowerShell-Diagnostico.md) · [chat condensado](conocimiento/resolucion-tecnica/Chat-Resumen-OPC.md) · [chuleta Git](conocimiento/resolucion-tecnica/GitHub-Chuleta.md) · [software TIA V18](conocimiento/resolucion-tecnica/software-instalado.md) |
| [variables/](conocimiento/variables/) | [Informe de variables](conocimiento/variables/Informe-variables.md) · [tags centrífuga 1](conocimiento/variables/tabla-variables-centrifuga-1.md) · [listado completo Runtime](conocimiento/variables/listado-completo-variables.md) |

## Resultado de referencia

- **OPC DA:** ProgID `OPCServer.WinCC_SCADA.1` (OPC Expert)
- **OPC UA:** `opc.tcp://192.168.0.221:4861`, servicio `OpcUaServerWinCCPro` (estación de planta); en laboratorio, `opc.tcp://localhost:4861`

## Nota

Este repositorio es documentación. El push a GitHub lo hace el usuario de forma manual. Comandos: [`conocimiento/resolucion-tecnica/GitHub-Chuleta.md`](conocimiento/resolucion-tecnica/GitHub-Chuleta.md).
