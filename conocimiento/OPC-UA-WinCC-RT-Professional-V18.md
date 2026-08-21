# Informe: OPC DA y OPC UA en WinCC Runtime Professional V18

**Proyecto:** PC Station `HMI_RT_1` [WinCC RT Professional] leyendo un **S7-315-2DP** por **MPI** (PC Adapter USB / RS485).  
**Objetivo:** exponer tags de WinCC a un cliente (OPC Expert, luego Optix / Ignition).  
**Equipo de prueba:** VM Windows, hostname `DESKTOP-KC6QA5V`, usuario `desktop-kc6qa5v\gigabyte-pc`.  
**Fecha de validación:** 2026-08-18.  
**Resultado:** OPC DA y OPC UA funcionando. Tags visibles en OPC Expert (DA) y UaExpert (UA).

Arquitectura real:

```
S7-315-2DP  --MPI RS485-->  WinCC RT Professional (PC Station)
                              |-- OPC DA  ProgID OPCServer.WinCC_SCADA / OPCServer.WinCC_SCADA.1
                              |-- OPC UA  opc.tcp://<host>:4861
                                          servicio OpcUaServerWinCCPro
```

El cliente OPC **no habla con el PLC**. El 315-2DP **no tiene Ethernet**.

---

## 1. Resultado final (lo que quedó funcionando)

| Canal | Cómo se identifica | Servicio Windows | Cliente de prueba |
|---|---|---|---|
| **OPC DA** | ProgID `OPCServer.WinCC_SCADA` y versión `OPCServer.WinCC_SCADA.1` | `OPCServer.WinCC_SCADA` (arranca con Runtime) | OPC Expert |
| **OPC UA** | URL `opc.tcp://localhost:4861` | `OpcUaServerWinCCPro` | UaExpert (y OPC Expert si soporta UA) |

Ejecutable UA:

`C:\Program Files (x86)\Siemens\Automation\SCADA-RT_V11\WinCC\OPC\UAServer\OpcUaServerWinCCPro.exe`

**No** se llama `OpcUaServerWinCC.exe`. Ese nombre es de WinCC clásico V7 y hizo perder tiempo en `Test-Path`.

Configuración TIA (Runtime settings → OPC settings):

- Puerto **4861**
- URL `opc.tcp://[HostName]:4861`
- Política **None** habilitada para prueba
- En planta: pasar a Sign / SignAndEncrypt

---

## 2. Cronología de lo que pasó

### Fase A — Pedido inicial: leer tags por OPC DA

Se buscaba leer variables desde WinCC Runtime Professional V18 con **OPC Expert**. El Runtime ya levantaba datos del PLC por MPI.

Se encontró un servidor DA, se eligió el **equivocado**:

- Descripción: `OPC.SimaticHMI.CoRtHmiRTm`
- ProgID: `OPC.SimaticHMI.CoRtHmiRTm.1`
- Mensaje OPC Expert: *OPC Server is not using a port because process is not running* / *Process is not currently running*

Ese ProgID es el OPC DA de **WinCC Runtime Advanced / Comfort**, no el de Professional. TIA registra la clase COM aunque Advanced no esté en ejecución. Por eso OPC Expert lo “ve” y el proceso no existe.

Otros DA que sí aparecían y **tampoco** son el de Professional:

- `OPC.Siemens.XML.1` — pasarela XML-DA
- `OPC.SimaticNET.1` — SIMATIC NET hacia PLC (no WinCC; además el MPI ya lo usa WinCC)
- `OPC.SimaticNET.DP.1` — Profibus DP vía SIMATIC NET

**No estaban** en la lista: `OPCServer.WinCC_SCADA` ni `OPCServer.WinCC`.

### Fase B — Confusión con OPC UA (chat previo y netstat 4861)

En un chat anterior se había insistido en OPC UA (puertos 4861/4870). `netstat` no mostraba nada en 4861. Se concluyó (a medias) que faltaba el componente UA Server y que solo estaba `OPCUA_Client`.

