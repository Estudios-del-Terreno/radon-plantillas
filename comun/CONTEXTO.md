# CONTEXTO COMÚN — Radón / Estudios del Terreno S.L.

> **Fuente única de verdad** para los dos agentes de radón. Todo identificador, credencial,
> colección de Notion, gotcha de Outlook/SharePoint y regla de negocio compartida vive **aquí**.
> Si un dato de este archivo contradice a un archivo de rol, **gana este archivo**.
> Los archivos de rol (`roles/generador-informes.md`, `roles/procesado-actas.md`) solo contienen
> el procedimiento específico de cada tarea y **referencian** este documento para lo común.

---

## 1. Quién es Luis y qué hacemos

**Estudios del Terreno S.L.** (marca **Radón Canarias**, Tenerife) es una consultoría de medición
de gas radón que opera principalmente en **Canarias**. El laboratorio que analiza los detectores es
**Radonova**. El pipeline tiene dos etapas, una por agente:

1. **Procesado de actas** (`roles/procesado-actas.md`) — etapa de archivo, por **campaña**:
   descarga las actas de Radonova del correo, las cruza con la base de Notion, las sube a SharePoint
   y las enlaza en Notion. Deja las actas archivadas y Notion completo.
2. **Generador de informes** (`roles/generador-informes.md`) — etapa de entregable, por **centro**:
   a partir del PDF de Radonova genera el informe DOCX desde plantilla y ensambla el PDF combinado
   final para el cliente.

La etapa 1 es **upstream** de la 2: el acta que archiva el procesador puede ser la misma que consume
el generador. ✅ **Ya se puede DESCARGAR de SharePoint** con `download_shared_file` / `get_shared_item`
(resuelven un enlace de compartición de SharePoint y bajan los bytes). Por eso el generador puede
coger el acta de **dos sitios equivalentes** (mismo fichero): el campo **`Actas` de Notion** (enlace
SharePoint) o el **adjunto del correo de Radonova**. **Por defecto, pregunta a Luis cuál usar**
(ver §3.2 y generador §8.2).

Trabaja siempre en **español**.

---

## 2. Marco regulatorio (rige TODOS los informes)

- **Real Decreto 1029/2022** e **Instrucción IS-47 del CSN**.
- **300 Bq/m³** = nivel de referencia legal (umbral que decide plantilla positiva/negativa).
- **100 Bq/m³** = umbral de recomendación OMS / acción.
- **Municipios Canarias Zona II:** todos los de Gran Canaria y todos los de Tenerife
  **salvo San Juan de la Rambla y La Guancha**.
- Valores fijos de presupuesto: `impuesto_label = IGIC`, `impuesto_porcentaje = 7`.

---

## 3. Identificadores fijos

### 3.1 Cuenta Microsoft 365 (correo de trabajo)
- Correo: `info@estudiosdelterreno.com`
- `account_id`: `29ce514e-f778-4896-9022-333d0a217786.6031b76d-6c1b-48cb-8a8c-c2b4da035473`
- ⚠️ **Outlook ≠ Gmail.** Para correo usa SIEMPRE el MCP de Microsoft.
  `luisheratm@gmail.com` es solo personal; NUNCA se usa para trabajo.

### 3.2 SharePoint (destino de archivo de actas)
- Sitio: `estudiosdelterreno.sharepoint.com/sites/geotecnia` (aparece como "Estudios Del Terreno, S.L.")
- Biblioteca: **IT**
- `drive_id` de IT: `b!WPWtbG7mkECwK4v24TUZYKAWPQc37cRIijRd_7KXeeUW6PLSPpU9QakXXksxzZw8`
  - Si falla, re-resuélvelo: `sharepoint_resolve_site` → `sharepoint_list_drives` → coge el drive "IT".
- ⚠️ Esta cuenta **no tiene OneDrive personal** (`/me/drive` da 404). Usa SIEMPRE SharePoint.
- ✅ **DESCARGA de SharePoint (actualizado jun 2026):** ya funciona con
  **`download_shared_file`** (y **`get_shared_item`** para resolver metadatos) a partir del **enlace
  de compartición** que guarda Notion en el campo `Actas`
  (`https://estudiosdelterreno.sharepoint.com/:b:/s/geotecnia/<token>`). Resuelven el enlace vía
  Graph `/shares` y bajan los bytes. ⚠️ `get_file` **sigue dando 404** (usa `/me/drive`) y
  `search_files` devuelve `download_url: null` — **no** los uses para descargar; usa
  `download_shared_file`.
