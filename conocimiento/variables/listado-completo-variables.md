# Listado completo de variables — Runtime PC Station

Exportación de la **Default tag table** de WinCC Runtime Professional (TIA Portal V18). Tags de proceso: **270** (90 por centrífuga).

| Centrífuga | Conexión | PLC | Prefijo |
|---|---|---|---|
| 1 | `HMI_Connection_1` | `PLC_1_2420` | `1…` |
| 2 | `HMI_Connection_2` | `PLC_2_2419` | `2…` |
| 3 | `HMI_Connection_3` | `PLC_3_2650` | `3…` |

El resto del identificador es idéntico entre máquinas. Catálogo prioritario (centrífuga 1, con notas de uso): [`tabla-variables-centrifuga-1.md`](tabla-variables-centrifuga-1.md). Semántica operativa: [`Informe-variables.md`](Informe-variables.md).

## Centrífuga 1 — `HMI_Connection_1` / `PLC_1_2420`

### Estado (bits) (`1B…`)

| Tag | Tipo | Dirección | Significado |
|---|---|---|---|
| `1B01_ANY_FAULT` | Bool | `%M40.4` | Existencia de alguna falla |
| `1B02_AUTO_MODE_ON` | Bool | `%M62.7` | Funcionamiento en automático |
| `1B03_MAN_MODE_ON` | Bool | `%M63.0` | Funcionamiento en manual |
| `1B04_CHARGE_MODE_ON` | Bool | `%M63.3` | Modo de carga activo |
| `1B05_SPIN_MODE_ON` | Bool | `%M68.3` | Centrifugado |
| `1B06_DCH_MODE_ON` | Bool | `%M63.1` | Descarga |
| `1B07_MAIN_GATE_CLOSED` | Bool | `%M105.6` | Compuerta principal cerrada |
| `1B08_MAIN_GATE_LEVEL` | Bool | `%M105.7` | Nivel de compuerta principal |
| `1B09_WASH_STEAM` | Bool | `%DB1.DBX123.1` | Lavado con vapor |
| `1B10_WASH_WATER_OK` | Bool | `%DB1.DBX125.2` | Lavado (agua) OK |
| `1B11_FEEDER_A` | Bool | `%DB1.DBX123.2` | Alimentador A |
| `1B12_FEEDER_B` | Bool | `%DB1.DBX123.3` | Alimentador B |
| `1B13_DCH_BLOCKED` | Bool | `%DB1.DBX123.0` | Descarga bloqueada |
| `1B14_DCH_HOME_UPPER` | Bool | `%DB1.DBX124.4` | Descarga en home superior |
| `1B15_MECH_BRAKE` | Bool | `%DB1.DBX123.6` | Freno mecánico |

### Ciclo y proceso (enteros) (`1C…`)

| Tag | Tipo | Dirección | Significado |
|---|---|---|---|
| `1C01_REF_VELOC` | Int | `%DB1.DBW328` | RPM de referencia |
| `1C02_ACT_VELOC` | Int | `%DB1.DBW330` | RPM (promedio en el ciclo) |
| `1C03_PIW_VFABB_DIV10` | Int | `%DB150.DBW4` | % de carga promedio en el ciclo (÷10) |
| `1C04_NIV_VIBRACION` | Int | `%DB1.DBW334` | Nivel de vibración |
| `1C05_NIV_COMPUERTA` | Int | `%DB1.DBW336` | Nivel de compuerta |
| `1C06_CY_CHARGE_TIME` | Int | `%DB1.DBW354` | Tiempo de carga |
| `1C07_CY_ACCEL_TIME` | Int | `%DB1.DBW358` | Tiempo de aceleración |
| `1C08_CY_SPIN_TIME` | Int | `%DB1.DBW362` | Tiempo de centrifugado |
| `1C09_CY_DECCEL_TIME` | Int | `%DB1.DBW366` | Tiempo de desaceleración |
| `1C10_CY_DCH_TIME` | Int | `%DB1.DBW370` | Tiempo de descarga |
| `1C11_CY_WASH_TIME` | Int | `%DB1.DBW374` | Tiempo de lavado |
| `1C12_CY_WAIT_TIME` | Int | `%DB1.DBW378` | Espera por sincronismos |
| `1C13_CY_TOTAL_TIME` | Int | `%DB1.DBW382` | Tiempo total de ciclo |
| `1C14_CY_NRO_COUNT` | Int | `%DB1.DBW390` | Número de ciclos en automático |
| `1C15_CY_HOUR` | Int | `%DB1.DBW388` | Ciclos por hora (÷10) |
| `1C16_CY_CH_START_SPEED` | Int | `%DB1.DBW384` | Velocidad al inicio de carga |
| `1C17_CY_CH_END_SPEED` | Int | `%DB1.DBW386` | Velocidad al fin de carga |
| `1C18_TOTAL_AZUC` | Int | `%DB1.DBW758` | Total azúcar (acumulado) |
| `1C19_TOTAL_MCOC` | Int | `%DB1.DBW760` | Total MCOC (acumulado) |