Eso era **cierto para el motor**, pero se mezcló con DA. OPC DA **no usa** el puerto 4861. La prueba de `netstat` no diagnostica DA.

En TIA, Runtime settings **sí** mostraba “Configuration of the OPC Unified Architecture Server” y puerto 4861. Esa pantalla es del **Engineering**: configura el proyecto; **no instala** `OpcUaServerWinCCPro.exe`.

### Fase C — El Runtime Professional está, el servidor OPC no

Con Runtime **en ejecución** y tags MPI válidas:

- Registro Windows: **no** existían `OPCServer.WinCC_SCADA` ni `OPCServer.WinCC` (ni en `HKLM:\SOFTWARE\Classes` ni en `WOW6432Node`).
- No había `OPCServer*.exe`.
- Software instalado: Runtime Professional V18 + **`OPCUA_Client`** + simulación SCADA. **No** había paquete OPC UA Server.
- En disco, `WinCC\bin` tenía canales **cliente** (`OPC.chn`, `OPC UA WinCC Channel.chn`, `OPCTags.exe`) y un XML de plantilla `OpcUaServerWinCC.xml`.
- `WinCC\OPC` solo tenía **`UAClient`**.

Conclusión de esa fase: TIA + Updates pueden dejar un Runtime que **ejecuta el HMI y el MPI** sin copiar los **servidores** OPC (DA/UA). El casillero no está en el setup de TIA Portal.

### Fase D — Dónde está el instalador (desvío largo)

Se intentó Modify/Repair desde **TIA Portal V18 STEP 7 Prof. + WinCC Professional**. Overview solo listaba **OPC XML Gateway**. Aviso: componentes ya instalados en versión más alta (Upd5) no se reparan.

Ese ISO es **Engineering**, no Runtime Professional.

En SiePortal:

- **109807122 / 109814516** = **WinCC Unified PC RT** → **no usar**.
- **109807225** = página de **Updates** de WinCC RT Professional V18, no el ISO base.
- V18 está en product cancellation: el base casi no se publica como trial.

El ISO que sí sirve se titula **SIMATIC WinCC Runtime Professional V18**. Suele venir con la licencia **6AV2105-…** (DVD) o download comercial. En *Tools* aparecen:

- WinCC OPC DA Server
- WinCC OPC HDA Server
- WinCC OPC A&E Server
- WinCC OPC-XML DA Server
- **WinCC OPC UA Server** ← el que faltaba

En la primera pasada el disco C: estaba en **rojo** (12 GB libres vs 14.5 GB pedidos). No reinstalar SQL Server 2019 si ya corre `MSSQL$WINCC`.

### Fase E — Instalación accidentada en la VM

1. Primera corrida del setup: casilla OPC UA marcada, “sin error”, reinicio. **No había** `UAServer` ni servicio. Causa: al reiniciar la VM se **desconectó el ISO** y el setup no terminó de copiar.
2. Al continuar: varios reinicios. Los de Siemens son **normales** si el asistente retoma. Una **pantalla azul** (“la PC ha tenido un problema”) **no** es el reinicio de Siemens.
3. Cuando el setup **sí terminó**: aparecieron carpetas `UAServer`, `DataAccess`, `HistDataAccess`, `AlarmEvent`, etc. Servicio `OpcUaServerWinCCPro` (primero Paused, luego Stopped). El exe real es `OpcUaServerWinCCPro.exe`.
4. Hacía falta **Update 5** del Runtime para igualar TIA V18 Upd5 antes de abrir el proyecto.

