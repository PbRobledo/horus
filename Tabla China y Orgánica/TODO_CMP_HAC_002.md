# TO-DO · CMP_HAC 002 - Tabla China y Orgánica

Orden de trabajo: **Fase 1 (modelo) → Gate de validación → Fase 2 (visuales en hoja nueva)**.
No se arranca la Fase 2 hasta que el Gate esté aprobado.

Estado al 2026-10-30: **Fases 1–8 aplicadas; datos recargados al 30/10/2026. Pendiente validación (G3, G4 y G5) y publicación.**

---

## Fase 0 · Preparación

- [~] 0.1 ~~Respaldo con git~~ — omitido a pedido (el reporte publicado está en Power BI Service). Copia local de los originales en el scratchpad de la sesión de Claude.
- [ ] 0.2 Captura de las tarjetas actuales en Power BI Service (línea base) — usuario.
- [x] 0.3 Consulta de control en SQL Server: resultados en la tabla del Gate (S35–S43).

---

## Fase 1 · Desarrollo del modelo (datos)

### 1A · Calendario único
- [x] 1A.1 Fecha/hora automáticas desactivada (`__PBI_TimeIntelligenceEnabled = 0`); eliminadas las 6 `LocalDateTable_*`, `DateTableTemplate_*`, sus relaciones y las `variation` de las columnas. Eliminada la metadata lingüística autogenerada (`cultures/es-AR.tmdl`), que apuntaba a esas jerarquías; Desktop la regenera.
- [x] 1A.2 Tabla calculada `Calendario` (Date, Semana inicio, Año, Mes, Año ISO, Semana ISO, Semana, Semana índice). Cubre de lunes a domingo las semanas con faena de ambas tablas.
- [x] 1A.3 Marcada como tabla de fechas (`dataCategory: Time`, `Date` como clave); `Semana` ordenada por `Semana índice`, `Mes` por `Mes Nro`.
- [x] 1A.4 Relaciones `Calendario[Date]` 1:* `fecha_faena` en ambas tablas.

### 1B · Limpieza de columnas
- [x] 1B.1 `summarizeBy: none` en `precio_final`, `rinde`, `promedio_cabeza`, `promedio_media_res`, `precio_tabla_pond`, `semana_iso`, `anio_iso`, `plazo`.
- [x] 1B.2 Formatos: montos `$ #,0`, precio `$ #,0.00`, kg `#,0`, cabezas `#,0.#`, rinde `0.0%`, promedios `#,0.0`.

### 1C · Tabla de medidas (corrección)
- [x] 1C.1 `Tabla` → `_Medidas`, carpetas `China\…`, `Orgánico\…`, `General`.
- [x] 1C.2 Medidas base por programa (prefijo `China` / `Org`): Tropas, Cabezas, Kg Vivos, Kg Carne, Kg Liquidados, Monto Hacienda, Monto Gastos, Monto Total, **Precio Final (ponderado)**.
- [x] 1C.3 Borradas las duplicadas (`Precio Final Promedio`, `Monto Total (Final con Gastos)`, `Kg Vivos TOT`, `Kg Carne TOT`, `Precio Final C/ Dig.`, etc.).
- [x] 1C.4 `LastRefresh` (rota) reemplazada por `Ultima carga ETL`.

### 1D · Medidas nuevas (55 medidas en total)
- [x] 1D.1 `Precio Hacienda`, `Gastos x Kg`.
- [x] 1D.2 `Rinde` ponderado (solo tropas con kg vivos), `Peso Vivo Prom`.
- [x] 1D.3 `% Kg Liquidado`.
- [x] 1D.4 `Precio Sem Ant`, `Var % Sem Ant`, `Precio 4 Sem Móvil` (toman la última semana con datos del contexto).
- [x] 1D.5 `Premium Orgánico`, `Premium Orgánico %`.
- [x] 1D.6 `% Cab A`…`% Cab E`, `% Decomiso`, `% Menor 80`.
- [x] 1D.7 `Peso Medio Res`, `Plazo Pond`.