### Alarmas / fallas testigo (`1F…`)

| Tag | Tipo | Dirección | Significado |
|---|---|---|---|
| `1F01_FALLA_TESTIGO` | Bool | `%DB1.DBX271.0` | ALR 01 — Aire comprimido |
| `1F02_FALLA_TESTIGO` | Bool | `%DB1.DBX271.1` | ALR 02 — P.E. veloc. exces. |
| `1F03_FALLA_TESTIGO` | Bool | `%DB1.DBX271.2` | ALR 03 — Comp. princ. nivel |
| `1F04_FALLA_TESTIGO` | Bool | `%DB1.DBX271.3` | ALR 04 — Inversor frec. |
| `1F05_FALLA_TESTIGO` | Bool | `%DB1.DBX271.4` | ALR 05 — Lavado pres. baja |
| `1F06_FALLA_TESTIGO` | Bool | `%DB1.DBX271.5` | ALR 06 — Lavado posición |
| `1F07_FALLA_TESTIGO` | Bool | `%DB1.DBX271.6` | ALR 07 — Indicador carga |
| `1F08_FALLA_TESTIGO` | Bool | `%DB1.DBX271.7` | ALR 08 — Nivel bajo mezcl. |
| `1F09_FALLA_TESTIGO` | Bool | `%DB1.DBX270.0` | ALR 09 — Compuerta princ. |
| `1F10_FALLA_TESTIGO` | Bool | `%DB1.DBX270.1` | ALR 10 — Tiempo de carga |
| `1F11_FALLA_TESTIGO` | Bool | `%DB1.DBX270.2` | ALR 11 — Compuerta goteo |
| `1F12_FALLA_TESTIGO` | Bool | `%DB1.DBX270.3` | ALR 12 — Sep. miel blanca |
| `1F13_FALLA_TESTIGO` | Bool | `%DB1.DBX270.4` | ALR 13 — Sep. miel verde |
| `1F14_FALLA_TESTIGO` | Bool | `%DB1.DBX270.5` | ALR 14 — Oscilación |
| `1F15_FALLA_TESTIGO` | Bool | `%DB1.DBX270.6` | ALR 15 — Vibración |
| `1F16_FALLA_TESTIGO` | Bool | `%DB1.DBX270.7` | ALR 16 — Velocidad |
| `1F17_FALLA_TESTIGO` | Bool | `%DB1.DBX273.0` | ALR 17 — Salida de azúcar |
| `1F18_FALLA_TESTIGO` | Bool | `%DB1.DBX273.1` | ALR 18 — Cinta transporte |
| `1F19_FALLA_TESTIGO` | Bool | `%DB1.DBX273.2` | ALR 19 — Tapa de fondo |
| `1F20_FALLA_TESTIGO` | Bool | `%DB1.DBX273.3` | ALR 20 — Dados |
| `1F21_FALLA_TESTIGO` | Bool | `%DB1.DBX273.4` | ALR 21 — Descarg. posic. |
| `1F22_FALLA_TESTIGO` | Bool | `%DB1.DBX273.5` | ALR 22 — Descarg. vertical |
| `1F23_FALLA_TESTIGO` | Bool | `%DB1.DBX273.6` | ALR 23 — Freno activo |
| `1F24_FALLA_TESTIGO` | Bool | `%DB1.DBX273.7` | ALR 24 — Parámetros |

### Parámetros / consignas (`1P…`)