Recomendación para la próxima PC: copiar el ISO a `C:\Temp\WinCC_RT_V18\` y ejecutar `Setup.exe` **desde disco**, no desde DVD virtual.

### Fase F — Servicio instalado pero no arranca (0x80004005)

Runtime leyendo el PLC. `Start-Service OpcUaServerWinCCPro` fallaba.

- Evento System **7023**: *The OpcUaServerWinCCPro service terminated with the following error: Unspecified error*
- `sc.exe start` llegaba a `START_PENDING` con un PID y el proceso **moría**.
- Application log **sin** evento 1000.
- Licencias ALM **OK** (WinCC Runtime Professional). La licencia **OPC UA Server Process Historian** **no aplica**.
- Cambiar el Log On del servicio a `gigabyte-pc` **no alcanzó**. Windows **no deja** iniciar un servicio con cuenta local **sin contraseña**; hubo que crear una. Aun así, 0x80004005.

El XML del servidor estaba bien (puerto 4861, None, `ForceCertificateCreation=true`). Al ejecutar el `.exe` a mano volvía al prompt sin texto (es un servicio).

Se activó **trace** en el XML. El log real:

`C:\Users\Public\Documents\Siemens\WinCC\OPC\UAServer\OpcUaServerWinCCPro.txt`

```
Couldn't access certificate store.
Please install a valid certificate and restart the server.
```

Había `.der` y `.pfx` creados a las **19:14** (instalación a medias / BSOD): certificado **inválido**. Permisos NTFS en PKI no bastaron.

**Arreglo que funcionó:** mover esos certificados a `PKI\_bak`, dejar que el servidor cree unos nuevos. El servicio pasó a **Running** y **4861 LISTENING**.

### Fase G — Clientes

1. **OPC Expert + DA** `OPCServer.WinCC_SCADA.1` → **funcionó** (tags OK). Eso **no** es OPC UA; es COM/DA.
2. **OPC Expert + UA** `opc.tcp://localhost:4861`, None + Anonymous y luego usuario Windows → descubría el servidor y el certificado, luego: *Failed to connect … using given credentials*. Carpeta `rejected` vacía; el log UA **no registró** el intento. Conclusión: stack UA de OPC Expert poco fiable aquí. Se hizo **Clear** de credenciales.
3. **UaExpert** → `BadCertificateUntrusted` (normal, cert autofirmado). **TRUST SERVER CERTIFICATE** → **tags visibles**. OPC UA validado.

Para Optix/Ignition usar el camino **UaExpert** (UA), no el ProgID DA.

---

## 3. Desvíos (qué no volver a hacer)

| Desvío | Por qué no |
|---|---|
| Conectar OPC Expert a `OPC.SimaticHMI.CoRtHmiRTm.1` | Es Advanced. Proceso no corre. |
| Diagnosticar DA con `netstat` 4861/4870 | DA no usa esos puertos. |
| “Agregar” a mano `OPCServer.WinCC_SCADA` en OPC Expert sin registro COM | Falla Class not registered. |
| Instalar/reparar con ISO de **TIA Portal** | No trae OPC UA Server de Professional. |
| Bajar **WinCC Unified PC RT** | Otro producto. |
| Confiar en SiePortal para el ISO **base** V18 | Ahí están los Updates. El base es el DVD Runtime Professional. |
| Licencia Process Historian OPC UA | No arranca `OpcUaServerWinCCPro`. |
| SIMATIC NET OPC hacia el PLC | No son tags de WinCC; el MPI ya está ocupado. |
| Buscar `OpcUaServerWinCC.exe` | En RT Professional V18 el nombre es `OpcUaServerWinCCPro.exe`. |
| `Get-ChildItem -Recurse` de todo Siemens | Tarda minutos; usar rutas fijas. |
| Instalar Runtime desde ISO de VM | Al reboot se desmonta y no copia `UAServer`. |
| Creer que TIA OPC settings = servidor instalado | Solo configura el proyecto. |
| Anonymous/usuario en OPC Expert UA sin UaExpert | En esta PC falló por el cliente, no por WinCC. |
| Tratar `BadCertificateUntrusted` como cert corrupto | Primera vez: hay que **Trust** en UaExpert. |

---

## 4. Procedimiento limpio para otra PC (orden correcto)