### 1E · Ajuste de las hojas existentes
- [x] 1E.1 Tarjetas de precio → `China Precio Final` / `Org Precio Final` (antes: promedio simple).
- [x] 1E.2 Resto de tarjetas → medidas `China …` / `Org …` (se quitaron los formatos fijados en el visual; manda el de la medida).
- [x] 1E.3 Tablas: valores numéricos → medidas; `semana_iso` sin resumir; Año/Mes de la jerarquía automática → `Calendario[Año]` / `Calendario[Mes]`.
- [x] 1E.4 Segmentadores → `Calendario[Semana]` (selección guardada: 2026-S40, la misma que tenían).
- [ ] 1E.5 Sincronizar los dos segmentadores (Desktop → Ver → Sincronizar segmentaciones) — en Desktop, 1 minuto.

---

## Gate · Validación (responsable: usuario)

**Al abrir en Desktop:** abrir el `.pbip` → **Actualizar** (el calendario se calcula con el refresh) → revisar que no haya errores ni visuales rotos.

- [x] G.1 Tarjetas y totales = valores de SQL Server (consulta del 2026-10-30, últimas cargas del ETL el 2026-10-25):

| Semana | China tropas | China cabezas | **China precio** | Org tropas | Org cabezas | **Org precio** | Premium Org |
|---|---|---|---|---|---|---|---|
| 2026-S35 | 34 | 828 | 6.410,64 | 23 | 901 | 7.180,73 | 770,09 |
| 2026-S36 | 39 | 955 | 6.462,80 | 38 | 1.263 | 7.155,19 | 692,39 |
| 2026-S37 | 43 | 1.061 | 6.502,95 | 50 | 1.803 | 7.161,38 | 658,44 |
| 2026-S38 | 49 | 1.322 | 6.543,41 | 42 | 1.732 | 7.294,33 | 750,91 |
| 2026-S39 | 46 | 1.674 | 6.674,58 | 41 | 1.397 | 7.205,46 | 530,88 |
| 2026-S40 | 41 | 1.073 | 6.690,37 | 48 | 1.663 | 7.140,05 | 449,67 |
| 2026-S41 | 32 | 954 | 6.715,56 | 43 | 1.557 | 7.200,28 | 484,72 |
| 2026-S42 | 30 | 736 | 6.682,11 | 43 | 1.552 | 7.220,02 | 537,91 |
| 2026-S43 | 49 | 1.229 | 6.627,55 | 44 | 1.525 | 7.182,76 | 555,21 |

  Referencia de lo que mostraban antes las tarjetas (promedio simple), S40: China 6.652,70 · Orgánico 7.092,26.

- [x] G.2 Medidas nuevas, control en 2 semanas:

| Medida | China S40 | China S43 | Org S40 | Org S43 |
|---|---|---|---|---|
| Precio Hacienda | 6.636,23 | 6.569,36 | 7.067,00 | 7.076,86 |
| Gastos x Kg | 54,15 | 58,20 | 73,05 | 105,90 |
| % Kg Liquidado | 100,0% | **96,9%** | 100,0% | 100,0% |
| Precio Sem Ant | 6.674,58 | 6.682,11 | 7.205,46 | 7.220,02 |
| Var % Sem Ant | +0,2% | -0,8% | -0,9% | -0,5% |
| Precio 4 Sem Móvil | 6.608,70 | 6.676,85 | 7.199,62 | 7.184,50 |
| Rinde | 51,3% | 50,9% | 52,0% | 51,3% |
| Peso Medio Res | 130,3 | 119,2 | 127,2 | 120,1 |
| % Cab A | 3,3% | 1,5% | 0,1% | 1,3% |
| % Decomiso | 0,8% | 1,6% | 1,0% | 1,0% |
| % Menor 80 | 6,5% | 11,8% | 1,4% | 1,6% |
| Plazo Pond | 29,4 | 29,6 | 29,3 | 27,3 |

