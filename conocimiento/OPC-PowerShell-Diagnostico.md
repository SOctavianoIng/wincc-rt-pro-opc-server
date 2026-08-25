# PowerShell — Diagnóstico OPC en WinCC Runtime Professional

**Propósito:** secuencia cronológica de comandos usados y validados (2026-08-18) para diagnosticar e instalar el servidor OPC (DA/UA) en una PC Station con WinCC RT Professional V18.

**Para quién:** técnico en planta o agente de IA en **otra estación**. Usar junto con:

- [`AGENT-PROMPT.md`](AGENT-PROMPT.md) — prompt de sesión
- [`AGENT-CONTEXT.md`](AGENT-CONTEXT.md) — hechos cerrados y árbol de decisión
- [`OPC-UA-WinCC-RT-Professional-V18.md`](OPC-UA-WinCC-RT-Professional-V18.md) — informe completo

**Reglas de uso**

- Ejecutar en la **PC del Runtime**, preferible **PowerShell como Administrador**.
- Runtime HMI abierto cuando se pruebe el servicio UA o el ProgID DA.
- Rutas **fijas**. No usar `Get-ChildItem -Recurse` sobre todo `C:\Program Files*\Siemens` (se cuelga).
- Un paso → evidencia → siguiente paso. No saltar a DCOM/firewall si el componente aún no está instalado.
- Sustituir `desktop-kc6qa5v\gigabyte-pc` por el usuario de `whoami` en la estación actual.

**Ruta base**

```text
C:\Program Files (x86)\Siemens\Automation\SCADA-RT_V11\WinCC
```

---

## Orden cronológico (mapa rápido)

| # | Fase | Qué responde |
|---|---|---|
| 1 | Contexto de estación | ¿Quién/dónde estamos? |
| 2 | Registro COM DA | ¿Existe ProgID Professional? |
| 3 | Inventario software | ¿Hay Runtime + OPCUA_Client o Server? |
| 4 | Procesos con Runtime | ¿Hay algo OPC vivo? |
| 5 | Disco: `bin` y `OPC` | ¿Cliente o servidor en disco? |
| 6 | Puerto 4861 / servicios | ¿Escucha UA? (no diagnostica DA) |
| 7 | Nombre correcto del exe | `OpcUaServerWinCCPro.exe` |
| 8 | Espacio / temporales | Antes del setup del ISO Runtime |
| 9 | Post-instalación | ¿Apareció `UAServer` y el servicio? |
| 10 | Arranque fallido 0x80004005 | Evento 7023 + config del servicio |
| 11 | Trace + log Siemens | Causa real (PKI) |
| 12 | Regenerar certificado | Arreglo que dejó Running + LISTENING |
| 13 | Comprobación viva UA | Estado final del servidor |
| 14 | Cliente / rejected | Solo si el cliente deja cert en rejected |

---

## 1. Contexto de la estación

### 1.1 Identidad Windows

```powershell
whoami
hostname
```

**Reseña:** fija el usuario y el hostname para Log On del servicio, grupo `SIMATIC HMI`, permisos PKI e endpoint `opc.tcp://<host>:4861`.

**Acción esperada:** algo como `DOMINIO\usuario` o `PC\usuario`, y el nombre de la máquina. Anotarlos en «Estación actual» del prompt del agente.

---

## 2. Fase A — ¿Está el OPC DA de Professional? (registro COM)

### 2.1 Claves del ProgID Professional

```powershell
Get-Item HKLM:\SOFTWARE\Classes\OPCServer.WinCC_SCADA -ErrorAction SilentlyContinue
Get-Item HKLM:\SOFTWARE\Classes\OPCServer.WinCC -ErrorAction SilentlyContinue
Get-Item HKLM:\SOFTWARE\WOW6432Node\Classes\OPCServer.WinCC_SCADA -ErrorAction SilentlyContinue
Get-Item HKLM:\SOFTWARE\WOW6432Node\Classes\OPCServer.WinCC -ErrorAction SilentlyContinue
```

**Reseña:** OPC Expert solo lista clases COM registradas. Si estas cuatro consultas no devuelven objeto, el DA de Professional **no está instalado/registrado**.

**Acción esperada:**

- **Con salida:** el ProgID existe → probar en OPC Expert `OPCServer.WinCC_SCADA.1` con Runtime abierto.
- **Sin salida:** no pelear DCOM ni agregar ProgID a mano → falta el componente del ISO Runtime Professional.

### 2.2 Listar ProgIDs OPC/WinCC relevantes

