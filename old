# Ana Pizarro — Inventario · Contexto del proyecto

Última actualización: 16 nov 2026

## Qué es
App web de inventario y ventas de telas para Ana Pizarro, con tres bodegas/puntos: **Bodega Central**, **Bodega 140** (1er y 2º piso) y **Bodega 79**.
Todo el programa está en un solo archivo: `index.html` (~12.300 líneas: HTML, CSS y JavaScript juntos).
Los datos viven en **Supabase**, proyecto `bwppzypyaidcsgeiclua`. El archivo solo trae la clave pública (anon).

## Archivos de este proyecto
| Archivo | Qué es |
|---|---|
| `index.html` | El programa. Es la versión del 16 nov 2026 (antes `Descargas\index.html`) más la reparación de Anomalías. |
| `supabase/fiados.sql` | Crea las tablas `fiados` y `fiado_pagos` (créditos) con permisos abiertos (RLS `acceso_total`). |
| `supabase/historial.sql` | Agrega `usuario`, `lote_id` y `creado` (hora real del servidor) a ventas, traslados, entradas, rollos y correcciones. Se puede correr varias veces sin problema. |
| `supabase/asistente-ia.ts` | Función de Supabase `asistente-ia`: guarda la clave de la IA fuera del index, llama a Claude y pone un tope diario (tabla `ia_uso`). |

Las versiones anteriores (`index_111` … `index_128`) siguen en Descargas.

## Base de datos (Supabase)
**Tablas:** `inventario`, `telas`, `referencias`, `lotes` (rollos), `historial_rollos`, `alertas_rollos`, `ventas`, `entradas`, `traslados`, `correcciones`, `conteos`, `revisiones`, `solicitudes`, `avisos`, `anomalias`, `fiados`, `fiado_pagos`, `asesoras`, `configuracion`, `log_cambios`, `ia_uso`.

**Funciones (RPC) de stock:** `descontar_stock`, `agregar_stock`, `trasladar_stock`. Son las únicas que cambian el stock.

## Usuarios y roles
- **Admin:** ve todo (Dashboard, Analíticas con PIN, Gestión, Anomalías, Verificaciones, Caja histórica…).
- **Asesora:** entra directo a *Registrar venta*. En **modo beta** solo ve: venta, ubicación, traslados, entradas y créditos.
- Los administradores están escritos en el código (`USUARIOS_SISTEMA`). Las asesoras vienen de la tabla `asesoras` (`syncUsuarios`).

## Pantallas
Dashboard · Alertas · Registrar venta · Inventario · Ubicación · Calendario · Analíticas · Precios · Entradas · Traslados · Referencias · Asesoras · Rendimiento · Créditos (fiados) · Anomalías · Verificaciones · Avisos · Caja · Gestión · Historial · Asistente IA.

## Reglas del negocio (tomadas del código)
- **Ventas:** primero se guardan en Supabase y solo después cambia la pantalla. Si la venta tiene un rollo, se le descuentan los metros a ese rollo.
- **Pago combinado:** se guarda con el primer método. La división va en la observación (`"Efectivo: $X + Nequi: $Y"`) y el cierre de caja la reparte.
- **Créditos (fiados):** la tela sale del stock el día de la venta, pero la plata cuenta en la caja el día que se cobra. Cada abono tiene su fecha, su método y su caja. No se puede abonar más que el saldo. Si una venta a crédito se corrige o se elimina, el crédito se ajusta con ella.
- **Venta sin stock:** no descuenta nada. Se envía a **Anomalías** para que el admin suba el stock y registre la venta.
- **Anomalías:** la asesora reporta lo que no cuadra sin cambiar nada. El admin decide: poner el stock, registrar la venta, marcar como revisada o descartar.
- **Ventas atrasadas:** la asesora las puede registrar, y al admin le llega un aviso.
- **Caja de cobro:** cada venta guarda de dónde salió la tela (`punto`) y en qué caja entró la plata (`punto_cobro`).