1. TIA V18 + PC Station **WinCC RT Professional** comunicando MPI con el PLC.
2. ALM: licencia **WinCC Runtime Professional** válida.
3. Copiar ISO **WinCC Runtime Professional V18** a `C:\Temp\WinCC_RT_V18\`.
4. Setup personalizado → Tools → marcar **WinCC OPC UA Server** (y DA si se quiere). Disco **sin rojo**. No reinstalar SQL si `MSSQL$WINCC` ya corre.
5. Dejar que reinicie las veces que pida Siemens (asistente debe retomar). Medio siempre accesible (carpeta local).
6. Instalar el **mismo Update** que TIA (Upd5).
7. Verificar carpeta `UAServer` + servicio `OpcUaServerWinCCPro`.
8. Compilar, arrancar HMI, tags MPI OK.
9. Si el servicio no queda Running: log de certificado → mover `.der`/`.pfx` viejos → `Start-Service`.
10. UaExpert: `opc.tcp://localhost:4861`, endpoint **None**, **Trust certificate**, ver tags.
11. OPC Expert DA: `OPCServer.WinCC_SCADA.1` si se necesita DA.

---

## 5. Comandos PowerShell relevantes (los que se usaron)

Ejecutar en la **PC del Runtime**. Varios requieren **administrador**. El Runtime debe estar abierto cuando se prueba el servicio UA.

### 5.1 Registro COM — ¿existe el servidor DA de Professional?

```powershell
Get-Item HKLM:\SOFTWARE\Classes\OPCServer.WinCC_SCADA -ErrorAction SilentlyContinue
Get-Item HKLM:\SOFTWARE\Classes\OPCServer.WinCC -ErrorAction SilentlyContinue
Get-Item HKLM:\SOFTWARE\WOW6432Node\Classes\OPCServer.WinCC_SCADA -ErrorAction SilentlyContinue
Get-Item HKLM:\SOFTWARE\WOW6432Node\Classes\OPCServer.WinCC -ErrorAction SilentlyContinue
```

Vacío = DA de Professional **no registrado**.

Listar ProgIDs OPC/WinCC (filtro; no recorrer todo el registro a ciegas):

```powershell
Get-ChildItem HKLM:\SOFTWARE\Classes, HKLM:\SOFTWARE\WOW6432Node\Classes -ErrorAction SilentlyContinue |
  Where-Object { $_.PSChildName -match 'OPCServer|WinCC_SCADA|WinCC\.OPC|SimaticHMI' } |
  Select-Object -ExpandProperty PSChildName
```

### 5.2 ¿Qué software OPC/WinCC hay instalado?

```powershell
Get-ItemProperty HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Uninstall\*,
                 HKLM:\SOFTWARE\WOW6432Node\Microsoft\Windows\CurrentVersion\Uninstall\* |
  Where-Object { $_.DisplayName -match 'WinCC|OPC|SCADA' } |
  Select-Object DisplayName, DisplayVersion |
  Sort-Object DisplayName |
  Format-Table -AutoSize
```

### 5.3 Archivos en disco (rutas fijas; no recurse de todo Siemens)

```powershell
Get-ChildItem "C:\Program Files (x86)\Siemens\Automation\SCADA-RT_V11\WinCC\bin" -Filter "*OPC*"
Get-ChildItem "C:\Program Files (x86)\Siemens\Automation\SCADA-RT_V11\WinCC\OPC"
Test-Path "C:\Program Files (x86)\Siemens\Automation\SCADA-RT_V11\WinCC\OPC\UAServer\OpcUaServerWinCCPro.exe"
Get-ChildItem "C:\Program Files (x86)\Siemens\Automation\SCADA-RT_V11\WinCC\OPC\UAServer" |
  Select-Object Name, Mode, LastWriteTime
```

Nombre correcto del exe: **`OpcUaServerWinCCPro.exe`**.  
`Test-Path ...\OpcUaServerWinCC.exe` da **False** aunque el servidor esté instalado.

### 5.4 Servicios y puerto UA

```powershell
Get-Service *OpcUa*
Get-Service *WinCC*
Get-Service OpcUaServerWinCCPro
netstat -ano | findstr "4861"
```