| Tag | Tipo | Dirección | Significado |
|---|---|---|---|
| `1P01_CHARGE_SPEED` | Int | `%DB1.DBW6` | Velocidad de carga |
| `1P02_CHARGE_ACCEL` | Int | `%DB1.DBW8` | Aceleración de carga |
| `1P03_AMED_CHARGE_TIME` | Int | `%DB1.DBW10` | Tiempo de carga (amedido) |
| `1P04_CHARGE_THICKNESS` | Int | `%DB1.DBW12` | Espesor de carga |
| `1P05_FIXED_CHARGE_TIME` | Int | `%DB1.DBW14` | Tiempo de carga fijo |
| `1P06_SYRUP_GREEN` | Int | `%DB1.DBW16` | Miel verde |
| `1P07_WASH_1_START` | Int | `%DB1.DBW18` | Inicio lavado 1 |
| `1P08_WASH_1_TIME` | Int | `%DB1.DBW20` | Tiempo lavado 1 |
| `1P09_SYRUP_WHITE` | Int | `%DB1.DBW22` | Miel blanca |
| `1P10_DELAY_ST_WASH_2` | Int | `%DB1.DBW24` | Retardo inicio lavado 2 |
| `1P11_WASH_2_TIME` | Int | `%DB1.DBW26` | Tiempo lavado 2 |
| `1P12_START_STEAM_WASH` | Int | `%DB1.DBW28` | Inicio lavado con vapor |
| `1P13_TIME_STEAM_WASH` | Int | `%DB1.DBW30` | Tiempo lavado con vapor |
| `1P14_SPIN_SPEED` | Int | `%DB1.DBW32` | Velocidad de centrifugado |
| `1P15_SPIN_TIME` | Int | `%DB1.DBW34` | Tiempo de centrifugado |
| `1P16_MAX_VIBRATION` | Int | `%DB1.DBW36` | Vibración máxima |
| `1P17_PLOUGH_WASH_TIME` | Int | `%DB1.DBW38` | Tiempo lavado de arado |
| `1P18_PLOUGH_BOTTON_TIME` | Int | `%DB1.DBW40` | Tiempo arado / fondo |
| `1P19_AIR_BOTTON_CONE` | Int | `%DB1.DBW42` | Aire cono de fondo |
| `1P20_ACCELERATION` | Int | `%DB1.DBW44` | Aceleración |
| `1P21_DECELERATION` | Int | `%DB1.DBW46` | Desaceleración |
| `1P22_MIN_SEQ_TIME` | Int | `%DB1.DBW48` | Tiempo mínimo de secuencia |
| `1P23_STEAM_CLEAN` | Int | `%DB1.DBW50` | Limpieza con vapor |
| `1P24_WASH_SUGAR_OUTLET` | Int | `%DB1.DBW52` | Lavado salida de azúcar |
| `1P25_START_WASH_INLET` | Int | `%DB1.DBW54` | Inicio lavado de entrada |
| `1P26_TIME_WASH_INLET` | Int | `%DB1.DBW56` | Tiempo lavado de entrada |
| `1P27_AIR_DIST_CONE` | Int | `%DB1.DBW58` | Aire cono de distribución |
| `1P28_DCH_MID_STOP` | Int | `%DB1.DBW60` | Parada intermedia de descarga |
| `1P29_MAN_STOP_RAMP_SP` | Int | `%DB1.DBW62` | Rampa de parada manual (velocidad) |
| `1P30_MAN_STOP_RAMP_TI` | Int | `%DB1.DBW64` | Rampa de parada manual (tiempo) |
| `1P31_WASH_MEDIA_P01` | Int | `%DB1.DBW2` | Medio de lavado P01 |
| `1P32_WASH_P02` | Int | `%DB1.DBW4` | Lavado P02 |

## Centrífuga 2 — `HMI_Connection_2` / `PLC_2_2419`

### Estado (bits) (`2B…`)