- [x] G.3 La fila de total de las tablas muestra el precio ponderado (no suma ni promedio).
- [x] G.4 Hojas existentes sin visuales rotos; columnas Año/Mes de las tablas correctas.
- [x] G.5 `Ultima carga ETL` muestra "Datos al 25/10/2026 14:27".
- [x] G.6 **Aprobado** por el usuario (2026-10-30).

---

## Fase 2 · Visuales (hoja nueva: "Análisis semanal")

Decisiones del usuario: **una sola hoja**; el segmentador de Semana **filtra todos los visuales** (para ver
tendencia, dejarlo sin selección o elegir varias semanas). Las hojas "Tabla China" y "Tabla Orgánico" no se tocan.

### 2A · Soporte en el modelo
- [x] 2A.1 Dimensión `Tropas` (tropa, Programa, Tipo compra, Comprador, Proveedor) que une ambos programas; relaciones 1:* a `tropa` de cada tabla. Permite cruzar China y Orgánico en un mismo visual.
- [x] 2A.2 `China/Org Precio Ult Sem` (precio de la última semana del contexto) y `China/Org Desvío vs Total` (precio del comprador/proveedor − precio del programa).
- [x] 2A.3 `Var % Sem Ant` compara la **última semana** del contexto (con una sola semana da lo mismo que lo validado en G.2).
- [x] 2A.4 Carpeta `Total ambos programas`: Tropas, Cabezas, Kg Carne, Precio Final, Precio Hacienda, Gastos x Kg, Rinde, Cab A–E, Cab Decomiso, % Menor 80.

### 2B · Hoja "Análisis semanal" (1920×1080, mismo fondo)
- [x] 2.1 Hoja + segmentadores Semana (`Calendario[Semana]`) y Tipo de compra (`Tropas[Tipo compra]`) + tarjeta "Última carga".
- [x] 2.2 KPIs China y Orgánico: precio última semana, var. vs semana anterior, móvil 4 sem., % kg liquidado, rinde (+ premium en Orgánico).
- [x] 2.3 Kg carne por semana (columnas China / Orgánico). *Separado del precio: un gráfico, un eje.*
- [x] 2.4 Precio final ponderado por semana: líneas China / Orgánico + móvil 4 sem. punteado; premium en el tooltip.
- [x] 2.5 Gastos por kg por semana (columnas China / Orgánico); precio hacienda en el tooltip. *Reemplaza la pila hacienda+gastos: los gastos son <1% del precio y no se verían.*
- [x] 2.6 Mix de calidades 100% por programa (A–E + decomiso); % menor 80 kg en el tooltip.
- [x] 2.7 Matriz Comprador → Proveedor: tropas, precio y desvío por programa.
- [x] 2.8 Dispersión rinde vs precio por tropa, color por programa.
- [x] 2.9 Segmentador por tipo de compra (en la base solo hay Directa estancia y Directa intermediario).
- [x] 2.10 Detalle por tropa en el tooltip de la dispersión (cabezas, precio hacienda, gastos x kg, % menor 80).
- [x] 2.11 Formato condicional del desvío → hecho en 3C.4.
- [ ] 2.12 Sincronizar el segmentador de Semana con las otras hojas (si se quiere) — en Desktop. **No sincronizarlo con "KPIs semanales"**: una semana fija dejaría el correo congelado.

Colores: China `#2a78d6` (azul), Orgánico `#eb6834` (naranja), validados para daltonismo; calidades en rampa verde A (oscuro) → E (claro), decomiso gris.

## Fase 3 · Rediseño tras la revisión del usuario

Pedido: gráficos simples y precisos para la jefatura (sin dispersión ni barras 100% apiladas), mapa de calor
rojo/verde en el desvío de la matriz, y separar una hoja de **KPIs puros** que es la única que se envía por correo
(suscripción de Power BI Service).

### 3A · Modelo
- [x] 3A.1 Medidas `Ult Sem` por programa (carpeta `China\Última semana` / `Orgánico\Última semana`): Kg Carne, Cabezas, Rinde, % Kg Liquidado. Toda la hoja de KPIs muestra la misma semana.
- [x] 3A.2 `Premium Orgánico Ult Sem` y `Premium Orgánico % Ult Sem` (General).
- [x] 3A.3 `Semana reportada` (General): "Semana 2026-S43 · faena del 19/10 al 25/10/2026".
- [x] 3A.4 `China/Org Cab A`…`E` y `Cab Decomiso` (carpeta Faena), para las tortas.