- Herramientas SharePoint disponibles: descarga (`download_shared_file`, `get_shared_item`),
  subida/gestión (`sharepoint_upload_file`, `sharepoint_create_link`, `sharepoint_list_drives`,
  `sharepoint_resolve_site`).
- **Estructura de carpetas:** `IT/campañas/<campaña>/<Ficha>/` contiene el acta `<comisión>.pdf` y
  (tras el generador) el `Informe_*.docx` y `Informe_*.pdf`. La carpeta `<Ficha>` = título de la
  ficha, salvo que el procesador haya tenido que limpiar caracteres prohibidos (p. ej.
  "José Mendoza y Juan Mendoza **C.B.**" → carpeta "...**CB**", sin punto final). Resuelve la carpeta
  real con `get_shared_item` del acta si dudas.
- ⚠️ **Subida y límite de ruta Windows (MAX_PATH 260):** `sharepoint_upload_file` lee el fichero
  **local**; si la ruta local supera ~260 caracteres falla con `No such file or directory` (aunque
  el fichero exista). Sube desde un **nombre local corto** (el nombre destino en SharePoint sí puede
  ser largo y descriptivo).

### 3.3 Notion — base "campañas"

Hay **DOS niveles** y es crítico no confundirlos:

1. **Índice de campañas** — base titulada **CAMPAÑAS**, solo columnas `Name` + `Tags`. Sirve para
   *localizar* una campaña; **no** contiene datos de fichas.
   - Fetch por: `collection://34fde4a3-1373-80a3-a79c-c86010370631`
   - (Su data source real es `collection://34fde4a3-1373-80f4-891c-000b60bac8d4`.)

2. **Tabla de fichas de una campaña** — aquí viven los datos reales que usan los dos agentes:
   `Ficha` (título), `Direccion`, `N* Comision`, `N* detectores`, **`Acta recibida`** (checkbox —
   este es el nombre real del campo, NO "Acta encontrada"), `Actas` (file), `Stage` (con estados
   como "Recogido"), `Municipio`, fechas, `Informe DOCX/PDF borrador`…
   - ⚠️ **El ID de esta tabla es PROPIO de cada campaña.** No lo hardcodees: resuélvelo cada vez
     (busca la campaña por nombre, o desde el índice CAMPAÑAS) y usa su `collection://`.
   - **Campaña actual (ejemplo):** Farmacias GC "Feb - Mar" (la tabla aparece titulada
     "Applications (1)") = `collection://34fde4a3-1373-8134-a75a-000b2944ed1a`.

- ⚠️ Las consultas SQL de los agentes (`SELECT "Ficha","Direccion","Acta recibida"…`) **solo
  funcionan contra la tabla de fichas (nivel 2)**, nunca contra el índice CAMPAÑAS (nivel 1).
- El `page_id` de cada ficha sale del campo `url` de la fila (los 32 caracteres hex finales).
- **La búsqueda más fiable es por número de comisión** (`N* Comision`), porque las fichas se nombran
  por el **titular**, no por el nombre comercial.

### 3.4 Radonova — correo de actas
- Asunto: empieza por
  `Radonova Laboratories - Ya está disponible el resultado de tu medida de radón (NNNNNNN:1)`
- El número de **7 dígitos** `NNNNNNN` es la **comisión** — la clave que une correo ↔ PDF ↔ ficha.
- Cada acta suele llegar **duplicada** (dos `replyTo` distintos): es el mismo PDF, procesa una sola.
- Cada correo trae 2 adjuntos: el **PDF** (el acta) y un **PNG** de logo (~8984 bytes, ignóralo).
- La **dirección** del centro viene en el cuerpo, tras la línea "Dirección:".

---

## 4. "Acta" = el PDF de Radonova (regla transversal)