Estado bueno: `OpcUaServerWinCCPro` = **Running** y `TCP 0.0.0.0:4861 LISTENING`.

Config del servicio (ruta del exe, cuenta, dependencias):

```powershell
sc.exe qc OpcUaServerWinCCPro
sc.exe start OpcUaServerWinCCPro
```

Evento si muere al arrancar:

```powershell
Get-WinEvent -FilterHashtable @{LogName='System'; Id=7023; StartTime=(Get-Date).AddHours(-3)} |
  Where-Object { $_.Message -match 'OpcUaServerWinCCPro' } |
  Select-Object -First 5 TimeCreated, @{n='Msg';e={$_.Message}} |
  Format-List
```

El 7023 que vimos: *Unspecified error* (HRESULT `0x80004005`). El detalle está en el log Siemens, no en Windows.

### 5.5 Usuario Windows (para Log On del servicio, si se usa cuenta local)

```powershell
whoami
```

Salida de la VM de prueba: `desktop-kc6qa5v\gigabyte-pc`.

Grupo Siemens (OPC / Runtime):

```powershell
Add-LocalGroupMember -Group "SIMATIC HMI" -Member "gigabyte-pc" -ErrorAction SilentlyContinue
Get-LocalGroupMember "SIMATIC HMI"
```

### 5.6 Trace del servidor UA (imprescindible si 0x80004005)

Activar `<TraceEnable>true</TraceEnable>` en la sección `<Trace>` de:

`C:\Program Files (x86)\Siemens\Automation\SCADA-RT_V11\WinCC\OPC\UAServer\OpcUaServerWinCCPro.xml`

PowerShell que se usó (cambia **todas** las apariciones de TraceEnable false→true):

```powershell
$xml = "C:\Program Files (x86)\Siemens\Automation\SCADA-RT_V11\WinCC\OPC\UAServer\OpcUaServerWinCCPro.xml"
(Get-Content $xml -Raw).Replace('<TraceEnable>false</TraceEnable>','<TraceEnable>true</TraceEnable>') |
  Set-Content $xml -Encoding UTF8
[xml](Get-Content $xml -Raw); "XML OK"
```

Leer el log:

```powershell
Get-Content "C:\Users\Public\Documents\Siemens\WinCC\OPC\UAServer\OpcUaServerWinCCPro.txt" -Tail 50
```

Mensajes clave:

```
Couldn't access certificate store.
Please install a valid certificate and restart the server.
No own certificate found. Creating a new application instance certificate.
```

### 5.7 Permisos PKI y regenerar certificado (el arreglo que levantó el servicio)

```powershell
$ua = "C:\Program Files (x86)\Siemens\Automation\SCADA-RT_V11\WinCC\OPC\UAServer"
$pki = "$ua\PKI"

New-Item "$pki\CA\rejected" -ItemType Directory -Force | Out-Null
icacls $ua /grant "desktop-kc6qa5v\gigabyte-pc:(OI)(CI)F" /T

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

En otra PC, sustituir `desktop-kc6qa5v\gigabyte-pc` por el usuario de `whoami`.

Tras regenerar deben aparecer un `.der` y un `.pfx` **nuevos**. El servicio debe quedar **Running**.

### 5.8 Confiar certificado de cliente (si UaExpert/OPC Expert deja un .der en rejected)

```powershell
Get-ChildItem "C:\Program Files (x86)\Siemens\Automation\SCADA-RT_V11\WinCC\OPC\UAServer\PKI\CA\rejected"
Get-ChildItem "C:\Program Files (x86)\Siemens\Automation\SCADA-RT_V11\WinCC\OPC\UAServer\PKI\CA\certs"
Copy-Item "C:\Program Files (x86)\Siemens\Automation\SCADA-RT_V11\WinCC\OPC\UAServer\PKI\CA\rejected\*" `
  "C:\Program Files (x86)\Siemens\Automation\SCADA-RT_V11\WinCC\OPC\UAServer\PKI\CA\certs\" -Force
```

