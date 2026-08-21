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
- `AGENT-CONTEXT.md` — marco para un agente de IA  
- `AGENT-PROMPT.md` — prompt listo para otra estación  

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

*Fin del hilo editado. Para comandos adicionales y tablas de desvíos, ver `OPC-UA-WinCC-RT-Professional-V18.md`.*
