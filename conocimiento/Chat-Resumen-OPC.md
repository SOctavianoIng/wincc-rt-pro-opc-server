# Hilo de chat editado — OPC DA/UA en WinCC Runtime Professional V18

**Tipo de documento:** transcripción condensada del chat de diagnóstico e implementación.  
**Fecha del trabajo:** 2026-08-18 (cierre documental 2026-08-21).  
**Estación de referencia:** VM Windows `DESKTOP-KC6QA5V`, usuario `desktop-kc6qa5v\gigabyte-pc`.  
**Proyecto TIA:** PC Station `HMI_RT_1` [WinCC RT Professional] ↔ PLC **S7-315-2DP** por **MPI** (PC Adapter USB / RS485).  
**Objetivo:** leer tags de WinCC desde un cliente OPC (OPC Expert, luego Optix/Ignition).  
**Resultado:** OPC DA (`OPCServer.WinCC_SCADA.1`) y OPC UA (`opc.tcp://localhost:4861`, servicio `OpcUaServerWinCCPro`) operativos; tags visibles en OPC Expert (DA) y UaExpert (UA).

Este archivo **mantiene el hilo** del diálogo (Usuario → Asistente), elimina idas y vueltas repetidas, y deja las consultas mejor redactadas junto con los comandos completos que resolvieron el caso.

Documentos relacionados en esta carpeta:

- `OPC-UA-WinCC-RT-Professional-V18.md` — informe técnico estructurado  
- `OPC-PowerShell-Diagnostico.md` — secuencia de comandos (incluye §12.1 sin Runtime)  
- `AGENT-CONTEXT.md` — marco para un agente de IA  
- `AGENT-PROMPT.md` — prompt listo para otra estación  

**Seguimiento:** §20 (2026-08-25) hilo corto; **§21 (2026-08-25/26)** tutorial — etapa definitiva PC WinCC RT V18 Server OPC UA + cliente remoto KEPServer.

---

## 0. Contexto previo (chat anterior, resumido)

Antes de este hilo se había explorado **OPC UA** de WinCC RT Professional. Se confirmó que **nada escuchaba en el puerto 4861**. En software instalado aparecía `WinCC Runtime Professional V18 - OPCUA_Client`, pero **no** un componente de servidor UA. Parte del chat previo mezcló conclusiones de “falta OPC UA Server” con la decisión posterior de usar **OPC DA nativo**. Ese contexto se reutilizó aquí, pero el pedido concreto de este chat fue **OPC DA** y, al final, también **OPC UA**.

---

## 1. Pedido inicial: diagnosticar OPC DA con OPC Expert

**Usuario:** Necesito leer tags desde WinCC Runtime Professional V18 (PC Station RT) que toma datos por MPI RS485 desde un PLC. Me decidí por el **OPC DA nativo** de WinCC Professional. Adjunto el historial de un chat previo (mucho de OPC UA / puerto 4861) y tengo una captura de error de OPC Expert. ¿Cómo seguimos el diagnóstico y los próximos pasos?

**Asistente:** El chat previo mezcló OPC UA con OPC DA. Puntos clave:

- WinCC RT Professional **sí** tiene servidor OPC DA nativo.
- OPC DA **no** usa los puertos UA 4861/4870; `netstat` sobre esos puertos no diagnostica DA.
- OPC DA va por COM/DCOM.
- ProgID documentado para RT Professional: `OPCServer.WinCC_SCADA` (también puede figurar `OPCServer.WinCC`).
- Licencia: WinCC Runtime Professional (no hace falta Process Historian para esto).

Prueba decisiva: OPC Expert **en la misma PC** que el Runtime, con el HMI abierto. Si falla, pedir ProgID exacto, código hex y si es local o remoto.

---

## 2. Primera evidencia en OPC Expert: servidor equivocado

**Usuario:** OPC Expert está en la misma máquina virtual que TIA Portal y el Runtime en ejecución.

- Descripción: `OPC.SimaticHMI.CoRtHmiRTm`
- ProgID: `OPC.SimaticHMI.CoRtHmiRTm.1`
- Host: `DESKTOP-KC6QA5V`
- Mensaje: *OPC Server is not using a port because process is not running* / *Process is not currently running*

**Asistente:** Ese ProgID es el OPC DA de **WinCC Runtime Advanced / Comfort**, no el de Professional. OPC Expert lo encuentra porque TIA registra la clase COM, pero el proceso no corre porque el Runtime activo es Professional. El texto de “no usa puerto” de OPC Expert **no aplica** a OPC DA de Professional.

Servidor correcto: **`OPCServer.WinCC_SCADA`** (o `OPCServer.WinCC`). No usar `CoRtHmiRTm`.

---

## 3. Los ProgID de Professional no aparecen en la lista

**Usuario:** En OPC Expert veo `OPC.Siemens.XML.1`, `OPC.SimaticNET.1`, `OPC.SimaticNET.DP.1` y varios UA. **No** están `OPCServer.WinCC_SCADA` ni `OPCServer.WinCC`. ¿Agrego a mano el ProgID?

**Asistente:** OPC Expert lista clases COM **registradas en Windows**, no “servidores del proyecto TIA”. Si no aparece, no está registrado. Agregar a mano sirve solo para confirmar *Class not registered*. Mejor verificar el registro:

```powershell
Get-Item HKLM:\SOFTWARE\Classes\OPCServer.WinCC_SCADA -ErrorAction SilentlyContinue
Get-Item HKLM:\SOFTWARE\Classes\OPCServer.WinCC -ErrorAction SilentlyContinue
Get-Item HKLM:\SOFTWARE\WOW6432Node\Classes\OPCServer.WinCC_SCADA -ErrorAction SilentlyContinue
Get-Item HKLM:\SOFTWARE\WOW6432Node\Classes\OPCServer.WinCC -ErrorAction SilentlyContinue
```

No usar:

