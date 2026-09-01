# Variables prioritarias — centrífuga 1

Catálogo de vinculación OPC UA / Ignition para la máquina 1.

| Campo | Valor |
|---|---|
| Conexión WinCC | `HMI_Connection_1` |
| PLC | `PLC_1_2420` |
| Prefijo | `1…` |

Para centrífugas 2 y 3 cambiar el primer dígito (`2…` / `3…`). Listado completo: [`listado-completo-variables.md`](listado-completo-variables.md).

## Estado (bits)

| Tag | Tipo | Dirección | Significado | Nota |
|---|---|---|---|---|
| `1B01_ANY_FAULT` | Bool | `%M40.4` | Existencia de alguna falla |  |
| `1B02_AUTO_MODE_ON` | Bool | `%M62.7` | Funcionamiento en automático |  |
| `1B03_MAN_MODE_ON` | Bool | `%M63.0` | Funcionamiento en manual |  |
| `1B04_CHARGE_MODE_ON` | Bool | `%M63.3` | Modo de carga activo |  |
| `1B05_SPIN_MODE_ON` | Bool | `%M68.3` | Centrifugado |  |
| `1B06_DCH_MODE_ON` | Bool | `%M63.1` | Descarga |  |
| `1B07_MAIN_GATE_CLOSED` | Bool | `%M105.6` | Compuerta principal cerrada |  |
| `1B08_MAIN_GATE_LEVEL` | Bool | `%M105.7` | Nivel de compuerta principal |  |
| `1B09_WASH_STEAM` | Bool | `%DB1.DBX123.1` | Lavado con vapor |  |
| `1B10_WASH_WATER_OK` | Bool | `%DB1.DBX125.2` | Lavado (agua) OK |  |
| `1B11_FEEDER_A` | Bool | `%DB1.DBX123.2` | Alimentador A |  |
| `1B12_FEEDER_B` | Bool | `%DB1.DBX123.3` | Alimentador B |  |
| `1B13_DCH_BLOCKED` | Bool | `%DB1.DBX123.0` | Descarga bloqueada |  |
| `1B14_DCH_HOME_UPPER` | Bool | `%DB1.DBX124.4` | Descarga en home superior |  |
| `1B15_MECH_BRAKE` | Bool | `%DB1.DBX123.6` | Freno mecánico |  |

## Ciclo y proceso (enteros)

| Tag | Tipo | Dirección | Significado | Nota |
|---|---|---|---|---|
| `1C01_REF_VELOC` | Int | `%DB1.DBW328` | RPM de referencia |  |
| `1C02_ACT_VELOC` | Int | `%DB1.DBW330` | RPM (promedio en el ciclo) |  |
| `1C03_PIW_VFABB_DIV10` | Int | `%DB150.DBW4` | % de carga promedio en el ciclo (÷10) | Dividir por 10 |
| `1C04_NIV_VIBRACION` | Int | `%DB1.DBW334` | Nivel de vibración |  |
| `1C05_NIV_COMPUERTA` | Int | `%DB1.DBW336` | Nivel de compuerta |  |
| `1C06_CY_CHARGE_TIME` | Int | `%DB1.DBW354` | Tiempo de carga | % respecto de CY_TOTAL_TIME |
| `1C07_CY_ACCEL_TIME` | Int | `%DB1.DBW358` | Tiempo de aceleración | % respecto de CY_TOTAL_TIME |
| `1C08_CY_SPIN_TIME` | Int | `%DB1.DBW362` | Tiempo de centrifugado | % respecto de CY_TOTAL_TIME |
| `1C09_CY_DECCEL_TIME` | Int | `%DB1.DBW366` | Tiempo de desaceleración | % respecto de CY_TOTAL_TIME |
| `1C10_CY_DCH_TIME` | Int | `%DB1.DBW370` | Tiempo de descarga | % respecto de CY_TOTAL_TIME |
| `1C11_CY_WASH_TIME` | Int | `%DB1.DBW374` | Tiempo de lavado | % respecto de CY_TOTAL_TIME |
| `1C12_CY_WAIT_TIME` | Int | `%DB1.DBW378` | Espera por sincronismos | % respecto de CY_TOTAL_TIME |
| `1C13_CY_TOTAL_TIME` | Int | `%DB1.DBW382` | Tiempo total de ciclo |  |
| `1C14_CY_NRO_COUNT` | Int | `%DB1.DBW390` | Número de ciclos en automático |  |
| `1C15_CY_HOUR` | Int | `%DB1.DBW388` | Ciclos por hora (÷10) | Dividir por 10 |
| `1C16_CY_CH_START_SPEED` | Int | `%DB1.DBW384` | Velocidad al inicio de carga |  |
| `1C17_CY_CH_END_SPEED` | Int | `%DB1.DBW386` | Velocidad al fin de carga |  |
| `1C18_TOTAL_AZUC` | Int | `%DB1.DBW758` | Total azúcar (acumulado) |  |
| `1C19_TOTAL_MCOC` | Int | `%DB1.DBW760` | Total MCOC (acumulado) |  |

