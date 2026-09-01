# Software instalado — TIA Portal V18

Inventario exportado desde TIA Portal en la **estación de laboratorio** (referencia de ingeniería). Sirve para documentar con qué versión se validó el servidor OPC de WinCC Runtime Professional.

Los paquetes HSP / Motion / PID y el detalle de componentes internos del instalador se omiten: no aplican a esta PC Station.

## Entorno

| Campo | Valor |
|---|---|
| Equipo | `DESKTOP-KC6QA5V` |
| Usuario | `DESKTOP-KC6QA5V\Gigabyte-PC` |
| Sistema | Windows 10 Pro (`10.0.19045`) |
| Ruta TIA Portal | `C:\Program Files\Siemens\Automation\Portal V18` |

## Paquetes TIA (lo que cuenta)

| Producto | Versión |
|---|---|
| Totally Integrated Automation Portal | V18 Update 5 |
| STEP 7 Professional | V18 Update 5 |
| WinCC Professional (Engineering) | V18 Update 5 |

## Productos SIMATIC

| Nombre | Versión |
|---|---|
| Automation License Manager | V6.2 + Upd3 |
| S7-PCT | V3.5 SP3 Upd5 |
| S7-PLCSIM | V5.4 + SP8 + Upd2 / V18 SP2 |
| S7-PLCSIM Advanced | V5.0 Upd2 |
| SIMATIC NET PC Software | V18.0 |
| SIMATIC NET PC Software Doc | V18.0 |
| SIMATIC NET SOFTNET-IE RNA | V17.0 |
| SIMATIC OPC-XML-Gateway | V18.0 Upd5 |
| SIMATIC ProSave | V18.0 Upd5 |
| SIMATIC S7-Block Privacy | V1.0 + SP6 |
| SIMATIC S7-SCL | V5.7 |
| SIMATIC S7-Web2PLC | V1.0 + SP5 |
| SIMATIC STEP 7 Prof - STEP 7 Safety - WinCC Prof | V18.0 Upd5 |
| SIMATIC WinCC Runtime Advanced Simulation | V17.0 Upd8 |
| SIMATIC WinCC Runtime Professional Simulation | V18.0 Upd5 |
| SIMATIC WinCC/Audit Viewer | 2008 SP2 |
| STEP 7 | V5.7 + SP2 |
| TIA Administrator | V2.0 |
| TIA Administrator Update | V3.0.2.0 |
| TIA Portal Project Server | V1.1 |
| UMC Status Application | V2.10 |
| User Management Component | V2.13 Sp1 Upd0 |

## Componentes relevantes para OPC / Runtime

En el inventario de **SIMATIC Components** los ítems que importan para este proyecto:

| Componente | Versión | Nota |
|---|---|---|
| Totally Integrated Automation Portal V18 (TIAP18) | V18.0 SP1 UPD5 | Engineering |
| WinCC Runtime Professional V18 — SIMATIC WinCC Runtime | V18.0 Upd5 | Runtime de la PC Station |
| WinCC Runtime Professional V18 — OPCUA_Client | V1.1 + SP3 + Upd1 | **Cliente** UA de WinCC, no el servidor |
| WinCC Runtime Advanced V17.0 | V17.0 UPD8 | Presente en la VM; **no** es el Runtime de esta PC Station |
| SIEMENS OPC | 3.9 | Stack OPC genérico |
| OPC-XML-Gateway | 18.0.1.5 | No es el camino hacia Ignition |

El **servidor OPC UA** de planta (`OpcUaServerWinCCPro`) no aparece como producto TIA: se instala desde el ISO **WinCC Runtime Professional**, casilla Tools → WinCC OPC UA Server.