| Tag | Tipo | Dirección | Significado |
|---|---|---|---|
| `2B01_ANY_FAULT` | Bool | `%M40.4` | Existencia de alguna falla |
| `2B02_AUTO_MODE_ON` | Bool | `%M62.7` | Funcionamiento en automático |
| `2B03_MAN_MODE_ON` | Bool | `%M63.0` | Funcionamiento en manual |
| `2B04_CHARGE_MODE_ON` | Bool | `%M63.3` | Modo de carga activo |
| `2B05_SPIN_MODE_ON` | Bool | `%M68.3` | Centrifugado |
| `2B06_DCH_MODE_ON` | Bool | `%M63.1` | Descarga |
| `2B07_MAIN_GATE_CLOSED` | Bool | `%M105.6` | Compuerta principal cerrada |
| `2B08_MAIN_GATE_LEVEL` | Bool | `%M105.7` | Nivel de compuerta principal |
| `2B09_WASH_STEAM` | Bool | `%DB1.DBX123.1` | Lavado con vapor |
| `2B10_WASH_WATER_OK` | Bool | `%DB1.DBX125.2` | Lavado (agua) OK |
| `2B11_FEEDER_A` | Bool | `%DB1.DBX123.2` | Alimentador A |
| `2B12_FEEDER_B` | Bool | `%DB1.DBX123.3` | Alimentador B |
| `2B13_DCH_BLOCKED` | Bool | `%DB1.DBX123.0` | Descarga bloqueada |
| `2B14_DCH_HOME_UPPER` | Bool | `%DB1.DBX124.4` | Descarga en home superior |
| `2B15_MECH_BRAKE` | Bool | `%DB1.DBX123.6` | Freno mecánico |

### Ciclo y proceso (enteros) (`2C…`)

| Tag | Tipo | Dirección | Significado |
|---|---|---|---|
| `2C01_REF_VELOC` | Int | `%DB1.DBW328` | RPM de referencia |
| `2C02_ACT_VELOC` | Int | `%DB1.DBW330` | RPM (promedio en el ciclo) |
| `2C03_PIW_VFABB_DIV10` | Int | `%DB150.DBW4` | % de carga promedio en el ciclo (÷10) |
| `2C04_NIV_VIBRACION` | Int | `%DB1.DBW334` | Nivel de vibración |
| `2C05_NIV_COMPUERTA` | Int | `%DB1.DBW336` | Nivel de compuerta |
| `2C06_CY_CHARGE_TIME` | Int | `%DB1.DBW354` | Tiempo de carga |
| `2C07_CY_ACCEL_TIME` | Int | `%DB1.DBW358` | Tiempo de aceleración |
| `2C08_CY_SPIN_TIME` | Int | `%DB1.DBW362` | Tiempo de centrifugado |
| `2C09_CY_DECCEL_TIME` | Int | `%DB1.DBW366` | Tiempo de desaceleración |
| `2C10_CY_DCH_TIME` | Int | `%DB1.DBW370` | Tiempo de descarga |
| `2C11_CY_WASH_TIME` | Int | `%DB1.DBW374` | Tiempo de lavado |
| `2C12_CY_WAIT_TIME` | Int | `%DB1.DBW378` | Espera por sincronismos |
| `2C13_CY_TOTAL_TIME` | Int | `%DB1.DBW382` | Tiempo total de ciclo |
| `2C14_CY_NRO_COUNT` | Int | `%DB1.DBW390` | Número de ciclos en automático |
| `2C15_CY_HOUR` | Int | `%DB1.DBW388` | Ciclos por hora (÷10) |
| `2C16_CY_CH_START_SPEED` | Int | `%DB1.DBW384` | Velocidad al inicio de carga |
| `2C17_CY_CH_END_SPEED` | Int | `%DB1.DBW386` | Velocidad al fin de carga |
| `2C18_TOTAL_AZUC` | Int | `%DB1.DBW758` | Total azúcar (acumulado) |
| `2C19_TOTAL_MCOC` | Int | `%DB1.DBW760` | Total MCOC (acumulado) |

### Alarmas / fallas testigo (`2F…`)