| Servidor | Qué es |
|---|---|
| `OPC.SimaticHMI.CoRtHmiRTm.1` | DA de Runtime Advanced |
| `OPC.Siemens.XML.1` | XML-DA |
| `OPC.SimaticNET.1` / `.DP.1` | SIMATIC NET (PLC, no WinCC) |

---

## 4. Registro vacío: el DA de Professional no está

**Usuario:** Las cuatro consultas al registro no devolvieron nada (PowerShell sin salida de objeto).

**Asistente:** Diagnóstico cerrado para DA: **el servidor OPC DA de WinCC Runtime Professional no está registrado** en esa VM. Hay que agregar el componente con el instalador de **WinCC Runtime Professional**, no pelear DCOM ni UA todavía.

---

## 5. Comprobar “si el servicio está corriendo” sin reinstalar

**Usuario:** Es mucho trabajo reinstalar. Estoy casi seguro de que la instalación fue completa. ¿Otro método para comprobar que el servicio esté corriendo? Voy a probar con Runtime en ejecución (antes corrí comandos sin Runtime).

**Asistente:** OPC DA de Professional **no es un servicio TCP fijo**. Si las claves de registro están vacías, el componente no está, con Runtime parado o arriba. Con Runtime abierto, verificar procesos, ProgIDs y binarios:

```powershell
Write-Host "=== Procesos OPC/WinCC ==="
Get-Process | Where-Object { $_.ProcessName -match 'opc|wincc|CCPtm|pdlrt' } |
  Select-Object ProcessName, Id, Path

Write-Host "`n=== ProgIDs OPC registrados ==="
Get-ChildItem HKLM:\SOFTWARE\Classes, HKLM:\SOFTWARE\WOW6432Node\Classes -ErrorAction SilentlyContinue |
  Where-Object { $_.PSChildName -match 'OPCServer|WinCC_SCADA|WinCC\.OPC|SimaticHMI' } |
  Select-Object -ExpandProperty PSChildName

Write-Host "`n=== Ejecutable en disco ==="
Get-ChildItem "C:\Program Files*","C:\Program Files (x86)\Siemens" -Recurse -Filter "OPCServer*.exe" -ErrorAction SilentlyContinue |
  Select-Object FullName
```

*(Nota: el recurse amplio sobre Siemens es lento; más adelante se pasó a rutas fijas.)*

---

## 6. Runtime abierto: solo hay Advanced / SimaticHMI, no Professional DA

**Usuario:** WinCC RT en ejecución leyendo variables del PLC. La búsqueda de ProgIDs muestra muchos `SimaticHMI.*` y `OPC.SimaticHMI.CoRtHmiRTm`, pero **ningún** `OPCServer.WinCC_SCADA`. La búsqueda de `OPCServer*.exe` no devolvió nada.

**Asistente:** Con Runtime leyendo el PLC, queda claro: **no está el OPC DA de Professional**. Solo el de Advanced. Pedir confirmación del tipo de dispositivo en TIA: **WinCC RT Professional** vs **Advanced**.

**Usuario:** Es un PC Station Profesional.

**Asistente:** Entonces el Runtime/MPI están bien; falta el servidor OPC. Siguiente paso: modificar instalación de **SIMATIC WinCC Runtime Professional V18** y marcar componentes OPC Server.

---

## 7. El setup no muestra “OPC Server” — inventario de software

**Usuario:** Ya intenté Modify y no hay casillero “OPC Server”. La instalación es completa. Paso listado de software (Windows Uninstall + TIA Components) y archivos `*opc*` bajo Siemens.

Hallazgos relevantes del listado:

- `SIMATIC WinCC Runtime Professional V18.0 Upd5` / `SIMATIC WinCC Runtime V18.0`
- `WinCC Runtime Professional V18 - OPCUA_Client`
- `SIMATIC OPC-XML-Gateway`, `SIEMENS OPC`, SIMATIC NET
- Runtime Advanced V17 también presente
- En disco: `OPCTags.exe`, canales `OPC.chn` / `OPC UA WinCC Channel.chn`, `SOPC_CoRtHmiRTm.exe` (Advanced), `SIMATIC.NET\...\opcdaserver.exe` — **no** un `OPCServer.exe` de WinCC SCADA

**Asistente:** El listado confirma Runtime Professional + **cliente** UA. Pedir listado de `WinCC\OPC` y de `bin` filtrado OPC, y si en TIA Runtime settings hay sección OPC.

```powershell
Get-ChildItem "C:\Program Files (x86)\Siemens\Automation\SCADA-RT_V11\WinCC\OPC" -Recurse |
  Select-Object FullName

Get-ChildItem "C:\Program Files (x86)\Siemens\Automation\SCADA-RT_V11\WinCC\bin" -Filter "*OPC*"
```

---

## 8. Solo cliente en bin; TIA sí muestra OPC UA

**Usuario:** En `WinCC\bin` aparecen `OPC.chn`, `OPC UA WinCC Channel.chn`, `OPCTags.exe`, `OpcUaServerWinCC.xml`. En TIA Runtime settings **sí** hay configuración OPC UA.

**Asistente:** Eso cierra OPC DA “nativo faltante” de otra forma: en `bin` están canales **cliente**, no el servidor DA. La pantalla TIA OPC UA es Engineering. Preferible ir a **OPC UA** (mejor para Optix/Ignition), pero hay que verificar si el motor escucha en 4861.

---

## 9. Captura TIA: puerto 4861 configurado — pero Windows no escucha

**Usuario:** Prefiero el servidor OPC UA de TIA Portal. Adjunto captura: Runtime settings → OPC settings, puerto **4861**, URL `opc.tcp://[HostName]:4861`, políticas None/Basic128/Basic256/Basic256Sha256.

Luego, con Runtime arriba:

```text
netstat -ano | findstr "4861"     → vacío
Get-Process ... OpcUa|CCOpcUa|WinCC → solo opcualds (Local Discovery Server)
Get-Service ... OpcUa|WinCC → CCOpcUaImporter Stopped; SQL WINCC Running
```