## Alarmas / fallas testigo

| Tag | Tipo | Dirección | Significado | Nota |
|---|---|---|---|---|
| `1F01_FALLA_TESTIGO` | Bool | `%DB1.DBX271.0` | ALR 01 — Aire comprimido |  |
| `1F02_FALLA_TESTIGO` | Bool | `%DB1.DBX271.1` | ALR 02 — P.E. veloc. exces. |  |
| `1F03_FALLA_TESTIGO` | Bool | `%DB1.DBX271.2` | ALR 03 — Comp. princ. nivel |  |
| `1F04_FALLA_TESTIGO` | Bool | `%DB1.DBX271.3` | ALR 04 — Inversor frec. |  |
| `1F05_FALLA_TESTIGO` | Bool | `%DB1.DBX271.4` | ALR 05 — Lavado pres. baja |  |
| `1F06_FALLA_TESTIGO` | Bool | `%DB1.DBX271.5` | ALR 06 — Lavado posición |  |
| `1F07_FALLA_TESTIGO` | Bool | `%DB1.DBX271.6` | ALR 07 — Indicador carga |  |
| `1F08_FALLA_TESTIGO` | Bool | `%DB1.DBX271.7` | ALR 08 — Nivel bajo mezcl. |  |
| `1F09_FALLA_TESTIGO` | Bool | `%DB1.DBX270.0` | ALR 09 — Compuerta princ. |  |
| `1F10_FALLA_TESTIGO` | Bool | `%DB1.DBX270.1` | ALR 10 — Tiempo de carga |  |
| `1F11_FALLA_TESTIGO` | Bool | `%DB1.DBX270.2` | ALR 11 — Compuerta goteo |  |
| `1F12_FALLA_TESTIGO` | Bool | `%DB1.DBX270.3` | ALR 12 — Sep. miel blanca |  |
| `1F13_FALLA_TESTIGO` | Bool | `%DB1.DBX270.4` | ALR 13 — Sep. miel verde |  |
| `1F14_FALLA_TESTIGO` | Bool | `%DB1.DBX270.5` | ALR 14 — Oscilación |  |
| `1F15_FALLA_TESTIGO` | Bool | `%DB1.DBX270.6` | ALR 15 — Vibración |  |
| `1F16_FALLA_TESTIGO` | Bool | `%DB1.DBX270.7` | ALR 16 — Velocidad |  |
| `1F17_FALLA_TESTIGO` | Bool | `%DB1.DBX273.0` | ALR 17 — Salida de azúcar |  |
| `1F18_FALLA_TESTIGO` | Bool | `%DB1.DBX273.1` | ALR 18 — Cinta transporte |  |
| `1F19_FALLA_TESTIGO` | Bool | `%DB1.DBX273.2` | ALR 19 — Tapa de fondo |  |
| `1F20_FALLA_TESTIGO` | Bool | `%DB1.DBX273.3` | ALR 20 — Dados |  |
| `1F21_FALLA_TESTIGO` | Bool | `%DB1.DBX273.4` | ALR 21 — Descarg. posic. |  |
| `1F22_FALLA_TESTIGO` | Bool | `%DB1.DBX273.5` | ALR 22 — Descarg. vertical |  |
| `1F23_FALLA_TESTIGO` | Bool | `%DB1.DBX273.6` | ALR 23 — Freno activo |  |
| `1F24_FALLA_TESTIGO` | Bool | `%DB1.DBX273.7` | ALR 24 — Parámetros |  |