⚠️ **"Actas" = el PDF de resultados de Radonova**, el informe original del laboratorio.
**No** existen documentos separados de colocación/retirada que buscar en Outlook: el PDF de Radonova
**es** la sección de actas. Esto aplica a los dos agentes.

**Autoridad sobre el conteo de detectores:** el PDF de Radonova **siempre** manda en el número de
detectores. Si Notion muestra otro número, usa el del PDF sin preguntar.

**`CLIENTE_NOMBRE` = titular** (persona jurídica/física en registro), **nunca** el nombre comercial.

**Prioridad de datos administrativos:** en caso normal, los datos administrativos de Notion (sobre
todo direcciones) tienen prioridad sobre el PDF. Pero **cruza siempre comisión + conteo de
detectores** entre Notion y el PDF; si una ficha está claramente mal asignada (dirección/nombre que
no encajan y sin vínculo plausible), usa los datos del PDF.

---

## 5. Cruce por dirección (calle + número)

El correo de Radonova trae la **dirección**, no siempre el titular. **Cruza por calle + número**,
normalizando: sin acentos, en minúsculas, quitando prefijos de vía ("Calle/Avda/Paseo/CC/C/…").
El titular ayuda como respaldo, pero **la dirección manda**.

- Si una comisión no casa con ninguna ficha, **NO la fuerces**: repórtala como "sin ficha" y
  pregunta a Luis (puede ser de otro trabajo / otra isla).
- Las comisiones de Tenerife u otros clientes (no farmacias GC) se descartan del lote de farmacias
  POR DEFECTO — pero Luis a veces añade fichas sueltas (p. ej. "Casino puerto de la cruz" en la
  tabla de farmacias). Comprueba siempre la tabla antes de descartar: si la ficha existe (aunque
  sin dirección), procésala.

### 5.1 Fichas "genéricas" (sin dirección, con nombre por tipo de cliente)

Algunas fichas están guardadas en Notion **sin dirección** y con un título genérico que describe el
**tipo de cliente** (no el titular), p. ej. `Almacen`, `Libreria`, `Ortopedia`, `Estetica`,
`Casino puerto de la cruz`, `Copistería`, `GMR TF`. Para estas:

- El cruce por dirección **no aplica** (Direccion está vacía).
- Cruza por **tipo** comparando con el **nombre comercial** que aparece en la línea "Dirección:"
  del PDF: "Almacen Parque, …" → ficha `Almacen`; "Librería Bécquer, …" → ficha `Libreria`;
  "Ortopedia Martinez, …" → ficha `Ortopedia`; "Krésia Espacio de Salud, …" → ficha `Estetica`;
  "CASINO PUERTO DE LA CRUZ, …" → ficha `Casino puerto de la cruz`.
- Si el tipo es ambiguo o hay varias fichas genéricas del mismo tipo en el lote, **pregunta** a Luis.

---

## 6. Notas resueltas

- **Los dos `collection://` que circulaban no eran dos campañas:** uno era el índice **CAMPAÑAS**
  (Name + Tags, sin datos de fichas) y el otro la **tabla de fichas** real de la campaña actual.
  Aclarado en §3.3. Regla: para trabajar usa siempre la **tabla de fichas** y **resuelve su ID por
  campaña**, no lo hardcodees.

---

## 7. Gotchas compartidos (Outlook / Notion / SharePoint)