### 3B · Hoja "KPIs semanales" (nueva, antes de "Análisis semanal")
- [x] 3B.1 **Sin segmentadores**: siempre muestra la última semana con datos, así cada correo llega con la semana nueva.
- [x] 3B.2 Encabezado: semana reportada + fecha de última carga.
- [x] 3B.3 Por programa: precio final (grande), var. vs semana anterior, móvil 4 semanas; kg carne, cabezas, rinde, % kg liquidado.
- [x] 3B.4 Franja inferior: premium orgánico en $/kg y en %.
- [ ] 3B.5 En Service: suscripción **solo a la página "KPIs semanales"** (Suscribirse → elegir página → frecuencia semanal, después de la carga del ETL).

### 3C · Hoja "Análisis semanal" (solo gráficos)
- [x] 3C.1 Se quitan los KPIs (pasan a su hoja), la dispersión y el mix 100% apilado.
- [x] 3C.2 Fila 1 (más alta): precio por semana, kg carne por semana, gastos por kg.
- [x] 3C.3 Fila 2: torta de calidades China, torta de calidades Orgánico (% de cabezas) y matriz comprador/proveedor.
- [x] 3C.4 Matriz: fondo con degradé en las columnas de desvío: verde (por debajo del total del programa) → blanco (0) → rojo (por encima). Reemplaza 2.11.

## Fase 4 · Aviso de tropas sin costear (hojas "Tabla China" y "Tabla Orgánico")

Origen: el ETL marca `tropa_liquidada = 0` cuando la tropa no tiene log de costeo con OC de hacienda en SAP;
esas tropas tienen cabezas y kilos pero monto 0 y precio vacío. El ETL recarga la semana cerrada + las 3 previas,
así que una tropa que se costea en SAP dentro de ese plazo se corrige sola en la siguiente corrida.

- [x] 4.1 Medidas en `China\Costeo` / `Orgánico\Costeo`: `Tropas sin costear`, `Aviso costeo` (texto con las tropas, proveedor y fecha de faena; hasta 3 y "y N más") y `Aviso costeo color`.
- [x] 4.2 Tarjeta arriba de cada tabla, al lado del segmentador de semana: rojo con ⚠ si hay pendientes, verde si está todo costeado. Respeta la semana elegida.
- [ ] G4.1 China, todas las semanas o 2026-S43: "⚠ 1 tropa sin costear en SAP … 080351/26 (OVIDIO OTERO S.A., faena 20/10)", en rojo.
- [ ] G4.2 China 2026-S40 y Orgánico (cualquier semana): mensaje verde "Todas las tropas de la selección están costeadas en SAP."

## Fase 5 · KPIs iguales en todas las hojas, última carga y datos al 30/10/2026

- [x] 5.1 "Tabla China" y "Tabla Orgánico": las 5 tarjetas viejas (cabezas, precio, kg vivos, kg carne, monto) se reemplazan por los 7 KPIs de "KPIs semanales": precio final, var. vs semana anterior, móvil 4 semanas, kg carne, cabezas, rinde, % kg liquidado.
- [x] 5.2 Los segmentadores de semana de esas hojas arrancan **sin semana elegida** (antes quedaba guardada la S40): las tarjetas muestran la última semana con datos, igual que la hoja de KPIs; la tabla muestra todas las semanas. Si se elige una semana, tarjetas y tabla pasan a esa semana.
- [x] 5.3 "Datos al dd/mm/aaaa hh:mm" en las cuatro hojas, mismo lugar (arriba a la derecha, debajo de la banda verde).
- [x] 5.4 El aviso de costeo detalla hasta 3 tropas (se acortó para dejar lugar a la fecha).
- [x] 5.5 ETL corrido el 30/10/2026 16:36: recargó S41–S44 en SQL Server (China 132 filas, Orgánico 169). La S44 (faena 26/10 al 30/10) está parcialmente liquidada.