```powershell
Get-ChildItem HKLM:\SOFTWARE\Classes, HKLM:\SOFTWARE\WOW6432Node\Classes -ErrorAction SilentlyContinue |
  Where-Object { $_.PSChildName -match 'OPCServer|WinCC_SCADA|WinCC\.OPC|SimaticHMI' } |
  Select-Object -ExpandProperty PSChildName
```

**Reseña:** inventario filtrado (no recorrer todo el registro). Distingue Advanced (`CoRtHmiRTm`) de Professional (`WinCC_SCADA`).

**Acción esperada:** pueden aparecer `OPC.SimaticHMI.CoRtHmiRTm` / `.1` aunque el HMI sea Professional (TIA registra la clase). Eso **no** es el servidor correcto. Debe aparecer `OPCServer.WinCC_SCADA` / `.1` si DA Professional está instalado.

---

## 3. Fase C — Inventario de software instalado

```powershell
Get-ItemProperty HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Uninstall\*,
                 HKLM:\SOFTWARE\WOW6432Node\Microsoft\Windows\CurrentVersion\Uninstall\* |
  Where-Object { $_.DisplayName -match 'WinCC|OPC|SCADA' } |
  Select-Object DisplayName, DisplayVersion |
  Sort-Object DisplayName |
  Format-Table -AutoSize
```

**Reseña:** confirma Runtime Professional, Update, y si solo hay `OPCUA_Client` o ya hay servidor.

**Acción esperada:** típico **antes** de instalar el motor:

- `SIMATIC WinCC Runtime Professional V18…`
- `WinCC Runtime Professional V18 - OPCUA_Client`

Si solo está el Client, TIA puede mostrar OPC UA en Runtime settings **sin** tener el motor en disco.

---

## 4. Procesos con Runtime en ejecución

```powershell
Get-Process | Where-Object { $_.ProcessName -match 'opc|wincc|CCPtm|pdlrt' } |
  Select-Object ProcessName, Id, Path
```

**Reseña:** comprueba que el HMI/Runtime está vivo antes de culpar a OPC. No sustituye la prueba de registro COM ni de `UAServer`.

**Acción esperada:** procesos de Runtime (p. ej. relacionados a `pdlrt` / WinCC). Ver `opcualds` **no** significa que WinCC UA esté arriba (es el Local Discovery Server de OPC Foundation).

---

## 5. Disco — cliente vs servidor (rutas fijas)

### 5.1 Canales en `bin` (suelen ser cliente)

```powershell
Get-ChildItem "C:\Program Files (x86)\Siemens\Automation\SCADA-RT_V11\WinCC\bin" -Filter "*OPC*"
```

**Reseña:** `OPC.chn`, `OPC UA WinCC Channel.chn`, `OPCTags.exe` y un XML plantilla son típicos de **cliente / canal**. No prueban que el servidor esté instalado.

**Acción esperada:** listado de canales/cliente. Un `OpcUaServerWinCC.xml` en `bin` **no** implica servicio UA.

### 5.2 Carpeta `WinCC\OPC`

```powershell
Get-ChildItem "C:\Program Files (x86)\Siemens\Automation\SCADA-RT_V11\WinCC\OPC"
```

**Reseña:** diferenciador clave. Solo `UAClient` = falta el servidor. Tras instalación correcta aparecen `UAServer`, `DataAccess`, etc.

**Acción esperada:**

| Antes | Después (OK) |
|---|---|
| Solo `UAClient` | `UAServer`, `DataAccess`, `HistDataAccess`, `AlarmEvent`, … |

### 5.3 ¿Existe el ejecutable del servidor UA?

```powershell
Test-Path "C:\Program Files (x86)\Siemens\Automation\SCADA-RT_V11\WinCC\OPC\UAServer\OpcUaServerWinCCPro.exe"
```

**Reseña:** nombre correcto en RT Professional V18: **`OpcUaServerWinCCPro.exe`**. `OpcUaServerWinCC.exe` es de WinCC clásico V7 y da `False` aunque el servidor esté bien.

**Acción esperada:** `True` si el componente Tools → WinCC OPC UA Server quedó instalado. `False` → instalar desde ISO **WinCC Runtime Professional** (no ISO TIA, no Unified).

### 5.4 Contenido de `UAServer` (post-instalación)

```powershell
Get-ChildItem "C:\Program Files (x86)\Siemens\Automation\SCADA-RT_V11\WinCC\OPC\UAServer" |
  Select-Object Name, Mode, LastWriteTime
```