**Asistente:** TIA está configurado; el servidor UA **no está corriendo**. `opcualds` no es WinCC. Pedir búsqueda de `OpcUaServer*` y confirmar que el HMI es Professional (no Client).

**Usuario:** Solo existe  
`...\SCADA-RT_V11\WinCC\bin\OpcUaServerWinCC.xml`  
El proyecto es PC Station Professional, Runtime en ejecución, MPI OK.

**Asistente:** Solo hay un XML de plantilla. Falta `OpcUaServerWinCC.exe` / carpeta `UAServer`. Repair o reinstalar Runtime Professional con el medio correcto. La licencia Process Historian OPC UA **no** habilita este servidor.

---

## 10. ISO equivocado (TIA) vs Runtime Professional

**Usuario:** No hay Repair en Programas y características. Empiezo a desinstalar/reinstalar Update 5. Luego muestra setup de **TIA Portal V18 STEP 7 Prof. + WinCC Professional**: en Overview solo **OPC XML Gateway**, no OPC UA Server. Aviso de componentes en versión más alta.

**Asistente:** Ese asistente es **Engineering**, no Runtime Professional. **Cancelar.** El servidor UA sale del ISO/DVD **SIMATIC WinCC Runtime Professional V18**.

**Usuario:** WinCC Runtime Professional V18 completo no lo tengo; solo updates. ¿El Unified PC RT (109807122) es ese?

**Asistente:** **No.** Unified es otro producto. En SiePortal, **109807225** suelen ser updates, no el ISO base. El base viene con licencia **6AV2105-…** / DVD comercial. No desinstalar el Runtime que ya lee el PLC.

**Usuario:** No continué la desinstalación. Solo el Update 5. Luego confirma que sigue instalado `SIMATIC WinCC Runtime V18.0` (`18.00.0000`).

---

## 11. Se consigue el instalador correcto — casilla OPC UA Server

**Usuario:** Conseguí un instalador de WinCC Runtime Professional V18. En el setup, bajo Tools, aparecen:

- WinCC OPC DA Server  
- WinCC OPC HDA Server  
- WinCC OPC A&E Server  
- WinCC OPC-XML DA Server  
- **WinCC OPC UA Server** (desmarcado en la captura)  

Disco C: en rojo (12 GB libres vs 14.5 GB necesarios).

**Asistente:** Marcar **WinCC OPC UA Server** (y DA si se quiere). **No continuar** con disco en rojo. Desmarcar reinstalar SQL si `MSSQL$WINCC` ya existe. Limpiar temporales:

```powershell
Remove-Item "$env:TEMP\*" -Recurse -Force -ErrorAction SilentlyContinue
```

También: `%TEMP%`, `C:\Windows\Temp`, `cleanmgr`. No borrar Siemens ni SQL WINCC.

---

## 12. Instalación interrumpida por ISO de VM y BSOD

**Usuario (resumen de varios turnos):** Setup “OK” + reinicio, pero después:

```powershell
Test-Path "...\UAServer\OpcUaServerWinCC.exe"   # False
Get-ChildItem "...\WinCC\OPC"                   # solo UAClient
Get-Service *OpcUa*                             # vacío
```

Se descubre que el **disco de imagen se desconectó** al reiniciar la VM; la instalación no había terminado. Al continuar hay varios reinicios (normales) y una **pantalla azul**.

**Asistente:**

- Nombre correcto del exe más adelante: **`OpcUaServerWinCCPro.exe`**, no `OpcUaServerWinCC.exe`.
- Reinicios de Siemens son normales si el asistente retoma; BSOD no lo es.
- Mejor copiar el ISO a `C:\Temp\WinCC_RT_V18\` y ejecutar desde disco.
- Después del Finish real: verificar `UAServer`, servicio y Update 5 alineado con TIA.

---

## 13. Componente instalado: aparece UAServer y el servicio

**Usuario:** Tras completar la instalación:

```text
WinCC\OPC contiene: AlarmEvent, DataAccess, HistDataAccess, SSC, UAClient, UAServer, WinCC-OPC-XML, XMLDataAccess
Test-Path OpcUaServerWinCC.exe → False
Get-Service *OpcUa* → (sin match con ese filtro al inicio)
```

Luego:

```text
Get-Service *WinCC*
  Running  MSSQL$WINCC
  Stopped  OPCServer.WinCC_SCADA
  Stopped  OPCServerAE.WinCC_SCADA
  Stopped  OPCServerHDA.WinCC_SCADA
  Paused/Stopped  OpcUaServerWinCCPro
```

Y el listado de UAServer incluye `OpcUaServerWinCCPro.exe`, XML, DLLs y PKI.

**Asistente:** El servidor UA **sí quedó instalado**. El `Test-Path` fallaba por el nombre viejo. Instalar **Update 5** del Runtime para igualar TIA. Con Runtime abierto, el servicio debería pasar a Running; si no, arrancarlo y mirar 4861.

```powershell
Get-Service OpcUaServerWinCCPro, OPCServer.WinCC_SCADA
netstat -ano | findstr "4861"
```

---

## 14. Servicio no arranca: 0x80004005 / Unspecified error

**Usuario:** Runtime leyendo PLC. PowerShell admin:

```text
Start-Service OpcUaServerWinCCPro → Failed
Get-Service → Stopped
netstat 4861 → vacío
```

```text
sc.exe start OpcUaServerWinCCPro → START_PENDING, PID 9508, luego el proceso desaparece
sc.exe qc → BINARY_PATH ...\OpcUaServerWinCCPro.exe, AUTO_START, LocalSystem, dep RPCSS
Evento System 7023: terminated with Unspecified error
```

Licencias ALM OK (WinCC Runtime Professional). Process Historian no es la causa.

XML del servidor: puerto 4861, None habilitado, `ForceCertificateCreation=true`. Ejecutar el `.exe` a mano vuelve al prompt sin mensaje (es un servicio).

**Asistente:** Activar trace y leer el log Siemens. También se exploró Log On con la cuenta de Windows (`whoami` → `desktop-kc6qa5v\gigabyte-pc`); Windows no permite servicio con cuenta local **sin contraseña** (hubo que crear una). Aun así, 0x80004005 persistió hasta el log de certificado.

```powershell
$xml = "C:\Program Files (x86)\Siemens\Automation\SCADA-RT_V11\WinCC\OPC\UAServer\OpcUaServerWinCCPro.xml"
(Get-Content $xml -Raw).Replace('<TraceEnable>false</TraceEnable>','<TraceEnable>true</TraceEnable>') |
  Set-Content $xml -Encoding UTF8