### Gate 5 · Validación (usuario)

Cerrar Power BI Desktop **sin guardar** → reabrir el `.pbip` → **Actualizar**.

- [ ] G5.1 Las cuatro hojas: "Datos al 30/10/2026 16:36".
- [ ] G5.2 KPIs (hoja KPIs, Tabla China y Tabla Orgánico sin semana elegida) = semana **2026-S44**:

| | China | Orgánico |
|---|---|---|
| Precio final | 6.569,69 | 7.120,92 |
| Var. vs S43 | −0,9% (S43 = 6.631,46) | −0,9% (S43 = 7.182,76) |
| Móvil 4 semanas (S41–S44) | ≈ 6.659,2 | ≈ 7.183,1 |
| Kg carne | 115.056 | 338.821 |
| Cabezas | 475 | 1.429 |
| % kg liquidado | 92,7% | 96,7% |
| Premium orgánico | 551,23 $/kg · 8,4% | |

- [ ] G5.3 Aviso de costeo sin semana elegida: China "⚠ 1 tropa … 080561/26 (ESTABLECIMIENTO LA CARMEN SRL, faena 30/10)"; Orgánico "⚠ 2 tropas … 080569/26 y 080573/26 (AGROPECUARIA CINCO ESTRELLAS SRL, faena 30/10)". Con S43 elegida: verde en las dos (la 080351/26 ya se costeó en SAP y la recarga la corrigió).

## Fase 6 · Tortas de calidades con total de cabezas

- [x] 6.1 Título dinámico `China/Org Titulo calidades`: "Calidades China · 10.307 cabezas"; cambia con los segmentadores de Semana y Tipo de compra.
- [x] 6.2 Cada porción muestra cabezas y % del total.
- [x] 6.3 Control en SQL: la suma A–E + decomiso es igual a las cabezas en **cada** semana y tipo de compra (China 10.307 = 10.307; Orgánico 14.822 = 14.822). El total del título coincide con las porciones y con las cabezas del resto del reporte.
- [ ] G6.1 Sin filtros: China 10.307 cabezas, Orgánico 14.822. Con 2026-S44: China 475, Orgánico 1.429. S44 + "Directa estancia": China 439, Orgánico 1.283; "Directa intermediario": China 36, Orgánico 146.

## Fase 7 · Tortas con categoría real y total en el centro (reemplaza 6.1)

- [x] 7.1 Tabla `Calidades` (A, B, C, D, E, Decomiso; sin relaciones) y medidas `China/Org Cab por calidad`: suma las cabezas de las calidades visibles; sin filtro de calidad = total de cabezas. Calidad sin cabezas → vacío (no aparece una porción "0").
- [x] 7.2 Tortas con Calidad como categoría: colores fijos por calidad, leyenda abajo, cabezas y % en cada porción, título fijo.
- [x] 7.3 Tarjeta en el centro de cada torta: total de cabezas de la selección de los segmentadores (Semana, Tipo de compra); al hacer clic en una porción muestra las cabezas de esa calidad. El clic en la torta China no filtra la de Orgánico, y viceversa.
- [x] 7.4 Se quitan los títulos dinámicos de la fase 6 (`Titulo calidades`), que además dejaban una coma de más ("10.307, cabezas").
- [ ] G7.1 Sin filtros: centro China 10.307, Orgánico 14.822. 2026-S44: China 475 (B 11 · C 164 · D 172 · E 119 · Decomiso 9; sin A), Orgánico 1.429 (B 46 · C 984 · D 230 · E 147 · Decomiso 22; sin A).
- [ ] G7.2 Clic en la porción C de China (S44): el centro China pasa a 164; el centro Orgánico sigue en 1.429.
- [ ] G7.3 Si la tarjeta del centro queda corrida respecto del hueco, moverla a mano en Desktop (posición calculada, no medida).

## Fase 8 · Ajustes de lectura