**Reseña:** debe verse el `.exe`, el `.xml` de configuración y la carpeta `PKI`.

**Acción esperada:** `OpcUaServerWinCCPro.exe`, `OpcUaServerWinCCPro.xml`, `PKI`, DLLs.

---

## 6. Fase B — Servicios y puerto 4861 (solo OPC UA)

> **Importante:** `netstat` sobre 4861 **no diagnostica OPC DA**. DA va por COM/DCOM.

### 6.1 Servicios OPC / WinCC

```powershell
Get-Service *OpcUa*
Get-Service *WinCC*
Get-Service OpcUaServerWinCCPro, OPCServer.WinCC_SCADA -ErrorAction SilentlyContinue
```

**Reseña:** el servicio UA es `OpcUaServerWinCCPro`. Tras instalar DA también puede existir `OPCServer.WinCC_SCADA` (a menudo Stopped hasta que el Runtime lo use).

**Acción esperada:**

- Sin match / error → componente no instalado.
- `Stopped` / `Paused` → intentar arranque (sección 10).
- `Running` → pasar a `netstat`.

### 6.2 ¿Escucha el endpoint UA?

```powershell
netstat -ano | findstr "4861"
```

**Reseña:** prueba de vida del servidor UA. Vacío con servicio Stopped = motor apagado o inexistente. Vacío con TIA mostrando puerto 4861 = Engineering configurado, motor no instalado/corrido.

**Acción esperada (OK):**

```text
TCP    0.0.0.0:4861    0.0.0.0:0    LISTENING    <PID>
TCP    [::]:4861       [::]:0       LISTENING    <PID>
```

---

## 7. Antes del setup — espacio en disco

```powershell
Remove-Item "$env:TEMP\*" -Recurse -Force -ErrorAction SilentlyContinue
```

**Reseña:** el setup de Runtime Professional pide holgura (~14.5 GB en la referencia). Disco en rojo interrumpe o deja instalación a medias. Complementar con Liberador (`cleanmgr`) y limpieza de `C:\Windows\Temp` (admin). **No** borrar `Siemens` ni datos SQL `WINCC`.

**Acción esperada:** más espacio libre. Verificar que `MSSQL$WINCC` ya corre antes de marcar reinstalar SQL en el setup.

---

## 8. Post-instalación — ¿quedó el componente?

Repetir en este orden:

```powershell
Get-ChildItem "C:\Program Files (x86)\Siemens\Automation\SCADA-RT_V11\WinCC\OPC"
Test-Path "C:\Program Files (x86)\Siemens\Automation\SCADA-RT_V11\WinCC\OPC\UAServer\OpcUaServerWinCCPro.exe"
Get-Service OpcUaServerWinCCPro, OPCServer.WinCC_SCADA -ErrorAction SilentlyContinue
netstat -ano | findstr "4861"
```

**Reseña:** checklist mínima tras el ISO **SIMATIC WinCC Runtime Professional** (Tools → WinCC OPC UA Server / DA). Si el setup “terminó” pero solo hay `UAClient`, en VM suele ser ISO desconectado al reboot → instalar desde `C:\Temp\WinCC_RT_V18\`.

**Acción esperada:** `UAServer` presente, `Test-Path` = `True`, servicio existe (aunque esté Stopped). Luego alinear Update del Runtime con TIA (ej. Upd5).

---

## 9. Arranque del servicio UA

### 9.1 Start + verificación

```powershell
Start-Service OpcUaServerWinCCPro
Start-Sleep 4
Get-Service OpcUaServerWinCCPro
netstat -ano | findstr "4861"
```

**Reseña:** con Runtime abierto, el servicio debería quedar Running y 4861 en LISTENING. Si falla, no insistir a ciegas: pasar a diagnóstico 7023 / log.

**Acción esperada (OK):** `Status : Running` + líneas LISTENING en 4861.

### 9.2 Configuración del servicio (`sc`)

```powershell
sc.exe qc OpcUaServerWinCCPro
sc.exe start OpcUaServerWinCCPro
```

**Reseña:** `qc` muestra BINARY_PATH (`...\OpcUaServerWinCCPro.exe`), cuenta (suele ser LocalSystem), AUTO_START y dependencia RPCSS. `start` puede mostrar `START_PENDING` y un PID que luego **muere** si hay 0x80004005.

**Acción esperada (fallo típico):** START_PENDING → proceso desaparece → servicio Stopped. Continuar con evento 7023.

### 9.3 Evento System 7023

```powershell
Get-WinEvent -FilterHashtable @{LogName='System'; Id=7023; StartTime=(Get-Date).AddHours(-3)} |
  Where-Object { $_.Message -match 'OpcUaServerWinCCPro' } |
  Select-Object -First 5 TimeCreated, @{n='Msg';e={$_.Message}} |
  Format-List
