# Contexto para un agente de IA — WinCC RT Professional OPC

Usá este archivo **junto con** `OPC-UA-WinCC-RT-Professional-V18.md` (informe técnico validado) y, si el usuario lo pega, el prompt de `AGENT-PROMPT.md`.

No redescubras el problema desde cero. Esta estación de referencia **ya funciona**. El trabajo típico siguiente es **replicar o diagnosticar otra PC Station** con el mismo patrón.

---

## 1. Quién es el usuario y cómo trabajar

- Habla **español**. Respuestas cortas, un paso siguiente, sin recitar todo el informe.
- No está en un repo de aplicación: es **ingeniería Siemens** (TIA / WinCC / PLC) en PC o VM Windows.
- Valora diagnóstico con **evidencia** (servicio, carpeta, `netstat`, log) antes de reinstalar.
- No pidas desinstalar Runtime si el HMI/MPI ya leen el PLC.
- No mezcles OPC DA y OPC UA. Preguntá cuál está fallando **o** diagnosticá ambos por separado.
- PowerShell: comandos **cortos y de ruta fija**. Evitá `Get-ChildItem -Recurse` sobre todo `Siemens` (se cuelga).
- En VM: el ISO se desconecta al reiniciar. Setup desde **carpeta en disco**.

---

## 2. Arquitectura ya validada (2026-08-18)

```
S7-315-2DP  --MPI (PC Adapter USB)-->  WinCC RT Professional (PC Station)
                                         |-- OPC DA  OPCServer.WinCC_SCADA / .1
                                         |-- OPC UA  opc.tcp://<host>:4861
                                                     OpcUaServerWinCCPro
```

- El cliente OPC lee **WinCC**, no el PLC.
- El 315-2DP **no tiene Ethernet**. No propongas Profinet al PLC como atajo.
- Equipo de referencia: hostname `DESKTOP-KC6QA5V`, usuario `desktop-kc6qa5v\gigabyte-pc`.
- TIA Portal **V18 Upd5**. Runtime Professional debe coincidir (mismo Update).
- Licencia que cuenta: **WinCC Runtime Professional**. Ignorar **OPC UA Server Process Historian**.

---

## 3. Hechos que no hay que volver a “confirmar” en abstracto

Estos puntos **ya están comprobados**. En otra estación, verificá si se cumplen; no los debates como hipótesis abiertas.

1. TIA **Runtime settings → OPC UA puerto 4861** no instala el servidor. Es Engineering.
2. El motor UA está en el ISO **SIMATIC WinCC Runtime Professional V18**, casilla **Tools → WinCC OPC UA Server**.
3. El ISO **TIA Portal V18 STEP 7 + WinCC Professional** no trae esa casilla (solo OPC XML Gateway).
4. SiePortal **109807122** = Unified PC RT. **No usar**. **109807225** = updates, no el ISO base.
5. Carpeta `WinCC\OPC\UAClient` + paquete `OPCUA_Client` = WinCC como **cliente**. El servidor es `UAServer` + `OpcUaServerWinCCPro.exe` (no `OpcUaServerWinCC.exe`).
6. `OPC.SimaticHMI.CoRtHmiRTm.1` = **Advanced**. Mensaje “process is not running” es esperable en una PC Station Professional.
7. `OPC.SimaticNET` no expone tags de WinCC y pelea el MPI.
8. OPC DA no se diagnostica con `netstat 4861`.
9. Error servicio `0x80004005` / evento **7023 Unspecified error** → mirar  
   `C:\Users\Public\Documents\Siemens\WinCC\OPC\UAServer\OpcUaServerWinCCPro.txt`  
   En la referencia: certificado PKI inválido tras setup interrumpido. Arreglo: mover `.der`/`.pfx` viejos y dejar recrear.
10. OPC Expert validó **DA** (`OPCServer.WinCC_SCADA.1`). Su cliente **UA** falló con “given credentials”. **UaExpert** sí funcionó tras **TRUST SERVER CERTIFICATE** (`BadCertificateUntrusted` es normal).
11. Optix/Ignition = camino **OPC UA** (`opc.tcp://IP:4861`), no el ProgID DA.