Start-Service OpcUaServerWinCCPro -ErrorAction SilentlyContinue
Start-Sleep 4
Get-Content "C:\Users\Public\Documents\Siemens\WinCC\OPC\UAServer\OpcUaServerWinCCPro.txt" -Tail 50
Get-ChildItem "...\PKI\CA\certs"
Get-ChildItem "...\PKI\CA\private"
```

**Log decisivo:**

```text
Couldn't access certificate store.
Please install a valid certificate and restart the server.
```

Había `.der` y `.pfx` de las 19:14 (instalación a medias / BSOD): certificado inválido.

---

## 15. Resolución del servicio UA: regenerar certificado PKI

**Usuario:** Tras permisos `icacls` el error de certificado seguía. Luego se movieron los certs viejos y se reinició el servicio.

Comando completo (el que levantó el servidor):

```powershell
$ua = "C:\Program Files (x86)\Siemens\Automation\SCADA-RT_V11\WinCC\OPC\UAServer"
$pki = "$ua\PKI"

try { [xml](Get-Content "$ua\OpcUaServerWinCCPro.xml" -Raw); "XML OK" } catch { "XML ROTO: $_" }

New-Item "$pki\CA\rejected" -ItemType Directory -Force | Out-Null
icacls $ua /grant "desktop-kc6qa5v\gigabyte-pc:(OI)(CI)F" /T

New-Item "$pki\_bak" -ItemType Directory -Force | Out-Null
Move-Item "$pki\CA\certs\*.der" "$pki\_bak\" -Force -ErrorAction SilentlyContinue
Move-Item "$pki\CA\private\*.pfx" "$pki\_bak\" -Force -ErrorAction SilentlyContinue

Start-Service OpcUaServerWinCCPro
Start-Sleep 4
Get-Service OpcUaServerWinCCPro
Get-ChildItem "$pki\CA\certs","$pki\CA\private" | Select-Object FullName, Length
Get-Content "C:\Users\Public\Documents\Siemens\WinCC\OPC\UAServer\OpcUaServerWinCCPro.txt" -Tail 15
netstat -ano | findstr "4861"
```

**Resultado:**

```text
Status: Running  OpcUaServerWinCCPro
Log: No own certificate found. Creating a new application instance certificate.
TCP 0.0.0.0:4861 LISTENING
TCP [::]:4861 LISTENING
```

**Asistente:** Había **dos problemas**:

1. Faltaba el motor OPC (casilla del ISO Runtime Professional; TIA no lo instala).  
2. El certificado PKI quedó roto tras ISO desconectado / BSOD; regenerarlo levantó el servicio.

No era DCOM, ni firewall local, ni la licencia Process Historian.

---

## 16. Clientes: DA funciona; UA con OPC Expert falla; UaExpert OK

**Usuario:** OPC Expert funciona perfecto — pero con ProgID **`OPCServer.WinCC_SCADA.1`**.

**Asistente:** Eso es **OPC DA**, no UA. El `.1` es la versión COM del ProgID. Para Optix/Ignition probar UA:

- URL: `opc.tcp://localhost:4861`
- Endpoint: **None**
- Servicio debe seguir Running

**Usuario:** Servicio Running y 4861 LISTENING. En OPC Expert elige endpoint None + Anonymous / luego usuario Windows. Resultado del test:

```text
Retrieved UA Server information for opc.tcp://localhost:4861
Retrieved certificate ...
Created authentication token ...
Saved test credentials ...
Failed to connect ... using given credentials.
```

`PKI\CA\rejected` vacío; el log UA no registró el intento del cliente.

**Asistente:** El servidor responde; el fallo parece del stack UA de OPC Expert / credenciales guardadas. Probar **UaExpert**. Grupo `SIMATIC HMI` opcional. Clear de credenciales en OPC Expert.

**Usuario:** UaExpert muestra `BadCertificateUntrusted` (cert autofirmado Siemens OPC UA Server for WinCC Runtime Professional).