## Verificación y estado (sesión del 12–16 nov 2026)
- ✅ **Anomalías no cargaba.** Causa: error de escritura `_escetiquetaBod(...)` en `renderAnomalias`, que rompía la lista y la dejaba en "Cargando…". Se corrigió a `_esc(etiquetaBod(...))`. Se probó: carga los reportes pendientes (había 33 el 12 nov). **Ojo:** el index del 16 nov todavía traía el error; ya está corregido en esta copia.
- ⚠️ **Créditos no se guardan.** El código del programa funciona: una venta a crédito simulada (sin escribir nada) hizo todo el proceso bien. Pero en Supabase existe un solo crédito (#1, de prueba, del 29 oct) y ninguna venta con pago "fiado" después de esa fecha. Lo más probable es que Supabase esté rechazando los créditos nuevos. **Falta:** ver qué mensaje sale en pantalla al registrar un crédito, o hacer una prueba controlada en la base de datos.
- ✅ **"Ingresan tela y no se registra" (17 nov).** Causa principal: lo que hace una asesora con los rollos ("Sincronizado con los rollos", rollo nuevo como tela nueva, corrección de rollo) no cambia el stock. Queda como **solicitud pendiente** en Verificaciones hasta que un admin la apruebe. El 17 nov había **64 pendientes** (la más vieja del 16 oct) y **19 eran repetidas**, porque la asesora no veía ningún cambio y volvía a enviar lo mismo. Arreglos:
  - En *Registrar venta*, la asesora ve "⏳ Ya hay N cambios esperando aprobación" para esa tela, color y punto.
  - Si intenta enviar una solicitud idéntica, el programa no la envía y le avisa que ya está enviada.
  - En *Verificaciones*, las repetidas salen marcadas "REPETIDA". Si se aprueban todas, el stock se suma varias veces.
  - *Registrar entrada*: antes sumaba la tela en pantalla **antes** de guardar, así que si fallaba, al recargar desaparecía. Ahora suma en pantalla solo lo que la base de datos confirma y avisa si se guardó solo una parte. Además la fecha guardada ya no salta al día siguiente después de las 7 p. m. (usaba la hora UTC) y respeta la fecha del formulario. Si el color se escribió a mano y no existe, lo dice claramente.
- ✅ **Cambio de reglas (17 nov): las asesoras ya no piden aprobación para el stock.** Sus ajustes de stock (sincronizar con rollos, escribir el stock exacto, corregir un rollo "y mover el stock") y la tela nueva al crear un rollo se aplican de una. A cada uno le llega al admin un **aviso** en *Avisos*: "📦 Cambio de stock: TELA COLOR en PUNTO: antes → después (±m) — motivo".
  - **Control de inventario:** en *Revisar asesoras* hay un filtro nuevo, **"📦 Solo cambios de inventario"** (activo por defecto): entradas, ajustes, traslados, rollos y correcciones, sin ventas. Cada movimiento se marca "Está bien" o "Hay un problema" (tabla `revisiones`, que antes no se había usado nunca).
  - **Solicitudes viejas:** *Verificaciones* ahora solo guarda las de antes. El botón **"Resolver todas con los datos de hoy"** agrupa por tela, color y bodega, y compara el stock de hoy con lo que suman sus rollos hoy. Antes de cambiar nada muestra una vista previa:
    - **Ya al día:** stock = rollos. Se cierra sin cambiar nada.
    - **Se ajusta:** misma dirección que lo pedido y de tamaño parecido (hasta 1,5× o +1 m). El stock queda igual a los rollos de hoy, una sola vez aunque haya repetidas.
    - **Contar en físico:** dirección contraria, o la diferencia es mucho mayor que lo pedido. Se cierra (rechazada) sin tocar el stock y se puede imprimir la hoja de conteo.
    - Una "tela nueva" (carga) que no esté al día queda para aprobar a mano, porque así se crea su rollo.
    - Simulación con las 64 del 17 nov (todas de Viviana, del 16 oct al 17 nov, 41 telas): **18 telas al día** (26 sol.), **16 se ajustan** (22 sol.) y **7 para contar** (16 sol.: ENTG BLANCO-00, NVL VERDE ESMERALDA CON DORADO, LM MORADO-44, SDI VINOTINTO-43, CHAM AZUL PETROLEO-19, PLCH NEGRO-05 y ARMNV NEGRO-05 en 140 · 2º piso). Todavía no se ha ejecutado en la base real.
  - Las fechas de traslados, cargas, aprobaciones y ediciones de color usaban la hora UTC (después de las 7 p. m. quedaban con la fecha del día siguiente). Ya usan la fecha de Colombia.
- ✅ **Informe de cambios (17 nov), pantalla nueva solo para admin, en el menú de Administración.** Muestra cuatro columnas: **Ingresadas** (entradas), **Modificadas** (ajustes de stock), **Corregidas** (rollos corregidos o cerrados a mano y ventas corregidas o eliminadas) y **Trasladadas**. Hay una fila por tela y color, con el último día, quién lo hizo y la cantidad neta. El botón **+** abre el detalle.
  - En *Corregidas* no se cuentan los rollos nuevos o etiquetados, los cierres por venta normal ni la verificación automática al vender.
  - Periodos: hoy, esta semana, **este mes** (por defecto), mes pasado o un rango de fechas. Filtro "Solo asesoras" y buscador.
  - **Imprimir informe** (resumen) o **con detalle** (cada movimiento).
  - **Hoja de conteo:** se marcan las telas con la casilla (o "escoger todas" en una columna) y se imprime una hoja horizontal para contar en físico, con espacios por bodega, diferencia, observaciones y firmas. Al imprimir pregunta si se muestran las cantidades del sistema o si es un conteo "a ciegas".
  - *Gestión Admin → Conteo cíclico* también tiene ahora **Imprimir hoja**.
  - Mes de nov (hasta el 17): 215 ingresadas, 77 modificadas, 183 corregidas y 25 trasladadas.
- ⚠️ **Editar color en Referencias** (mensaje "Color actualizado SOLO en esta pantalla", línea ~10871) todavía puede sumar metros solo en pantalla cuando no hay conexión. No se revisó.
- ⚠️ **Funciones que no existen:** el botón "Guardar cambios" (línea ~1077) llama a `guardarEditMaestro()`. También se llaman `loadColors`, `previewColors` y `sugerirNumColor`, que no están definidas. Esas partes pueden fallar.

## Seguridad (importante antes de publicar la app en internet)
- Las contraseñas de los administradores están escritas en texto plano dentro de `index.html`. Quien abra el archivo o el código de la página las puede ver.
- Las tablas tienen permisos abiertos (`USING (true)`), así que cualquiera que tenga la clave anon (que va dentro del index) puede leer y escribir.
- Mientras el archivo esté solo en los computadores de la tienda, el riesgo es bajo. Si se publica en una dirección web, conviene pasar el inicio de sesión a **Supabase Auth** y cerrar los permisos de las tablas.

## Celular
- El index ya tiene metas de "app instalable" (manifest incrustado, `apple-mobile-web-app-capable`) y CSS responsive básico (menú lateral angosto por debajo de 900px).
- Revisado a 375px de ancho: el contenido de *Registrar venta* se sale por la derecha y el menú lateral ocupa espacio. Falta adaptar la pantalla al celular.

### Opciones para operarlo desde el celular
1. **App web instalable (PWA), recomendada.** Se publica `index.html` en un hosting gratuito con HTTPS (Netlify, Cloudflare Pages o GitHub Pages). En el celular se abre la dirección y se toca "Agregar a pantalla de inicio": queda con su ícono y a pantalla completa, en Android y en iPhone. Cada cambio se publica una vez y les llega a todos. Hay que adaptar el diseño al celular (menú inferior, formularios a lo ancho) y antes resolver la seguridad.
2. **App Android (.apk) con Capacitor.** Envuelve la misma página en una app instalable. Necesita Android Studio y reinstalar la app en cada cambio. Para iPhone hace falta una Mac y una cuenta de Apple Developer.
3. **Abrir el archivo en el celular.** No sirve bien: los navegadores de celular abren mal los archivos locales y no se actualizan solos.