---

## 4. Árbol de decisión rápido (otra estación)

```
¿El HMI Professional está en Runtime y las tags MPI tienen valor?
  NO → primero comunicación PLC/MPI. No toques OPC.
  SÍ ↓

¿Existe ...\WinCC\OPC\UAServer\OpcUaServerWinCCPro.exe
   y el servicio OpcUaServerWinCCPro?
  NO → instalar componente desde ISO WinCC Runtime Professional
        (no TIA, no Unified, no solo Update). Verificar disco y medio local.
  SÍ ↓

¿Get-Service OpcUaServerWinCCPro = Running y netstat 4861 LISTENING?
  NO → Start-Service. Si 0x80004005: trace + log txt + regenerar PKI.
  SÍ ↓

¿El usuario quiere DA o UA?
  DA  → OPC Expert, ProgID OPCServer.WinCC_SCADA.1
        (NO CoRtHmiRTm)
  UA  → UaExpert, opc.tcp://localhost:4861, None, Trust certificate
```

---

## 5. Cómo diagnosticar (orden, no en paralelo caótico)

1. Tipo de HMI en TIA: **WinCC RT Professional** vs Advanced vs Unified vs Client.
2. Runtime abierto + calidad de tags.
3. ¿Hay `UAServer` y servicio? ¿Hay ProgID `OPCServer.WinCC_SCADA` en registro?
4. Si falta componente → medio **Runtime Professional**, Tools, casilla Server.
5. Si el componente está y el servicio muere → log Siemens, no ALM Process Historian, no DCOM remoto.
6. Si UA escucha y el cliente falla → UaExpert + Trust; no pelear horas con OPC Expert UA.
7. DCOM/firewall solo si el cliente es **otra PC** y DA/UA **ya funcionan en local**.

---

## 6. Comandos mínimos (copiar del informe si hace falta el bloque completo)

Secuencia cronológica completa (reseña + acción esperada por comando): **`OPC-PowerShell-Diagnostico.md`**.

Ruta base:

`C:\Program Files (x86)\Siemens\Automation\SCADA-RT_V11\WinCC`

```powershell
Get-ChildItem "...\WinCC\OPC"
Test-Path "...\WinCC\OPC\UAServer\OpcUaServerWinCCPro.exe"
Get-Service OpcUaServerWinCCPro
netstat -ano | findstr "4861"
Get-Content "C:\Users\Public\Documents\Siemens\WinCC\OPC\UAServer\OpcUaServerWinCCPro.txt" -Tail 40
```

Regenerar certificado: ver sección 11 de `OPC-PowerShell-Diagnostico.md` (o 5.7 del informe).

---

## 7. Estilo de respuesta esperado

- Empezar por el estado (qué está instalado / escuchando / fallando).
- Un solo siguiente paso accionable.
- Distinguir **cliente vs servidor**, **DA vs UA**, **TIA vs Runtime ISO**, **Advanced vs Professional**.
- No abrir hilos de OPC UA Server de paneles Comfort / RT Advanced / Unified salvo que el dispositivo TIA sea ese.
- No sugerir Connectivity Pack de WinCC V7 como primer requisito de RT Professional TIA (la licencia documentada es WinCC Runtime Professional).
- Actualizar el informe solo si el usuario lo pide y hay un hallazgo nuevo.

---

## 8. Archivos de esta carpeta (`conocimiento/`)

| Archivo | Para quién |
|---|---|
| `OPC-UA-WinCC-RT-Professional-V18.md` | Humano + agente: cronología, desvíos, comandos |
| `OPC-PowerShell-Diagnostico.md` | Humano + agente: comandos PS en orden cronológico con reseña |
| `GitHub-Chuleta.md` | Humano: clone / pull / commit / push de este repo |
| `AGENT-CONTEXT.md` | Este archivo: marco mental del agente |
| `AGENT-PROMPT.md` | Prompt listo para pegar en un chat nuevo |
| `Chat-Resumen-OPC.md` | Transcripción condensada del chat de resolución |