**Asistente:** Pulsar **TRUST SERVER CERTIFICATE** (no es el mismo error que *Couldn't access certificate store* del servicio).

**Usuario:** Tras Trust, **veo las variables**. Funciona perfecto.

---

## 17. Documentación y materiales para otra estación / agente

**Usuario:** Actualizar el informe; crear contexto y prompt para un futuro agente; luego crear repo `wincc-rt-pro-opc-server` con carpeta `conocimiento` y un markdown del chat completo sin consultas duplicadas.

**Asistente:** Entregables en `conocimiento/`:

| Archivo | Contenido |
|---|---|
| `OPC-UA-WinCC-RT-Professional-V18.md` | Informe: cronología, desvíos, comandos, procedimiento limpio |
| `AGENT-CONTEXT.md` | Marco mental del agente |
| `AGENT-PROMPT.md` | Prompt para chat nuevo en otra PC |
| `Chat-Resumen-OPC.md` | Este hilo condensado |

---

## 18. Conceptos que hay que conservar (glosario del hilo)

| Concepto | Definición operativa |
|---|---|
| OPC DA | COM/DCOM. ProgID `OPCServer.WinCC_SCADA` / `.1`. No usa puerto 4861. |
| OPC UA | `opc.tcp://host:4861`. Servicio `OpcUaServerWinCCPro`. |
| Engineering (TIA) | Configura puerto/seguridad OPC en el proyecto; no copia el motor. |
| Runtime Professional ISO | Medio con Tools → WinCC OPC UA/DA Server. |
| `UAClient` / `OPCUA_Client` | WinCC como cliente. |
| `UAServer` / `OpcUaServerWinCCPro.exe` | Motor servidor UA. |
| `CoRtHmiRTm` | Servidor DA de Advanced. |
| `opcualds` | Local Discovery Server OPC Foundation, no WinCC. |
| 0x80004005 / 7023 | Servicio UA muere; mirar log Siemens. |
| Couldn't access certificate store | PKI del servidor inválido; regenerar `.der`/`.pfx`. |
| BadCertificateUntrusted | Cliente aún no confía el cert autofirmado; Trust en UaExpert. |
| Process Historian OPC UA license | No habilita este servidor. |
| Unified PC RT | Otro producto; no usar para esta PC Station. |

---

## 19. Checklist mínimo para la próxima estación

1. HMI = WinCC RT Professional; Runtime lee el PLC.  
2. ISO **WinCC Runtime Professional** (no TIA, no Unified), setup desde carpeta local.  
3. Marcar **WinCC OPC UA Server** (+ DA si hace falta). Disco con holgura.  
4. Mismo Update que TIA.  
5. Verificar `UAServer` + `Get-Service OpcUaServerWinCCPro`.  
6. Si 0x80004005 → trace + regenerar PKI (sección 15).  
7. UaExpert → `opc.tcp://localhost:4861` → None → Trust → tags.  
8. OPC Expert DA → `OPCServer.WinCC_SCADA.1` si se necesita DA.

---

## 20. Seguimiento 2026-08-25 — otra estación: servers ya instalados

**Contexto:** Runtime aún no abierto al inicio; PLC desconectado. Objetivo: saber si OPC UA/DA de WinCC están vivos e identificar ProgIDs. Documentación usada: `OPC-PowerShell-Diagnostico.md` (se agregó §12.1 atajo sin Runtime).

### 20.1 Comando “¿están corriendo?” sin Runtime

**Usuario:** ¿Cuál es el comando para saber si el OPC Server UA y DA de WinCC están corriendo? El Runtime todavía no está en ejecución.

**Asistente:** Sin Runtime, chequear servicios + puerto UA (no `Get-Process` de HMI):

```powershell
Get-Service OpcUaServerWinCCPro, OPCServer.WinCC_SCADA -ErrorAction SilentlyContinue
netstat -ano | findstr "4861"
```

Complemento instalación (aunque estén Stopped):

```powershell
Get-Service OpcUaServerWinCCPro, OPCServer.WinCC_SCADA -ErrorAction SilentlyContinue
Test-Path "C:\Program Files (x86)\Siemens\Automation\SCADA-RT_V11\WinCC\OPC\UAServer\OpcUaServerWinCCPro.exe"
Get-Item HKLM:\SOFTWARE\Classes\OPCServer.WinCC_SCADA -ErrorAction SilentlyContinue
```

DA suele quedar Stopped hasta que el Runtime lo use; `netstat 4861` no diagnostica DA.

### 20.2 Hueco en la base de conocimiento

**Usuario:** ¿Por qué esos comandos no están en la base de conocimientos?

**Asistente:** Sí estaban, pero **repartidos** (§2.1, §5.3, §6.1–6.2, §12). Faltaba un atajo único “Runtime apagado”.

**Usuario:** Agregalo.

**Asistente:** En `OPC-PowerShell-Diagnostico.md` → **§12.1 Sin Runtime** (+ §12.2 smoke UA con Runtime) y mapa rápido actualizado.

### 20.3 Evidencia: componentes OK, UA en Paused

**Usuario:** Salida:

```text
OPCServer.WinCC_SCADA   (ProgID en Classes)
True                    (Test-Path OpcUaServerWinCCPro.exe)
Stopped  OPCServer.WinCC_SCADA
Paused   OpcUaServerWinCCPro
```

**Asistente:** Instalación presente. DA Stopped = normal sin Runtime. UA **Paused** = no operativo aún. Arrancar:

```powershell
Start-Service OpcUaServerWinCCPro
Start-Sleep 4
Get-Service OpcUaServerWinCCPro
netstat -ano | findstr "4861"
```

### 20.4 ¿Probar con Runtime y PLC desconectado?

**Usuario:** Aunque no levante variables del PLC (desconectado), ¿puedo probar esto con Runtime en ejecución?

**Asistente:** Sí. El Runtime abierto basta para probar el **servidor OPC**. Tags ligados al PLC pueden verse con mala calidad; la conexión ProgID/endpoint es la evidencia. No hace falta MPI/PLC vivos para validar el motor.

### 20.5 UA Running + 4861 LISTENING

**Usuario:**

```text
TCP    0.0.0.0:4861    ...    LISTENING    4452
TCP    [::]:4861       ...    LISTENING    4452
Running  OpcUaServerWinCCPro
```

**Asistente:** UA OK. Siguiente: UaExpert → `opc.tcp://localhost:4861` → None → TRUST SERVER CERTIFICATE. DA: OPC Expert → `OPCServer.WinCC_SCADA.1` con Runtime abierto.

### 20.6 Por qué ahora fue más simple

**Usuario:** Bastante más simple que en la etapa de prueba. ¿Puede ser porque se rompió la instalación en esa instancia y no levantaban los Server OPC?

**Asistente:** Sí. En la referencia el setup del ISO Runtime quedó a medias (ISO desconectado al reboot de la VM) → solo `UAClient`, sin `UAServer`/servicio. Después, con motor instalado, el segundo tropiezo fue PKI corrupto (0x80004005). Aquí ProgID + exe + Running/4861 = estado post-instalación OK.

### 20.7 Listar ProgIDs / nombres de servers

**Usuario:** ¿Cómo sé los ProgID y nombres de los servers OPC de WinCC?

**Asistente:**

```powershell
Get-ChildItem HKLM:\SOFTWARE\Classes, HKLM:\SOFTWARE\WOW6432Node\Classes -ErrorAction SilentlyContinue |
  Where-Object { $_.PSChildName -match 'OPCServer|WinCC_SCADA|WinCC\.OPC|SimaticHMI' } |
  Select-Object -ExpandProperty PSChildName

Get-Service *OpcUa*, *WinCC*, OPCServer* -ErrorAction SilentlyContinue |
  Select-Object Name, DisplayName, Status
```

**Usuario:** Error de pipeline vacío / comandos en orden invertido al pegar.

**Asistente:** El pipeline debe ir `Get-ChildItem | Where-Object | Select-Object`. Preferir una sola línea si la consola invierte el pegado.

**Usuario:** Salida incluye `OPC.SimaticHMI.CoRtHmiRTm(.1)`, `OPCServer.WinCC_SCADA(.1)`, `OPCServerAE.WinCC_SCADA(.1)`, `OPCServerHDA.WinCC_SCADA(.1)` y muchos `SimaticHMI.CoRt…` (controles).

**Asistente:** Para tags Professional usar solo:

| Canal | Identificador |
|---|---|
| OPC DA | `OPCServer.WinCC_SCADA.1` |
| OPC A&E | `OPCServerAE.WinCC_SCADA.1` |
| OPC HDA | `OPCServerHDA.WinCC_SCADA.1` |
| OPC UA | `opc.tcp://localhost:4861` / servicio `OpcUaServerWinCCPro` (no ProgID) |

No usar `CoRtHmiRTm` (Advanced). El resto de `SimaticHMI.CoRt…` no son servers OPC de tags.

---

## 21. Etapa definitiva — PC WinCC Runtime Professional V18: Server OPC UA operativo + cliente remoto

**Tipo:** tutorial guiado (estación definitiva, 2026-08-25 / 2026-08-26).  
**Estación WinCC (ejemplo validado):** `DESKTOP-BH0RBSV`, usuario `desktop-bh0rbsv\siemens`, IP `192.168.0.221`.  
**Cliente remoto (ejemplo):** PC KEPServer `192.168.0.220`, misma red.  
**Resultado:** OPC UA local (UaExpert + Trust), OPC DA (`OPCServer.WinCC_SCADA.1`), y tags MPI vivos vía KEPServer OPC UA Client → Quick Client.

```text
PLC (MPI) → WinCC RT Professional V18 (PC Station)
              ├── OPC DA   ProgID OPCServer.WinCC_SCADA.1
              └── OPC UA   opc.tcp://<IP-WinCC>:4861
                           servicio OpcUaServerWinCCPro
                                    ↓
                         KEPServer (otra PC) → Quick Client
```

Sustituir `<IP-WinCC>` por la IPv4 real (sin símbolos `<>`). En el ejemplo: `192.168.0.221`.

Comandos detallados también en `OPC-PowerShell-Diagnostico.md` (§12.1 sin Runtime, §12.2 con Runtime).

---

### 21.1 Objetivo de esta etapa

Con el **componente servidor ya instalado** (ISO Runtime Professional → Tools → WinCC OPC UA/DA Server), dejar la PC WinCC como **servidor OPC UA definitivo** y validar:

1. Motor UA Running + puerto 4861.  
2. Cliente local UaExpert (Trust).  
3. Cliente DA local (opcional).  
4. Cliente remoto KEPServer en otra PC de la red.

> En la estación de prueba (2026-08-18) el setup del ISO quedó a medias y luego falló el PKI. Aquí el componente ya estaba: el trabajo es **arranque, enlace al Runtime, clientes y red**.

---

### 21.2 Paso 1 — ¿Está instalado el servidor? (Runtime puede estar apagado)

PowerShell (preferible Administrador):

```powershell
Get-Service OpcUaServerWinCCPro, OPCServer.WinCC_SCADA -ErrorAction SilentlyContinue
netstat -ano | findstr "4861"
Test-Path "C:\Program Files (x86)\Siemens\Automation\SCADA-RT_V11\WinCC\OPC\UAServer\OpcUaServerWinCCPro.exe"
Get-Item HKLM:\SOFTWARE\Classes\OPCServer.WinCC_SCADA -ErrorAction SilentlyContinue
Get-ChildItem "C:\Program Files (x86)\Siemens\Automation\SCADA-RT_V11\WinCC\OPC"
```

**Esperado (instalación OK):**

| Evidencia | Lectura |
|---|---|
| `Test-Path` = `True` | Exe UA presente |
| Carpetas `UAServer`, `DataAccess`, … | Servers instalados (no solo `UAClient`) |
| ProgID `OPCServer.WinCC_SCADA` | DA Professional registrado |
| `OpcUaServerWinCCPro` existe (Paused/Stopped/Running) | Servicio UA registrado |
| `OPCServer.WinCC_SCADA` Stopped | Normal sin Runtime |

Si falta `UAServer` o `Test-Path` es `False` → instalar desde ISO **WinCC Runtime Professional** (no ISO TIA). Ver secciones anteriores de este documento / informe.

---

### 21.3 Paso 2 — Identidad y ProgIDs útiles

```powershell
whoami
hostname
ipconfig

Get-ChildItem HKLM:\SOFTWARE\Classes, HKLM:\SOFTWARE\WOW6432Node\Classes -ErrorAction SilentlyContinue |
  Where-Object { $_.PSChildName -match 'OPCServer|WinCC_SCADA|WinCC\.OPC|SimaticHMI' } |
  Select-Object -ExpandProperty PSChildName
```

**Usar en clientes:**

| Canal | Identificador |
|---|---|
| OPC DA (tags) | `OPCServer.WinCC_SCADA.1` |
| OPC A&E | `OPCServerAE.WinCC_SCADA.1` |
| OPC HDA | `OPCServerHDA.WinCC_SCADA.1` |
| OPC UA | `opc.tcp://localhost:4861` (local) o `opc.tcp://<IP-WinCC>:4861` (remoto) |

**No usar:** `OPC.SimaticHMI.CoRtHmiRTm` / `.1` (Advanced).

> Pegar el pipeline en **este orden** (`Get-ChildItem | Where-Object | Select-Object`). Preferir una sola línea si la consola invierte el pegado multilínea.

---

### 21.4 Paso 3 — Runtime abierto + arranque del servicio UA

1. Abrir el **HMI Runtime Professional** con el proyecto cargado (pantallas/tags).  
2. Con PLC conectado se validan valores vivos; **sin PLC** igual se puede validar el motor OPC (tags con mala calidad).  
3. Arrancar / verificar UA:

```powershell
Start-Service OpcUaServerWinCCPro
Start-Sleep 4
Get-Service OpcUaServerWinCCPro
netstat -ano | findstr "4861"
```

**Esperado:**

```text
Running  OpcUaServerWinCCPro
TCP    0.0.0.0:4861    ...    LISTENING
TCP    [::]:4861       ...    LISTENING
```

`0.0.0.0:4861` indica que acepta conexiones de red (no solo localhost).

#### Desvío A — Servicio Running pero el log dice “no runtime project”

```powershell
Get-Content "C:\Users\Public\Documents\Siemens\WinCC\OPC\UAServer\OpcUaServerWinCCPro.txt" -Tail 30
```

Mensaje típico:

```text
ERROR: CRTIBase::GetWinCCProjectState: no runtime project was found, hr=0x80004005
```

**Solución:** el puerto puede estar LISTENING y aun así el UA **no ve** el proyecto Runtime. Con el HMI **ya cargado y estable**:

```powershell
Restart-Service OpcUaServerWinCCPro
Start-Sleep 5
Get-Content "C:\Users\Public\Documents\Siemens\WinCC\OPC\UAServer\OpcUaServerWinCCPro.txt" -Tail 15
```

El error periódico debe desaparecer. Recién entonces los clientes obtienen endpoints / tags.

**Orden correcto de arranque:** Runtime primero → luego (re)arranque de `OpcUaServerWinCCPro`.

#### Desvío B — UA en Paused / Stopped

```powershell
Start-Service OpcUaServerWinCCPro
Start-Sleep 4
Get-Service OpcUaServerWinCCPro
netstat -ano | findstr "4861"
```

Si vuelve a morir con 0x80004005 / evento 7023 → trace + regenerar PKI (secciones 10–15 de este documento / `OPC-PowerShell-Diagnostico.md`). En la estación definitiva de 2026-08-26 **no** fue necesario: bastó Runtime + Start/Restart del servicio.

---

### 21.5 Paso 4 — Cliente local UaExpert (OPC UA)

1. UaExpert en la **misma PC** que WinCC.  
2. No esperar discovery automático en `opc.tcp://localhost:4840` (Local Discovery Server). WinCC escucha en **4861**.  
3. En el diálogo **Add Server**:
   - Expandir **Custom Discovery**.  
   - **Doble clic** en `Double click to Add Server...`.  
   - URL **sin** `@`:

```text
opc.tcp://localhost:4861
```

   - Discover / Get Endpoints.  
   - Seleccionar el servidor Siemens y el endpoint **None**.  
   - Authentication: **Anonymous**.  
   - OK (se habilita solo con un endpoint seleccionado).  
4. Connect. Ante `BadCertificateUntrusted` → **TRUST SERVER CERTIFICATE**.  
5. Browse: tags visibles (con PLC, valores animados / calidad buena).

El prefijo `@` delante del nombre en el árbol Project es **normal** (entrada custom); no se escribe a mano en Configuration Name.

#### Desvíos UaExpert (soluciones)

| Síntoma | Solución |
|---|---|
| Log: FindServers en `:4840` BadTimeout | Ignorar LDS; agregar custom en **4861** |
| OK del Add Server en gris | Completar Custom Discovery → elegir **endpoint** None |
| URL solo en “Configuration Name” | No alcanza; hay que Discover + seleccionar endpoint |
| Servidor no aparece / no endpoints | Ver Desvío A (`no runtime project`) + `Restart-Service` con Runtime cargado |
| BadCertificateUntrusted | Trust del cert del servidor (autofirmado) |

---

### 21.6 Paso 5 — Cliente local OPC Expert (OPC DA)

Con Runtime abierto:

1. OPC Expert → conectar a **`OPCServer.WinCC_SCADA.1`**.  
2. No usar `CoRtHmiRTm`.  
3. Browse de tags.

OPC DA **no** se diagnostica con `netstat 4861`.

---

### 21.7 Paso 6 — Cliente remoto KEPServer (otra PC, misma red)

Arquitectura de prueba validada:

| Rol | IP ejemplo |
|---|---|
| WinCC (servidor UA) | `192.168.0.221` |
| KEPServer (cliente) | `192.168.0.220` |

#### 6.1 En PC WinCC — datos y estado

```powershell
ipconfig
Get-Service OpcUaServerWinCCPro
netstat -ano | findstr "4861"
```

URL para el cliente remoto:

```text
opc.tcp://192.168.0.221:4861
```

(Reemplazar por la IP actual de la PC WinCC.)

#### 6.2 Firewall — solo si hace falta

No es obligatorio “sí o sí”. Primero, **desde la PC de KEP**:

```powershell
Test-NetConnection 192.168.0.221 -Port 4861
```

- `TcpTestSucceeded : True` → **no** hace falta abrir firewall.  
- `False` (con UA Running en WinCC) → en la PC WinCC permitir TCP 4861:

```powershell
New-NetFirewallRule -DisplayName "WinCC OPC UA 4861" -Direction Inbound -Protocol TCP -LocalPort 4861 -Action Allow
```

En la prueba definitiva: `TcpTestSucceeded : True` sin regla nueva.

#### 6.3 En PC KEPServer — canal OPC UA Client

1. KEPServerEX Configuration → **New Channel** → driver **OPC UA Client**.  
2. **New Device** → Endpoint:

```text
opc.tcp://192.168.0.221:4861
```

3. Primera prueba (igual que UaExpert):

| Parámetro | Valor |
|---|---|
| Security Policy | **None** |
| Message Security Mode | **None** |
| Authentication | **Anonymous** (preferido) |

4. Si la UI exige usuario/contraseña: cuenta Windows de la PC WinCC, p. ej. `desktop-bh0rbsv\siemens` (o `.\siemens`) con su contraseña. Grupo **SIMATIC HMI** en WinCC puede ayudar.  
5. **Trust** del certificado del servidor WinCC en el almacén de KEP (Rejected → Trusted).  
6. Browse / importar tags.

#### 6.4 Valores en tiempo real

El **árbol de configuración** de KEP **no anima** valores. Usar **OPC Quick Client**:

1. Abrir Quick Client.  
2. Conectar al runtime de KEP.  
3. Ver **Value** y **Quality** (Good + valores que cambian con el PLC).

#### 6.5 Si WinCC rechaza el certificado del cliente KEP

En PC WinCC:

```powershell
Get-ChildItem "C:\Program Files (x86)\Siemens\Automation\SCADA-RT_V11\WinCC\OPC\UAServer\PKI\CA\rejected"
Copy-Item "C:\Program Files (x86)\Siemens\Automation\SCADA-RT_V11\WinCC\OPC\UAServer\PKI\CA\rejected\*" `
  "C:\Program Files (x86)\Siemens\Automation\SCADA-RT_V11\WinCC\OPC\UAServer\PKI\CA\certs\" -Force
```

(Si `rejected` no existe o está vacío, el problema no es rechazo de cert de cliente.)

En planta, tras la prueba, preferir **SignAndEncrypt** en lugar de dejar None abierto.

---

### 21.8 Checklist mínimo de la etapa definitiva

1. `UAServer` + `Test-Path OpcUaServerWinCCPro.exe` = True.  
2. Runtime HMI cargado → `Start-Service` / `Restart-Service OpcUaServerWinCCPro`.  
3. Log **sin** `no runtime project was found`.  
4. `Running` + `0.0.0.0:4861` LISTENING.  
5. UaExpert: Custom Discovery `opc.tcp://localhost:4861` → None → Trust → tags.  
6. (Opcional) OPC Expert DA: `OPCServer.WinCC_SCADA.1`.  
7. Desde PC KEP: `Test-NetConnection <IP-WinCC> -Port 4861` = True.  
8. KEP OPC UA Client → endpoint con IP → Trust → **Quick Client** con valores vivos.

---

### 21.9 Tabla de desvíos importantes y soluciones (esta etapa)

| Desvío | Evidencia | Solución planteada |
|---|---|---|
| Instalación incompleta (estación de prueba) | Solo `UAClient`, sin servicio | ISO Runtime Professional desde carpeta local; casilla OPC UA Server |
| PKI corrupto (estación de prueba) | 0x80004005, *Couldn't access certificate store* | Mover `.der`/`.pfx` a `_bak`; recrear cert; `Start-Service` |
| UA Paused/Stopped con componente OK | `Get-Service` Paused | `Start-Service OpcUaServerWinCCPro` con Runtime abierto |
| Puerto arriba pero sin proyecto RT | Log: `no runtime project was found` | Runtime cargado + `Restart-Service OpcUaServerWinCCPro` |
| UaExpert no encuentra el server | Discovery en `:4840` | Custom Discovery a `:4861` |
| OK gris en Add Server | Sin endpoint seleccionado | Doble clic Custom Add Server → Discover → endpoint None |
| `@` ante el nombre | Árbol Project | Normal en entradas custom |
| KEP pide user/password | Diálogo auth | Anonymous; o `PC\usuario` de WinCC |
| Firewall | Duda al remoto | Solo si `Test-NetConnection … -Port 4861` = False |
| Tags OK pero “no animan” en KEP | Árbol de config | Usar **OPC Quick Client** |
| `rejected` PathNotFound | Get-ChildItem | No es fallo de cliente; otra causa (enlace RT, Trust, etc.) |

---

### 21.10 Bloque de comandos — copia rápida (PC WinCC)

```powershell
# Identidad
whoami
hostname
ipconfig

# Instalación + servicios
Test-Path "C:\Program Files (x86)\Siemens\Automation\SCADA-RT_V11\WinCC\OPC\UAServer\OpcUaServerWinCCPro.exe"
Get-ChildItem "C:\Program Files (x86)\Siemens\Automation\SCADA-RT_V11\WinCC\OPC"
Get-Service OpcUaServerWinCCPro, OPCServer.WinCC_SCADA -ErrorAction SilentlyContinue
netstat -ano | findstr "4861"

# Arranque / re-enlace al Runtime
Start-Service OpcUaServerWinCCPro
# o, con Runtime ya cargado:
Restart-Service OpcUaServerWinCCPro
Start-Sleep 5
Get-Service OpcUaServerWinCCPro
netstat -ano | findstr "4861"
Get-Content "C:\Users\Public\Documents\Siemens\WinCC\OPC\UAServer\OpcUaServerWinCCPro.txt" -Tail 20

# ProgIDs DA Professional
Get-ChildItem HKLM:\SOFTWARE\Classes, HKLM:\SOFTWARE\WOW6432Node\Classes -ErrorAction SilentlyContinue | Where-Object { $_.PSChildName -match 'OPCServer\.WinCC_SCADA' } | Select-Object -ExpandProperty PSChildName
```

**PC KEP (red):**

```powershell
Test-NetConnection 192.168.0.221 -Port 4861
```

**Clientes:**

| Cliente | Endpoint / ProgID |
|---|---|
| UaExpert (local) | `opc.tcp://localhost:4861` + Trust |
| KEPServer (remoto) | `opc.tcp://192.168.0.221:4861` + Trust → Quick Client |
| OPC Expert (DA) | `OPCServer.WinCC_SCADA.1` |

---

*Fin del hilo editado (incluye seguimiento §20 y etapa definitiva §21, 2026-08-25/26). Comandos: `OPC-PowerShell-Diagnostico.md`. Informe: `OPC-UA-WinCC-RT-Professional-V18.md`.*