| Tag | Tipo | Dirección | Significado |
|---|---|---|---|
| `2F01_FALLA_TESTIGO` | Bool | `%DB1.DBX271.0` | ALR 01 — Aire comprimido |
| `2F02_FALLA_TESTIGO` | Bool | `%DB1.DBX271.1` | ALR 02 — P.E. veloc. exces. |
| `2F03_FALLA_TESTIGO` | Bool | `%DB1.DBX271.2` | ALR 03 — Comp. princ. nivel |
| `2F04_FALLA_TESTIGO` | Bool | `%DB1.DBX271.3` | ALR 04 — Inversor frec. |
| `2F05_FALLA_TESTIGO` | Bool | `%DB1.DBX271.4` | ALR 05 — Lavado pres. baja |
| `2F06_FALLA_TESTIGO` | Bool | `%DB1.DBX271.5` | ALR 06 — Lavado posición |
| `2F07_FALLA_TESTIGO` | Bool | `%DB1.DBX271.6` | ALR 07 — Indicador carga |
| `2F08_FALLA_TESTIGO` | Bool | `%DB1.DBX271.7` | ALR 08 — Nivel bajo mezcl. |
| `2F09_FALLA_TESTIGO` | Bool | `%DB1.DBX270.0` | ALR 09 — Compuerta princ. |
| `2F10_FALLA_TESTIGO` | Bool | `%DB1.DBX270.1` | ALR 10 — Tiempo de carga |
| `2F11_FALLA_TESTIGO` | Bool | `%DB1.DBX270.2` | ALR 11 — Compuerta goteo |
| `2F12_FALLA_TESTIGO` | Bool | `%DB1.DBX270.3` | ALR 12 — Sep. miel blanca |
| `2F13_FALLA_TESTIGO` | Bool | `%DB1.DBX270.4` | ALR 13 — Sep. miel verde |
| `2F14_FALLA_TESTIGO` | Bool | `%DB1.DBX270.5` | ALR 14 — Oscilación |
| `2F15_FALLA_TESTIGO` | Bool | `%DB1.DBX270.6` | ALR 15 — Vibración |
| `2F16_FALLA_TESTIGO` | Bool | `%DB1.DBX270.7` | ALR 16 — Velocidad |
| `2F17_FALLA_TESTIGO` | Bool | `%DB1.DBX273.0` | ALR 17 — Salida de azúcar |
| `2F18_FALLA_TESTIGO` | Bool | `%DB1.DBX273.1` | ALR 18 — Cinta transporte |
| `2F19_FALLA_TESTIGO` | Bool | `%DB1.DBX273.2` | ALR 19 — Tapa de fondo |
| `2F20_FALLA_TESTIGO` | Bool | `%DB1.DBX273.3` | ALR 20 — Dados |
| `2F21_FALLA_TESTIGO` | Bool | `%DB1.DBX273.4` | ALR 21 — Descarg. posic. |
| `2F22_FALLA_TESTIGO` | Bool | `%DB1.DBX273.5` | ALR 22 — Descarg. vertical |
| `2F23_FALLA_TESTIGO` | Bool | `%DB1.DBX273.6` | ALR 23 — Freno activo |
| `2F24_FALLA_TESTIGO` | Bool | `%DB1.DBX273.7` | ALR 24 — Parámetros |

### Parámetros / consignas (`2P…`)

| Tag | Tipo | Dirección | Significado |
|---|---|---|---|
| `2P01_CHARGE_SPEED` | Int | `%DB1.DBW6` | Velocidad de carga |
| `2P02_CHARGE_ACCEL` | Int | `%DB1.DBW8` | Aceleración de carga |
| `2P03_AMED_CHARGE_TIME` | Int | `%DB1.DBW10` | Tiempo de carga (amedido) |
| `2P04_CHARGE_THICKNESS` | Int | `%DB1.DBW12` | Espesor de carga |
| `2P05_FIXED_CHARGE_TIME` | Int | `%DB1.DBW14` | Tiempo de carga fijo |
| `2P06_SYRUP_GREEN` | Int | `%DB1.DBW16` | Miel verde |
| `2P07_WASH_1_START` | Int | `%DB1.DBW18` | Inicio lavado 1 |
| `2P08_WASH_1_TIME` | Int | `%DB1.DBW20` | Tiempo lavado 1 |
| `2P09_SYRUP_WHITE` | Int | `%DB1.DBW22` | Miel blanca |
| `2P10_DELAY_ST_WASH_2` | Int | `%DB1.DBW24` | Retardo inicio lavado 2 |
| `2P11_WASH_2_TIME` | Int | `%DB1.DBW26` | Tiempo lavado 2 |
| `2P12_START_STEAM_WASH` | Int | `%DB1.DBW28` | Inicio lavado con vapor |
| `2P13_TIME_STEAM_WASH` | Int | `%DB1.DBW30` | Tiempo lavado con vapor |
| `2P14_SPIN_SPEED` | Int | `%DB1.DBW32` | Velocidad de centrifugado |
| `2P15_SPIN_TIME` | Int | `%DB1.DBW34` | Tiempo de centrifugado |
| `2P16_MAX_VIBRATION` | Int | `%DB1.DBW36` | Vibración máxima |
| `2P17_PLOUGH_WASH_TIME` | Int | `%DB1.DBW38` | Tiempo lavado de arado |
| `2P18_PLOUGH_BOTTON_TIME` | Int | `%DB1.DBW40` | Tiempo arado / fondo |
| `2P19_AIR_BOTTON_CONE` | Int | `%DB1.DBW42` | Aire cono de fondo |
| `2P20_ACCELERATION` | Int | `%DB1.DBW44` | Aceleración |
| `2P21_DECELERATION` | Int | `%DB1.DBW46` | Desaceleración |
| `2P22_MIN_SEQ_TIME` | Int | `%DB1.DBW48` | Tiempo mínimo de secuencia |
| `2P23_STEAM_CLEAN` | Int | `%DB1.DBW50` | Limpieza con vapor |
| `2P24_WASH_SUGAR_OUTLET` | Int | `%DB1.DBW52` | Lavado salida de azúcar |
| `2P25_START_WASH_INLET` | Int | `%DB1.DBW54` | Inicio lavado de entrada |
| `2P26_TIME_WASH_INLET` | Int | `%DB1.DBW56` | Tiempo lavado de entrada |
| `2P27_AIR_DIST_CONE` | Int | `%DB1.DBW58` | Aire cono de distribución |
| `2P28_DCH_MID_STOP` | Int | `%DB1.DBW60` | Parada intermedia de descarga |
| `2P29_MAN_STOP_RAMP_SP` | Int | `%DB1.DBW62` | Rampa de parada manual (velocidad) |
| `2P30_MAN_STOP_RAMP_TI` | Int | `%DB1.DBW64` | Rampa de parada manual (tiempo) |
| `2P31_WASH_MEDIA_P01` | Int | `%DB1.DBW2` | Medio de lavado P01 |
| `2P32_WASH_P02` | Int | `%DB1.DBW4` | Lavado P02 |