## Parámetros / consignas

| Tag | Tipo | Dirección | Significado | Nota |
|---|---|---|---|---|
| `1P01_CHARGE_SPEED` | Int | `%DB1.DBW6` | Velocidad de carga |  |
| `1P02_CHARGE_ACCEL` | Int | `%DB1.DBW8` | Aceleración de carga |  |
| `1P03_AMED_CHARGE_TIME` | Int | `%DB1.DBW10` | Tiempo de carga (amedido) |  |
| `1P04_CHARGE_THICKNESS` | Int | `%DB1.DBW12` | Espesor de carga |  |
| `1P05_FIXED_CHARGE_TIME` | Int | `%DB1.DBW14` | Tiempo de carga fijo |  |
| `1P06_SYRUP_GREEN` | Int | `%DB1.DBW16` | Miel verde |  |
| `1P07_WASH_1_START` | Int | `%DB1.DBW18` | Inicio lavado 1 |  |
| `1P08_WASH_1_TIME` | Int | `%DB1.DBW20` | Tiempo lavado 1 |  |
| `1P09_SYRUP_WHITE` | Int | `%DB1.DBW22` | Miel blanca |  |
| `1P10_DELAY_ST_WASH_2` | Int | `%DB1.DBW24` | Retardo inicio lavado 2 |  |
| `1P11_WASH_2_TIME` | Int | `%DB1.DBW26` | Tiempo lavado 2 |  |
| `1P12_START_STEAM_WASH` | Int | `%DB1.DBW28` | Inicio lavado con vapor |  |
| `1P13_TIME_STEAM_WASH` | Int | `%DB1.DBW30` | Tiempo lavado con vapor |  |
| `1P14_SPIN_SPEED` | Int | `%DB1.DBW32` | Velocidad de centrifugado |  |
| `1P15_SPIN_TIME` | Int | `%DB1.DBW34` | Tiempo de centrifugado |  |
| `1P16_MAX_VIBRATION` | Int | `%DB1.DBW36` | Vibración máxima |  |
| `1P17_PLOUGH_WASH_TIME` | Int | `%DB1.DBW38` | Tiempo lavado de arado |  |
| `1P18_PLOUGH_BOTTON_TIME` | Int | `%DB1.DBW40` | Tiempo arado / fondo |  |
| `1P19_AIR_BOTTON_CONE` | Int | `%DB1.DBW42` | Aire cono de fondo |  |
| `1P20_ACCELERATION` | Int | `%DB1.DBW44` | Aceleración |  |
| `1P21_DECELERATION` | Int | `%DB1.DBW46` | Desaceleración |  |
| `1P22_MIN_SEQ_TIME` | Int | `%DB1.DBW48` | Tiempo mínimo de secuencia |  |
| `1P23_STEAM_CLEAN` | Int | `%DB1.DBW50` | Limpieza con vapor |  |
| `1P24_WASH_SUGAR_OUTLET` | Int | `%DB1.DBW52` | Lavado salida de azúcar |  |
| `1P25_START_WASH_INLET` | Int | `%DB1.DBW54` | Inicio lavado de entrada |  |
| `1P26_TIME_WASH_INLET` | Int | `%DB1.DBW56` | Tiempo lavado de entrada |  |
| `1P27_AIR_DIST_CONE` | Int | `%DB1.DBW58` | Aire cono de distribución |  |
| `1P28_DCH_MID_STOP` | Int | `%DB1.DBW60` | Parada intermedia de descarga |  |
| `1P29_MAN_STOP_RAMP_SP` | Int | `%DB1.DBW62` | Rampa de parada manual (velocidad) |  |
| `1P30_MAN_STOP_RAMP_TI` | Int | `%DB1.DBW64` | Rampa de parada manual (tiempo) |  |
| `1P31_WASH_MEDIA_P01` | Int | `%DB1.DBW2` | Medio de lavado P01 |  |
| `1P32_WASH_P02` | Int | `%DB1.DBW4` | Lavado P02 |  |