```

**Reseña:** Windows solo dice *Unspecified error* (HRESULT `0x80004005`). El detalle **no** está en Application Event 1000; está en el log Siemens (sección 10).

**Acción esperada:** mensaje *The OpcUaServerWinCCPro service terminated with the following error: Unspecified error*. Activar trace; no culpar licencia Process Historian.

---

## 10. Trace del servidor UA (imprescindible con 0x80004005)

### 10.1 Activar TraceEnable en el XML

```powershell
$xml = "C:\Program Files (x86)\Siemens\Automation\SCADA-RT_V11\WinCC\OPC\UAServer\OpcUaServerWinCCPro.xml"
(Get-Content $xml -Raw).Replace('<TraceEnable>false</TraceEnable>','<TraceEnable>true</TraceEnable>') |
  Set-Content $xml -Encoding UTF8
[xml](Get-Content $xml -Raw); "XML OK"
```

**Reseña:** habilita el log de texto del servidor. Validar que el XML sigue siendo parseable (`XML OK`). Si el XML queda roto, el servicio no arranca.

**Acción esperada:** `XML OK`. Luego intentar de nuevo `Start-Service`.

### 10.2 Leer el log Siemens

```powershell
Get-Content "C:\Users\Public\Documents\Siemens\WinCC\OPC\UAServer\OpcUaServerWinCCPro.txt" -Tail 50
```

**Reseña:** fuente de verdad cuando Windows solo dice Unspecified error.

**Acción esperada (caso de referencia — PKI corrupto):**

```text
Couldn't access certificate store.
Please install a valid certificate and restart the server.
```

**Acción esperada (tras regenerar cert):**

```text
No own certificate found. Creating a new application instance certificate.
```

También conviene mirar fechas de certs viejos:

```powershell
Get-ChildItem "C:\Program Files (x86)\Siemens\Automation\SCADA-RT_V11\WinCC\OPC\UAServer\PKI\CA\certs"
Get-ChildItem "C:\Program Files (x86)\Siemens\Automation\SCADA-RT_V11\WinCC\OPC\UAServer\PKI\CA\private"
```

Si `.der`/`.pfx` son de una instalación a medias / BSOD, son candidatos a regenerar.

---

## 11. Regenerar certificado PKI (arreglo que levantó el servicio)

> Sustituir el usuario por el de `whoami` en la estación destino.

```powershell
$ua = "C:\Program Files (x86)\Siemens\Automation\SCADA-RT_V11\WinCC\OPC\UAServer"
$pki = "$ua\PKI"

try { [xml](Get-Content "$ua\OpcUaServerWinCCPro.xml" -Raw); "XML OK" } catch { "XML ROTO: $_" }

New-Item "$pki\CA\rejected" -ItemType Directory -Force | Out-Null
icacls $ua /grant "$env:USERDOMAIN\$env:USERNAME:(OI)(CI)F" /T

New-Item "$pki\_bak" -ItemType Directory -Force | Out-Null
Move-Item "$pki\CA\certs\*.der" "$pki\_bak\" -Force -ErrorAction SilentlyContinue
Move-Item "$pki\CA\private\*.pfx" "$pki\_bak\" -Force -ErrorAction SilentlyContinue