## Centrífuga 3 — `HMI_Connection_3` / `PLC_3_2650`

### Estado (bits) (`3B…`)

| Tag | Tipo | Dirección | Significado |
|---|---|---|---|
| `3B01_ANY_FAULT` | Bool | `%M40.4` | Existencia de alguna falla |
| `3B02_AUTO_MODE_ON` | Bool | `%M62.7` | Funcionamiento en automático |
| `3B03_MAN_MODE_ON` | Bool | `%M63.0` | Funcionamiento en manual |
| `3B04_CHARGE_MODE_ON` | Bool | `%M63.3` | Modo de carga activo |
| `3B05_SPIN_MODE_ON` | Bool | `%M68.3` | Centrifugado |
| `3B06_DCH_MODE_ON` | Bool | `%M63.1` | Descarga |
| `3B07_MAIN_GATE_CLOSED` | Bool | `%M105.6` | Compuerta principal cerrada |
| `3B08_MAIN_GATE_LEVEL` | Bool | `%M105.7` | Nivel de compuerta principal |
| `3B09_WASH_STEAM` | Bool | `%DB1.DBX123.1` | Lavado con vapor |
| `3B10_WASH_WATER_OK` | Bool | `%DB1.DBX125.2` | Lavado (agua) OK |
| `3B11_FEEDER_A` | Bool | `%DB1.DBX123.2` | Alimentador A |
| `3B12_FEEDER_B` | Bool | `%DB1.DBX123.3` | Alimentador B |
| `3B13_DCH_BLOCKED` | Bool | `%DB1.DBX123.0` | Descarga bloqueada |
| `3B14_DCH_HOME_UPPER` | Bool | `%DB1.DBX124.4` | Descarga en home superior |
| `3B15_MECH_BRAKE` | Bool | `%DB1.DBX123.6` | Freno mecánico |

### Ciclo y proceso (enteros) (`3C…`)

