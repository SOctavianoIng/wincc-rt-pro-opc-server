# Informe técnico — Comunicación OPC UA con servidor WinCC Runtime Professional

**Entregable para el usuario final / desarrollo SCADA Ignition**  
**Alcance:** centrifugas azucareras automáticas de crudo — red MPI RS-485 con tres PLC Siemens S7-315-2DP  
**Fecha de cierre de commissioning:** agosto 2026  

| Documento | Uso |
|---|---|
| Este informe | Entregable operativo: arquitectura, conexión y puesta en marcha validadas |
| Sección 2 (variables) | Catálogo de tags para el desarrollador Ignition — *a completar* |

---

## 1. Introducción

Se implementó y validó la exposición de variables de proceso desde **WinCC Runtime Professional V18** (PC Station) hacia un cliente **OPC UA**, con destino el desarrollo del SCADA en **Ignition** sobre la PC de supervisión de planta.

Los PLC **S7-315-2DP** se comunican con WinCC por **MPI (RS-485)**; no disponen de Ethernet de proceso hacia el SCADA. El cliente OPC **no habla con el PLC**: lee tags del Runtime WinCC, que a su vez mantiene la comunicación MPI.

El trabajo cubrió: instalación y puesta a punto del **servidor OPC UA** de WinCC (`OpcUaServerWinCCPro`), verificación en local y por LAN, y **commissioning** del cliente OPC UA de Ignition hasta dejar la conexión operativa. A partir de este punto, el desarrollo de pantallas, historiales y lógica de Ignition puede realizarse sobre los datos servidos por ese endpoint.

---

## 2. Variables de PLC

*(Sección reservada. Se completará manualmente con el listado de variables: nombre, PLC de origen, significado y notas para el desarrollador Ignition.)*

---

## 3. Arquitectura de la solución

![Arquitectura de comunicación OPC UA — centrifugas de crudo: PLC S7-315-2DP (MPI RS-485) → WinCC Runtime Professional (servidor OPC UA) → Ignition SCADA](arquitectura-opc-ua-wincc-ignition.png)

```text
PLC S7-315-2DP (×3)  --MPI RS-485-->  WinCC Runtime Professional (PC Station)
                                         |
                                         +-- OPC UA  opc.tcp://<IP-WinCC>:4861
                                                     servicio Windows: OpcUaServerWinCCPro
                                                              |
                                                              v  (LAN Ethernet)
                                                     Ignition SCADA (PC de planta)
                                                     Cliente OPC UA nativo
```

| Elemento | Descripción |
|---|---|
| Campo / proceso | Centrifugas azucareras automáticas de crudo |
| Controladores | Tres PLC Siemens **S7-315-2DP** en red **MPI RS-485** |
| Capa HMI / Runtime | WinCC Runtime Professional V18 (PC Station) |
| Servidor de datos hacia SCADA | **OPC UA** nativo de WinCC RT Professional |
| Cliente SCADA | **Ignition** (conexión directa; no requiere KEPServer ni UaExpert en producción) |
| Canal opcional (pruebas) | OPC DA `OPCServer.WinCC_SCADA.1` — no es el camino recomendado para Ignition |

**Estación WinCC de planta (referencia validada):**

| Parámetro | Valor |
|---|---|
| Hostname | `DESKTOP-BH0RBSV` |
| Usuario Windows | `desktop-bh0rbsv\siemens` |
| Dirección IP | `192.168.0.221` |
| Puerto OPC UA | **4861** |
| Servicio Windows | `OpcUaServerWinCCPro` (inicio Automatic) |

---

## 4. Resultado del trabajo (qué quedó operativo)

| Ítem | Estado |
|---|---|
| Componente **WinCC OPC UA Server** instalado (ISO Runtime Professional → Tools) | Operativo |
| Servicio `OpcUaServerWinCCPro` | Running; escucha en `0.0.0.0:4861` |
| Lectura local de tags (UaExpert) | Validada (Trust del certificado del servidor) |
| Acceso por LAN (`Test-NetConnection` puerto 4861) | Validado |
| Cliente remoto UaExpert | Validado |
| Cliente **Ignition OPC UA** desde PC SCADA | Validado (ver §6 — Host Override) |
| OPC DA Professional (`OPCServer.WinCC_SCADA.1`) | Disponible como canal paralelo de prueba |

Ignition se conecta **directamente** al servidor OPC UA de WinCC. Herramientas como UaExpert o KEPServer se usaron solo para diagnóstico y prueba, no son intermediarios obligatorios en producción.

---

## 5. Datos de conexión para desarrollo en Ignition

### 5.1 Parámetros del cliente OPC UA

| Campo en Ignition | Valor a configurar |
|---|---|
| **Endpoint URL** | `opc.tcp://192.168.0.221:4861` |
| **Endpoint Host Override** (opciones avanzadas) | `opc.tcp://192.168.0.221:4861` |
| Security Policy / Mode (prueba) | **None / None** si el endpoint lo permite |
| Security (recomendado en planta) | **Sign** o **SignAndEncrypt** (p. ej. Basic256Sha256) |
| Autenticación | **Anonymous** si está habilitado en WinCC; si no, Username `desktop-bh0rbsv\siemens` + contraseña de esa cuenta Windows |
| Certificado | Confiar el certificado del servidor WinCC (Rejected → Trusted) |

> **Importante — Host Override:** debe usarse la **URL completa** (`opc.tcp://…`), igual que en Endpoint URL. Completar solo la IP (`192.168.0.221`) provoca fallo de conexión aunque el puerto 4861 responda en red.