Start-Service OpcUaServerWinCCPro
Start-Sleep 4
Get-Service OpcUaServerWinCCPro
Get-ChildItem "$pki\CA\certs","$pki\CA\private" | Select-Object FullName, Length
netstat -ano | findstr "4861"
Get-Content "C:\Users\Public\Documents\Siemens\WinCC\OPC\UAServer\OpcUaServerWinCCPro.txt" -Tail 15
```

**Reseña:** mueve certificados inválidos a `_bak`, deja `ForceCertificateCreation` crear unos nuevos, arranca el servicio y verifica puerto + log. Los permisos `icacls` solos **no bastaron** en la referencia; lo decisivo fue regenerar el cert.

**Acción esperada:**

- `Status : Running`
- `.der` y `.pfx` **nuevos** en `certs` / `private`
- `4861` LISTENING
- Log: creación de certificate de instancia

Si `$env:USERDOMAIN\$env:USERNAME` no aplica (cuenta Microsoft / AzureAD), usar el string exacto de `whoami`.

---

## 12. Comprobación rápida “¿está vivo UA?”

```powershell
Get-Service OpcUaServerWinCCPro
netstat -ano | findstr "4861"
```

**Reseña:** smoke test final antes de abrir el cliente. Si falla aquí, no pelear credenciales en OPC Expert/UaExpert.

**Acción esperada:**

```text
Running  OpcUaServerWinCCPro
TCP    0.0.0.0:4861    ...    LISTENING
```

---

## 13. Grupo SIMATIC HMI (opcional, clientes Windows)

```powershell
Add-LocalGroupMember -Group "SIMATIC HMI" -Member "$env:USERNAME" -ErrorAction SilentlyContinue
Get-LocalGroupMember "SIMATIC HMI"
```

**Reseña:** a veces ayuda con permisos OPC/Runtime. En la referencia no fue el factor que levantó el servicio (fue el PKI).

**Acción esperada:** el usuario de la sesión figura en el grupo.

---

## 14. Certificados de cliente en `rejected` (si aplica)

```powershell
Get-ChildItem "C:\Program Files (x86)\Siemens\Automation\SCADA-RT_V11\WinCC\OPC\UAServer\PKI\CA\rejected"
Get-ChildItem "C:\Program Files (x86)\Siemens\Automation\SCADA-RT_V11\WinCC\OPC\UAServer\PKI\CA\certs"
Copy-Item "C:\Program Files (x86)\Siemens\Automation\SCADA-RT_V11\WinCC\OPC\UAServer\PKI\CA\rejected\*" `
  "C:\Program Files (x86)\Siemens\Automation\SCADA-RT_V11\WinCC\OPC\UAServer\PKI\CA\certs\" -Force
```

**Reseña:** confiar certs de **cliente** que el servidor rechazó. En la prueba de referencia `rejected` estuvo vacío; el fallo de OPC Expert UA no fue este. En UaExpert el paso del lado **cliente** es **TRUST SERVER CERTIFICATE** ante `BadCertificateUntrusted` (normal con cert autofirmado).

**Acción esperada:** si hay `.der` en `rejected`, copiarlos a `certs` y reintentar conexión.

---

## 15. Clientes (no PowerShell, pero cierre del diagnóstico)

| Canal | Qué usar | Evidencia de éxito |
|---|---|---|
| **OPC DA** | OPC Expert → `OPCServer.WinCC_SCADA.1` | Tags visibles. No es UA. |
| **OPC UA** | UaExpert → `opc.tcp://localhost:4861`, endpoint **None**, Trust | Tags visibles. |
| Optix / Ignition | Mismo endpoint UA (`opc.tcp://IP:4861`) | No usar ProgID DA. |

No mezclar: `BadCertificateUntrusted` (cliente) ≠ `Couldn't access certificate store` (servicio).

---

## 16. Cómo debe usarlo el agente (otra PC)

Seguir el protocolo de [`AGENT-PROMPT.md`](AGENT-PROMPT.md) y el árbol de [`AGENT-CONTEXT.md`](AGENT-CONTEXT.md):

1. Completar «Estación actual» (`whoami`, hostname, tipo HMI, síntoma).
2. Clasificar: **A** falta componente · **B** servicio no arranca · **C** Running pero falla cliente · **D** ProgID equivocado · **E** Runtime/MPI no sano.
3. Ejecutar **solo** el bloque de comandos de la fase correspondiente (no toda la lista de una vez).
4. Pedir evidencia (pegar salida) antes de reinstalar o cambiar de ISO.
5. Si es **B** con 7023 → secciones 9 → 10 → 11 de este documento.
6. Si es **A** → ISO Runtime Professional desde carpeta local; luego sección 8.

---

## 17. Anti-patrones (comandos / interpretaciones a evitar)

| Evitar | Por qué |
|---|---|
| `netstat 4861` para diagnosticar DA | DA no usa ese puerto |
| `Test-Path ...\OpcUaServerWinCC.exe` | Nombre incorrecto en RT Professional V18 |
| `Get-ChildItem -Recurse` masivo sobre Siemens | Cuelga / tarda minutos |
| Agregar `OPCServer.WinCC_SCADA` a mano en OPC Expert | Sin registro COM → Class not registered |
| Culpar Process Historian / DCOM remoto primero | Primero componente + servicio + log PKI |
| Confundir solo `UAClient` con servidor instalado | Falta Tools → WinCC OPC UA Server |

---

*Documento derivado de la solución validada el 2026-08-18. Actualizar solo si hay un hallazgo nuevo en otra estación.*