| Tag | Tipo | Dirección | Significado |
|---|---|---|---|
| `3C01_REF_VELOC` | Int | `%DB1.DBW328` | RPM de referencia |
| `3C02_ACT_VELOC` | Int | `%DB1.DBW330` | RPM (promedio en el ciclo) |
| `3C03_PIW_VFABB_DIV10` | Int | `%DB150.DBW4` | % de carga promedio en el ciclo (÷10) |
| `3C04_NIV_VIBRACION` | Int | `%DB1.DBW334` | Nivel de vibración |
| `3C05_NIV_COMPUERTA` | Int | `%DB1.DBW336` | Nivel de compuerta |
| `3C06_CY_CHARGE_TIME` | Int | `%DB1.DBW354` | Tiempo de carga |
| `3C07_CY_ACCEL_TIME` | Int | `%DB1.DBW358` | Tiempo de aceleración |
| `3C08_CY_SPIN_TIME` | Int | `%DB1.DBW362` | Tiempo de centrifugado |
| `3C09_CY_DECCEL_TIME` | Int | `%DB1.DBW366` | Tiempo de desaceleración |
| `3C10_CY_DCH_TIME` | Int | `%DB1.DBW370` | Tiempo de descarga |
| `3C11_CY_WASH_TIME` | Int | `%DB1.DBW374` | Tiempo de lavado |
| `3C12_CY_WAIT_TIME` | Int | `%DB1.DBW378` | Espera por sincronismos |
| `3C13_CY_TOTAL_TIME` | Int | `%DB1.DBW382` | Tiempo total de ciclo |
| `3C14_CY_NRO_COUNT` | Int | `%DB1.DBW390` | Número de ciclos en automático |
| `3C15_CY_HOUR` | Int | `%DB1.DBW388` | Ciclos por hora (÷10) |
| `3C16_CY_CH_START_SPEED` | Int | `%DB1.DBW384` | Velocidad al inicio de carga |
| `3C17_CY_CH_END_SPEED` | Int | `%DB1.DBW386` | Velocidad al fin de carga |
| `3C18_TOTAL_AZUC` | Int | `%DB1.DBW758` | Total azúcar (acumulado) |
| `3C19_TOTAL_MCOC` | Int | `%DB1.DBW760` | Total MCOC (acumulado) |

### Alarmas / fallas testigo (`3F…`)

| Tag | Tipo | Dirección | Significado |
|---|---|---|---|
| `3F01_FALLA_TESTIGO` | Bool | `%DB1.DBX271.0` | ALR 01 — Aire comprimido |
| `3F02_FALLA_TESTIGO` | Bool | `%DB1.DBX271.1` | ALR 02 — P.E. veloc. exces. |
| `3F03_FALLA_TESTIGO` | Bool | `%DB1.DBX271.2` | ALR 03 — Comp. princ. nivel |
| `3F04_FALLA_TESTIGO` | Bool | `%DB1.DBX271.3` | ALR 04 — Inversor frec. |
| `3F05_FALLA_TESTIGO` | Bool | `%DB1.DBX271.4` | ALR 05 — Lavado pres. baja |
| `3F06_FALLA_TESTIGO` | Bool | `%DB1.DBX271.5` | ALR 06 — Lavado posición |
| `3F07_FALLA_TESTIGO` | Bool | `%DB1.DBX271.6` | ALR 07 — Indicador carga |
| `3F08_FALLA_TESTIGO` | Bool | `%DB1.DBX271.7` | ALR 08 — Nivel bajo mezcl. |
| `3F09_FALLA_TESTIGO` | Bool | `%DB1.DBX270.0` | ALR 09 — Compuerta princ. |
| `3F10_FALLA_TESTIGO` | Bool | `%DB1.DBX270.1` | ALR 10 — Tiempo de carga |
| `3F11_FALLA_TESTIGO` | Bool | `%DB1.DBX270.2` | ALR 11 — Compuerta goteo |
| `3F12_FALLA_TESTIGO` | Bool | `%DB1.DBX270.3` | ALR 12 — Sep. miel blanca |
| `3F13_FALLA_TESTIGO` | Bool | `%DB1.DBX270.4` | ALR 13 — Sep. miel verde |
| `3F14_FALLA_TESTIGO` | Bool | `%DB1.DBX270.5` | ALR 14 — Oscilación |
| `3F15_FALLA_TESTIGO` | Bool | `%DB1.DBX270.6` | ALR 15 — Vibración |
| `3F16_FALLA_TESTIGO` | Bool | `%DB1.DBX270.7` | ALR 16 — Velocidad |
| `3F17_FALLA_TESTIGO` | Bool | `%DB1.DBX273.0` | ALR 17 — Salida de azúcar |
| `3F18_FALLA_TESTIGO` | Bool | `%DB1.DBX273.1` | ALR 18 — Cinta transporte |
| `3F19_FALLA_TESTIGO` | Bool | `%DB1.DBX273.2` | ALR 19 — Tapa de fondo |
| `3F20_FALLA_TESTIGO` | Bool | `%DB1.DBX273.3` | ALR 20 — Dados |
| `3F21_FALLA_TESTIGO` | Bool | `%DB1.DBX273.4` | ALR 21 — Descarg. posic. |
| `3F22_FALLA_TESTIGO` | Bool | `%DB1.DBX273.5` | ALR 22 — Descarg. vertical |
| `3F23_FALLA_TESTIGO` | Bool | `%DB1.DBX273.6` | ALR 23 — Freno activo |
| `3F24_FALLA_TESTIGO` | Bool | `%DB1.DBX273.7` | ALR 24 — Parámetros |