| Problema | Solución |
|---|---|
| `search_emails` no da el Graph id usable | Usa `list_emails` / `get_email` para el `id` real (search_emails solo da un webLink EWS) |
| `bodyPreview` sin la dirección | `search_emails(query="<comisión> dirección")` fuerza que aparezca en el preview |
| ID crudo de correo en `get_email`/`get_attachment` | Sustituir `+`→`_` y `/`→`-` (base64 URL-safe) |
| SQL de Notion "could not be parsed" | No metas en el SELECT columnas con `*` (p. ej. `N* Comision`) ni `ORDER BY` entre comillas |
| Campo `Actas` (file) no se borra con `""` | Da "File not found"; hay que sobrescribir con otra URL |
| 429 `rate_limited` en Notion | Tras ~20 escrituras seguidas; esperar ~30s y reintentar (lo previo sí se guardó) |
| PDF no está en disco al subir a SharePoint | Re-descargar del correo: localizar Graph id (`list_emails`) y `get_attachment` |
| SharePoint rechaza nombre de carpeta/archivo | Quitar caracteres prohibidos `" * : < > ? / \ |` del nombre (no afecta al enlace de Notion) |
| OneDrive `/me/drive` da 404 | Esta cuenta no tiene OneDrive personal; usar SIEMPRE SharePoint (§3.2) |
| Descargar un acta de SharePoint | ✅ Usa **`download_shared_file`** con el enlace del campo `Actas` de Notion (`get_file`/`search_files` NO sirven: 404 / `download_url` null). El adjunto del correo de Radonova es el mismo fichero (alternativa) |
| `sharepoint_upload_file` da `No such file or directory` (el fichero sí existe) | Ruta local supera MAX_PATH de Windows (260). Copia a un nombre local corto y sube desde ahí (el nombre destino puede ser largo) |
| Subir DOCX/PDF final a la ficha de Notion | Sube a `IT/campañas/<campaña>/<Ficha>/` (`sharepoint_upload_file`), crea enlace (`sharepoint_create_link`) o usa el `webUrl`, y pon el valor `file://<json url-encoded {"source":"<url>"}>` en la propiedad de archivo (`Informe DOCX borrador` / `Informe PDF final`) con `notion-update-page` |

---

## 8. Registro de detectores en Radonova (MCP `radonova-registro`)

Nueva etapa **upstream de todo el pipeline**: cuando se **colocan/retiran** los detectores, hay que
**registrarlos en "Mis Páginas" de Radonova** (`online.radonova.com`) — nº de detector, fecha de
inicio (y fin cuando se retiran) y ubicación. Esto es lo que dispara que el laboratorio analice y,
más tarde, emita el acta (PDF) que consumen los otros dos agentes.

**MCP:** `radonova-registro` (proyecto `C:\Users\luish\ai\radonova-registro-mcp`; conduce un navegador
con Playwright). Herramientas:
- **`radonova_registrar_medicion`** — parámetros: `comision`, `password`, `detectores[]`
  (`{numero, comienzo, fin?, planta?, localidad?, detalles?}` con fechas `YYYY-MM-DD`), opcionales
  `contacto`, `direccion`, `identificacionEdificio`, `reemplazarExistentes` (borra las filas
  existentes antes — necesario para corregir), `guardar` (pulsa GUARDAR TODO al final).
- **`radonova_leer_medicion`** — login de solo lectura: comisión, pedido, si es editable y detectores.
- **`radonova_vaciar_medicion`** — BORRAR TODO: elimina todas las filas de detectores y vacía contacto
  y dirección, para dejar la comisión limpia (o deshacer una prueba). Solo `comision` + `password`.

Las herramientas de registro/vaciado **abren el navegador visible** y **devuelven una captura** del
formulario al chat para que Luis corrobore el resultado.

**De dónde salen los datos:** de la **foto del albarán Radtrak³** que envía Luis. En el **recuadro
rojo** están `Comisión número` y `Contraseña` (son ESPECÍFICAS de cada comisión). A mano vienen los
detectores (nº de 9 dígitos + ubicación) y la fecha de colocación. Extrae esos datos de la imagen y
pásalos al tool.

⚠️ **Gotchas del portal** (confirmados en el código fuente de la SPA):
- **Autoguardado**: cada campo se guarda al perder el foco. Rellenar = guardar; **no hay dry-run**.
- **Borrar** una fila ya guardada abre un **`confirm()` nativo**; el MCP lo acepta automáticamente.
  El nº de detector, una vez guardado, es de **solo lectura** → para corregirlo, `reemplazarExistentes`.
- **Rate-limit por IP** ("Too many requests, wait N seconds") si se hacen muchos logins seguidos;
  el MCP espera y reintenta, pero **no encadenes logins** innecesarios.
- Convención acordada con Luis: la **etiqueta escrita** va en **`localidad`**, `planta` vacía por
  defecto; **`fin` vacío** si el albarán no trae fecha de retirada. La comisión manda sobre Notion.
- Cruza el **nº de comisión** de la foto con la ficha de Notion (`N* Comision`, §3.3) para dejar
  constancia si procede.
