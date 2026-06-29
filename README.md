# radon-plantillas — Plantillas e instrucciones de radón (Estudios del Terreno S.L.)

Este repositorio es la **fuente única de verdad** para los agentes de radón de Luis. Contiene las
plantillas (informes, presupuestos, acreditaciones, contraportada) y las **instrucciones de cada
agente**, que se editan aquí en git y mandan sobre cualquier memoria o instrucción del proyecto.

## Estructura

```
radon-plantillas/
├── README.md                          ← este índice
├── comun/
│   └── CONTEXTO.md                    ← contexto compartido: identificadores, colecciones Notion,
│                                         SharePoint, gotchas, cruce por dirección, marco legal
├── roles/
│   ├── procesado-actas.md             ← AGENTE 1: archiva el lote de actas de una campaña
│   │                                     (Outlook → Notion → SharePoint)
│   ├── generador-informes.md          ← AGENTE 2: genera el informe DOCX y el PDF combinado final
│   └── generador-facturas.md          ← AGENTE 3: genera la factura DOCX y actualiza Notion
│
├── plantilla-informe-negativo.docx    ← informe cuando NINGÚN detector supera 300 Bq/m³
├── plantilla-informe-positivo.docx    ← informe cuando ALGÚN detector supera 300 Bq/m³
├── presupuesto-cte.docx               ← plantillas de presupuesto
├── presupuesto-laboral.docx
├── presupuesto-rapidos.docx
├── plantilla-factura.docx             ← plantilla de factura (entrega a cuenta / resto / completa)
├── acreditaciones/                    ← carpeta con varios PDFs (orden alfabético)
└── Contraportada.pdf                  ← última página del PDF combinado
```

## Los agentes

| Agente | Archivo de rol | Qué hace | Granularidad |
|---|---|---|---|
| **Procesado de actas** | `roles/procesado-actas.md` | Descarga las actas de Radonova del correo, las cruza con Notion, las sube a SharePoint y las enlaza | Por **campaña** (lote) |
| **Generador de informes** | `roles/generador-informes.md` | Genera el informe DOCX desde plantilla y ensambla el PDF combinado para el cliente | Por **centro** (uno) |
| **Generador de facturas** | `roles/generador-facturas.md` | Genera la factura DOCX desde `plantilla-factura.docx`, la sube a SharePoint y actualiza Notion | Por **factura** |

El procesado de actas es **upstream** del generador: archiva las actas y completa Notion; el generador
luego consume esos datos. (El acta archivada en SharePoint y la del correo de Radonova son el **mismo fichero**; pero con las herramientas actuales **no se puede descargar de SharePoint**, así que el generador coge el PDF del **adjunto del correo** — ver `comun/CONTEXTO.md` §3.2 y §7.)

## Cómo arranca cada agente

Cada proyecto de Claude clona este repo y, según su rol, lee:

1. **Siempre primero:** `comun/CONTEXTO.md` (lo compartido).
2. **Su archivo de rol:** `roles/procesado-actas.md`, `roles/generador-informes.md` **o** `roles/generador-facturas.md`.

El contexto común + el archivo de rol forman juntos el system prompt de ese agente. **Mandan sobre
cualquier otra instrucción**: son la fuente de verdad y se editan aquí.