En esta prueba `rejected` estuvo vacío; el fallo de OPC Expert UA no fue ese. En UaExpert el paso equivalente es **TRUST SERVER CERTIFICATE**.

### 5.9 Limpieza de temporales (disco justo antes del setup)

```powershell
Remove-Item "$env:TEMP\*" -Recurse -Force -ErrorAction SilentlyContinue
```

También: `%TEMP%` en el Explorador, `C:\Windows\Temp` (admin), Liberador `cleanmgr`. **No** borrar `C:\Program Files (x86)\Siemens\` ni datos SQL `WINCC`.

### 5.10 Comprobación rápida “¿está vivo UA?”

```powershell
Get-Service OpcUaServerWinCCPro
netstat -ano | findstr "4861"
```

Salida buena (ejemplo real):

```
Running  OpcUaServerWinCCPro
TCP    0.0.0.0:4861    0.0.0.0:0    LISTENING    268
TCP    [::]:4861       [::]:0       LISTENING    268
```

---

## 6. Conexión de clientes

### OPC DA (validado con OPC Expert)

- Servidor: **`OPCServer.WinCC_SCADA.1`**
- Runtime debe estar en ejecución.
- El `.1` es la versión COM del mismo ProgID `OPCServer.WinCC_SCADA`.
- **No** es OPC UA.

### OPC UA (validado con UaExpert)

- URL: `opc.tcp://localhost:4861` o `opc.tcp://DESKTOP-KC6QA5V:4861`
- Elegir endpoint **None / None** (prueba).
- Primera vez: **BadCertificateUntrusted** → **TRUST SERVER CERTIFICATE**.
- Usuario: Anonymous o cuenta Windows. Con UaExpert alcanzó Trust + None.
- OPC Expert UA en esta PC falló con *using given credentials* (Clear de credenciales no alcanzó). Usar UaExpert / Optix / Ignition.

Optix / Ignition: mismo endpoint, IP de la PC Station, puerto **4861**, confiar el certificado. En planta no dejar None.

---

## 7. Rutas y archivos importantes

| Ruta | Uso |
|---|---|
| `...\SCADA-RT_V11\WinCC\OPC\UAServer\` | Motor UA |
| `...\UAServer\OpcUaServerWinCCPro.exe` | Servicio |
| `...\UAServer\OpcUaServerWinCCPro.xml` | Puerto, seguridad, trace |
| `...\UAServer\PKI\CA\certs\` | Certificado del servidor (`.der`) |
| `...\UAServer\PKI\CA\private\` | Clave (`.pfx`) |
| `...\UAServer\PKI\CA\rejected\` | Certs de clientes rechazados |
| `C:\Users\Public\Documents\Siemens\WinCC\OPC\UAServer\OpcUaServerWinCCPro.txt` | Log de trace |
| `...\WinCC\OPC\UAClient\` | Cliente UA de WinCC (no es el servidor) |
| `...\WinCC\OPC\DataAccess\` | Servidor OPC DA |

---

## 8. Licencias

- Necesaria: **WinCC Runtime Professional** (Used con el Runtime activo).
- No usar: **OPC UA Server Process Historian**.
- Connectivity Pack es típico de WinCC V7 clásico; la documentación TIA de RT Professional cita la licencia de Runtime Professional para los servidores OPC.

---

## 9. Cómo repetirlo en la segunda computadora

1. No buscar el servidor UA dentro de TIA. Usar el ISO **WinCC Runtime Professional**.
2. Instalar desde **carpeta local**, marcar **WinCC OPC UA Server**.
3. Mismo Update que TIA.
4. Comprobar `UAServer` + `Get-Service OpcUaServerWinCCPro` **antes** de pelear con clientes.
5. Si 0x80004005: activar trace, leer el txt, **regenerar certificado** (mover `.der`/`.pfx` viejos).
6. Probar UA con **UaExpert** + Trust. DA con OPC Expert y `OPCServer.WinCC_SCADA.1`.
7. Documentar hostname, IP, puerto 4861 y usuario Windows de esa estación.
