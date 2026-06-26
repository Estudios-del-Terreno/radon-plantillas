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
el generador (de hecho el generador puede cogerla de SharePoint en vez de rebuscar en Outlook).

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

### 3.3 Notion — base "campañas"
La base de datos administrativa se llama **"campañas"**: tiene varias páginas/tablas por tipo de
campaña (p. ej. farmacias) y una página general. Datos administrativos: **titular**, dirección,
comisión, conteo de detectores, estado, acta.

Colecciones conocidas (⚠️ **CONFIRMAR cuál corresponde a qué**, ver §6):

| Campaña / tabla | `collection://` | Visto en |
|---|---|---|
| Farmacias Gran Canaria (general) | `34fde4a3-1373-80a3-a79c-c86010370631` | generador de informes |
| Farmacias Gran Canaria Feb - Mar | `34fde4a3-1373-8134-a75a-000b2944ed1a` | procesado de actas |

- El `page_id` de cada ficha sale del campo `url` de la fila (los 32 caracteres hex finales).
- **La búsqueda más fiable es por número de comisión**, porque las fichas se nombran por el
  **titular**, no por el nombre comercial.

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
- Las comisiones de Tenerife u otros clientes (no farmacias GC) se descartan del lote de farmacias.

---

## 6. Discrepancias pendientes de confirmar con Luis

- **Dos `collection://` distintos para "Farmacias Gran Canaria"** (§3.3). Hay que confirmar si son
  dos tablas reales distintas (general vs. campaña "Feb - Mar") o si una quedó obsoleta. Mientras no
  se confirme, usa la que corresponda a la campaña concreta que se esté procesando.

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
