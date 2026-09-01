# Informe de variables — centrífugas de crudo

Notas operativas para leer tags de WinCC Runtime Professional por OPC UA (destino Ignition).

| Documento | Uso |
|---|---|
| Este archivo | Arranque del Runtime, variables internas de WinCC y reglas de interpretación |
| [`tabla-variables-centrifuga-1.md`](tabla-variables-centrifuga-1.md) | Catálogo prioritario de tags PLC (máquina 1, con direcciones y notas) |
| [`listado-completo-variables.md`](listado-completo-variables.md) | Default tag table completa (tres centrífugas) |

Hay dos familias de tags:

| Familia | Origen | Uso |
|---|---|---|
| **Vinculadas al PLC** | `HMI_Connection_n` → S7-315-2DP | Prioritarias: estado, ciclo, alarmas, consignas |
| **Internas de WinCC** | Cálculos / scripts del Runtime (Pantalla Principal) | KPIs de la hora en curso; **no** sustituyen la lectura de PLC |

---

## 1. Condiciones para leer variables

El cliente OPC UA ve calidad válida cuando:

1. El Runtime WinCC está **en ejecución** (estación Running).
2. La PC Station está **conectada a MPI RS-485** y lee los tres PLC.
3. El servicio `OpcUaServerWinCCPro` está **Running**.

Sin Runtime o sin MPI el endpoint puede responder y el árbol aparecer, pero los valores de PLC suelen ir con calidad mala. Eso no es fallo de OPC UA: WinCC no tiene dato de campo.

El SCADA WinCC arranca como servicio de Windows, **sin intervención del operador**. Inicialización completa: **unos 5 minutos**. Software: SIMATIC WinCC Runtime Professional. Proyecto: **Runtime Station_2**.

Si no arrancó solo o se detuvo a mano: acceso directo de WinCC Runtime en el escritorio → verificar ruta del proyecto → **Run** (icono verde).

Detalle de pantallas del HMI: manual operativo de Indicadores de Producción (Producción e Instrumentos).

---

## 2. Convención `HMI_Connection_n`

| Centrífuga | Conexión | PLC | Prefijo |
|---|---|---|---|
| 1 | `HMI_Connection_1` | `PLC_1_2420` | `1…` |
| 2 | `HMI_Connection_2` | `PLC_2_2419` | `2…` |
| 3 | `HMI_Connection_3` | `PLC_3_2650` | `3…` |

Prefijos: `nB` bits de estado · `nC` ciclo/proceso · `nF` alarmas · `nP` consignas.

---

## 3. Variables internas del SCADA WinCC

Cálculos intermedios de la **Pantalla Principal de Producción**. En Ignition hay que distinguirlos de los tags de PLC.

### Estado operativo (por máquina)

| Magnitud | Contenido |
|---|---|
| Etapa del ciclo | 1 = Carga · 2 = Centrifugado · 3 = Descarga |
| Modo | Automático / Manual |
| Fallas activas | Cantidad y tipo |
| Velocidad | RPM (preferir `nC02_ACT_VELOC` como fuente PLC) |
| Carga del motor | % (preferir `nC03_PIW_VFABB_DIV10 / 10`) |
| Ciclo en curso | Encendido / apagado |

### Rendimiento

- **Ciclos completados:** solo ciclos correctos, en automático y sin fallas. Un ciclo válido va de carga a descarga cuando se abre la compuerta, las RPM bajan al mínimo y la carga del motor vuelve a máquina sin carga. En PLC: `nC14_CY_NRO_COUNT`.
- **Ciclos por hora:** cálculo del fabricante en el S7. En PLC: `nC15_CY_HOUR` (**dividir por 10**).
- **Fase en curso:** 1 a 3.

### Indicadores de ciclo

| Indicador | Significado |
|---|---|
| En ciclo | Ciclo de producción en curso |
| Fallo | Falla que detiene la máquina |
| Lavado | Sub-etapa (canasto, lavado 1 o lavado 2) |

### Tiempos de la hora en curso (minutos enteros)

| Magnitud | Significado |
|---|---|
| Minutos de ciclo | Tiempo ejecutando ciclo de producción |
| Minutos inactivos | Detenida por operador o en espera del próximo ciclo |
| Minutos en lavado | Ya **incluidos** en minutos de ciclo: no sumarlos aparte |
| Minutos en falla | Tiempo detenido por una falla |

Resolución de **minuto entero**; tendencias cada **un minuto**. Pueden diferir de los valores instantáneos del PLC: no interpretarlo como pérdida de OPC UA.

Estos minutos **no reemplazan** los tiempos de etapa `nC06` … `nC13`.

---

## 4. Variables vinculadas al PLC — reglas de uso

Listado con direcciones: [`tabla-variables-centrifuga-1.md`](tabla-variables-centrifuga-1.md).

**Tiempos de ciclo** (carga, aceleración, centrifugado, desaceleración, aceleración, descarga, lavado, espera por sincronismos): convertir cada etapa a **porcentaje de** `nC13_CY_TOTAL_TIME`.

**Escalas**

| Tag | Tratamiento |
|---|---|
| `nC15_CY_HOUR` | Dividir por 10 |
| `nC03_PIW_VFABB_DIV10` | Dividir por 10 |
| `nC02_ACT_VELOC` | RPM entero, sin escala |

**Estado de máquina:** combinar bits (`nB01` … `nB06`, lavados) para saber si funciona o está parada por falla. En Ignition generar un tag interno **máquina parada** para logging de tiempo de parada:

- Parada por falla: `nB01_ANY_FAULT` = 1
- En funcionamiento: automático (`nB02_AUTO_MODE_ON`) y alguna etapa de ciclo, con `ANY_FAULT` = 0
- Máquina parada: falla activa, o automático apagado / ninguna etapa activa

**Alarmas:** `nF01` … `nF24` = ALR 01 … 24 (mismo orden que el listado HMI). Ver [`tabla-variables-centrifuga-1.md`](tabla-variables-centrifuga-1.md) y `imagenes/alarmas-en-orden.png`.