### Parámetros / consignas (`3P…`)

| Tag | Tipo | Dirección | Significado |
|---|---|---|---|
| `3P01_CHARGE_SPEED` | Int | `%DB1.DBW6` | Velocidad de carga |
| `3P02_CHARGE_ACCEL` | Int | `%DB1.DBW8` | Aceleración de carga |
| `3P03_AMED_CHARGE_TIME` | Int | `%DB1.DBW10` | Tiempo de carga (amedido) |
| `3P04_CHARGE_THICKNESS` | Int | `%DB1.DBW12` | Espesor de carga |
| `3P05_FIXED_CHARGE_TIME` | Int | `%DB1.DBW14` | Tiempo de carga fijo |
| `3P06_SYRUP_GREEN` | Int | `%DB1.DBW16` | Miel verde |
| `3P07_WASH_1_START` | Int | `%DB1.DBW18` | Inicio lavado 1 |
| `3P08_WASH_1_TIME` | Int | `%DB1.DBW20` | Tiempo lavado 1 |
| `3P09_SYRUP_WHITE` | Int | `%DB1.DBW22` | Miel blanca |
| `3P10_DELAY_ST_WASH_2` | Int | `%DB1.DBW24` | Retardo inicio lavado 2 |
| `3P11_WASH_2_TIME` | Int | `%DB1.DBW26` | Tiempo lavado 2 |
| `3P12_START_STEAM_WASH` | Int | `%DB1.DBW28` | Inicio lavado con vapor |
| `3P13_TIME_STEAM_WASH` | Int | `%DB1.DBW30` | Tiempo lavado con vapor |
| `3P14_SPIN_SPEED` | Int | `%DB1.DBW32` | Velocidad de centrifugado |
| `3P15_SPIN_TIME` | Int | `%DB1.DBW34` | Tiempo de centrifugado |
| `3P16_MAX_VIBRATION` | Int | `%DB1.DBW36` | Vibración máxima |
| `3P17_PLOUGH_WASH_TIME` | Int | `%DB1.DBW38` | Tiempo lavado de arado |
| `3P18_PLOUGH_BOTTON_TIME` | Int | `%DB1.DBW40` | Tiempo arado / fondo |
| `3P19_AIR_BOTTON_CONE` | Int | `%DB1.DBW42` | Aire cono de fondo |
| `3P20_ACCELERATION` | Int | `%DB1.DBW44` | Aceleración |
| `3P21_DECELERATION` | Int | `%DB1.DBW46` | Desaceleración |
| `3P22_MIN_SEQ_TIME` | Int | `%DB1.DBW48` | Tiempo mínimo de secuencia |
| `3P23_STEAM_CLEAN` | Int | `%DB1.DBW50` | Limpieza con vapor |
| `3P24_WASH_SUGAR_OUTLET` | Int | `%DB1.DBW52` | Lavado salida de azúcar |
| `3P25_START_WASH_INLET` | Int | `%DB1.DBW54` | Inicio lavado de entrada |
| `3P26_TIME_WASH_INLET` | Int | `%DB1.DBW56` | Tiempo lavado de entrada |
| `3P27_AIR_DIST_CONE` | Int | `%DB1.DBW58` | Aire cono de distribución |
| `3P28_DCH_MID_STOP` | Int | `%DB1.DBW60` | Parada intermedia de descarga |
| `3P29_MAN_STOP_RAMP_SP` | Int | `%DB1.DBW62` | Rampa de parada manual (velocidad) |
| `3P30_MAN_STOP_RAMP_TI` | Int | `%DB1.DBW64` | Rampa de parada manual (tiempo) |
| `3P31_WASH_MEDIA_P01` | Int | `%DB1.DBW2` | Medio de lavado P01 |
| `3P32_WASH_P02` | Int | `%DB1.DBW4` | Lavado P02 |