- [x] 8.1 Todas las tarjetas (39, en las cuatro hojas) con unidades "Ninguna": "6.690,37" en vez de "6,69 mil".
- [x] 8.2 Columna `Calendario[Semana filtro]`: la última semana con faena aparece como **"Semana actual"**; el resto con su código (2026-S43…). Los segmentadores de semana de las tres hojas la usan, ordenados de la más nueva a la más vieja.
- [x] 8.3 "Análisis semanal": el segmentador de semana filtra solo las dos tortas (con sus centros) y la matriz. Precio, kg carne y gastos muestran siempre todas las semanas (sí siguen al tipo de compra).
- [x] 8.4 Títulos dinámicos (carpeta `General\Títulos`): dicen qué semana y qué tipo de compra se están viendo, p. ej. "Calidades China · Semana actual (2026-S44) · Todos los tipos de compra". Las tortas llevan el título en el color de su programa (China azul, Orgánico naranja). Los gráficos de tendencia dicen "Todas las semanas".
- [ ] G8.1 Tarjetas sin "mil" en ninguna hoja. Si alguna sigue mostrándolo: seleccionarla → Formato → Valor → Unidades de visualización → Ninguna (y avisar cuál, para ajustar el archivo).
- [ ] G8.2 Segmentador: primera opción "Semana actual"; al elegirla, tortas y matriz = S44 y los títulos dicen "Semana actual (2026-S44)". Precio / kg / gastos no cambian.
- [ ] G8.3 Elegir 2 semanas: títulos "2 semanas (… a …)". Elegir un tipo de compra: aparece en todos los títulos.

## Gate 3 · Validación (usuario)

**Antes:** cerrar Power BI Desktop **sin guardar** (si se guarda, Desktop pisa los cambios de la Fase 3 con la versión que tiene en memoria).
Reabrir el `.pbip` → **Actualizar**.

- [ ] G3.1 "KPIs semanales": todas las tarjetas muestran S43; China 6.627,55 · var −0,8% · móvil 6.676,85 — Orgánico 7.182,76 · var −0,5% · móvil 7.184,50 — premium 555,21 ($/kg) y 8,4%.
- [ ] G3.2 Kg carne, cabezas, rinde y % kg liquidado de la hoja de KPIs = los de "Tabla China" / "Tabla Orgánico" con el segmentador en 2026-S43 (China % kg liquidado ≈ 96,9%: la semana todavía no está liquidada del todo).
- [ ] G3.3 Tamaño de letra legible en el correo (probar con "Suscribirse → Enviar ahora" a uno mismo).
- [ ] G3.4 "Análisis semanal": tortas y matriz sin errores; desvíos positivos en rojo, negativos en verde, cercanos a 0 casi blancos.
- [ ] G3.5 Matriz (todas las semanas), mismos valores que en la tabla de abajo; tortas = porcentajes del mix de abajo.
- [ ] G3.6 **Aprobación** → publicar en Service y crear la suscripción (3B.5).

### Valores de referencia (calculados en SQL en la Fase 2)

- Matriz (todas las semanas):

| Comprador | China tropas | China precio | China desvío | Org. tropas | Org. precio | Org. desvío |
|---|---|---|---|---|---|---|
| Marcelo Malisani | 272 | 6.612,69 | +16,21 | 207 | 7.102,37 | −92,18 |
| Emmanuel La Blunda | 64 | 6.520,32 | −76,16 | 16 | 6.606,04 | −588,51 |
| Santiago Marchisio | 26 | 6.631,49 | +35,01 | 149 | 7.341,25 | +146,69 |
| C. Hacienda SJ | 1 | 6.792,53 | +196,05 | — | — | — |
| **Total** | **363** | **6.596,48** | | **372** | **7.194,55** | |

- Mix (todas las semanas): China A 0,7% · B 10,5% · C 43,2% · D 22,6% · E 21,3% · Decomiso 1,7% — Orgánico A 0,2% · B 8,4% · C 70,7% · D 12,5% · E 7,0% · Decomiso 1,1%.