| Incorrecto | Correcto |
|---|---|
| `192.168.0.221` | `opc.tcp://192.168.0.221:4861` |

### 5.2 Condiciones previas en la PC WinCC

1. Proyecto HMI / Runtime Professional en ejecución (tags MPI con calidad válida cuando los PLC están en línea).
2. Servicio `OpcUaServerWinCCPro` en estado **Running**.
3. Puerto **4861** en LISTENING (`netstat`).
4. Licencia **WinCC Runtime Professional** activa (no aplica la licencia “OPC UA Server Process Historian”).

Comprobación rápida (PowerShell en PC WinCC):

```powershell
Get-Service OpcUaServerWinCCPro
netstat -ano | findstr "4861"
```

Desde la PC de Ignition:

```powershell
Test-NetConnection -ComputerName 192.168.0.221 -Port 4861
```

`TcpTestSucceeded : True` confirma red y puerto; no sustituye la configuración correcta del cliente OPC UA ni el Trust del certificado.

---

## 6. Commissioning — hitos y resolución

### 6.1 Secuencia de puesta en marcha (resumen)

1. Verificación del motor OPC UA en la PC WinCC (servicio + puerto 4861).
2. Prueba de conectividad TCP desde la red de planta / PC SCADA.
3. Validación con **UaExpert** (Custom Discovery → endpoint → Trust certificate).
4. Configuración del **cliente OPC UA de Ignition** hacia `192.168.0.221:4861`.
5. Corrección de parámetros avanzados hasta conexión estable (Host Override).

### 6.2 Hallazgos relevantes durante el commissioning

| Situación | Lectura / acción |
|---|---|
| `Test-NetConnection` OK pero Ignition en Failed | No es (solo) firewall; revisar Endpoint, Host Override, seguridad y certificados |
| `password not configured` | Modo Username sin contraseña cargada en Ignition |
| `Bad_NonceInvalid` (nonce &lt; 32 bytes) | Combinación Username + seguridad None con el stack de Ignition; preferir Anonymous o canal Sign/SignAndEncrypt; verificar también Host Override |
| Certificado no confiado | Primera conexión: Trust del cert autofirmado de WinCC en Ignition |
| Carpeta PKI `rejected` ausente en WinCC | Puede crearse; en este commissioning no fue la causa del fallo de Ignition |

### 6.3 Resolución que dejó Ignition operativo

Con LAN y UaExpert ya validados, la comunicación Ignition ↔ WinCC OPC UA quedó operativa al completar **Endpoint Host Override** con:

```text
opc.tcp://192.168.0.221:4861
```

y no únicamente con la dirección IP.

---

## 7. Notas técnicas para operación y soporte

### 7.1 Qué no confundir

| Concepto | Aclaración |
|---|---|
| Pantalla OPC UA en TIA (Runtime settings) | Configura el proyecto; **no instala** el servidor |
| `UAClient` / paquete OPCUA_Client | WinCC como **cliente** UA; el servidor es carpeta `UAServer` + `OpcUaServerWinCCPro.exe` |
| ProgID `OPC.SimaticHMI.CoRtHmiRTm.1` | OPC DA de Runtime **Advanced**; no usar en esta PC Station Professional |
| OPC DA vs OPC UA | DA no usa el puerto 4861; Ignition debe usar **OPC UA** |
| ISO TIA Portal vs ISO Runtime Professional | El servidor UA se instala desde el ISO **WinCC Runtime Professional** (Tools → WinCC OPC UA Server) |

### 7.2 Si el servicio UA no arranca (referencia)

En la estación de laboratorio se observó fallo `0x80004005` / evento 7023 por certificado PKI inválido tras una instalación incompleta. El arreglo fue regenerar certificados (mover `.der`/`.pfx` viejos y reiniciar el servicio). Detalle operativo interno: ver documentación de soporte del repositorio (`OPC-UA-WinCC-RT-Professional-V18.md`, `OPC-PowerShell-Diagnostico.md`).

### 7.3 Seguridad en planta

- La política **None** es adecuada para puesta en marcha y pruebas.
- En operación se recomienda pasar a **Sign** / **SignAndEncrypt** y gestionar confianza de certificados en Ignition y, si aplica, en el almacén PKI de WinCC.
- No documentar contraseñas de cuentas Windows en repositorios o informes versionados; usar el almacén de credenciales de Ignition.

---

## 8. Alcance entregado y trabajo pendiente del desarrollador Ignition

**Entregado / validado**

- Servidor OPC UA WinCC RT Professional accesible en LAN.
- Parámetros de conexión para el cliente Ignition.
- Prueba de lectura de tags vía clientes OPC UA de referencia e Ignition.

**A cargo del desarrollo SCADA (Ignition)**

- Completar y usar el catálogo de la **sección 2** (variables por PLC).
- Mapear tags OPC UA a UDTs / tags de Ignition.
- Diseñar pantallas, alarmas, historial y lógica de centrifugas.
- Ajustar seguridad OPC UA de producción (certificados y políticas).

---

## 9. Referencias internas del proyecto

| Archivo | Contenido |
|---|---|
| `OPC-UA-WinCC-RT-Professional-V18.md` | Informe técnico de implementación del servidor (laboratorio y procedimiento) |
| `Chat-Resumen-OPC.md` | Hilo condensado; §21 estación definitiva; §22 commissioning Ignition |
| `OPC-PowerShell-Diagnostico.md` | Comandos de diagnóstico en orden cronológico |

---

*Fin del informe técnico entregable — Comunicación OPC UA WinCC Runtime Professional → Ignition SCADA.*
