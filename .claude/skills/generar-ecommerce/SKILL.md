---
name: generar-ecommerce
description: Genera un sitio ecommerce completo (Next.js + Tailwind) a partir de un briefing de negocio, siguiendo el pipeline briefing → 4 subagentes de spec en paralelo → aprobación humana → implementación → revisión con loop de corrección. Usar cuando el usuario pida crear, generar o construir una tienda/ecommerce nueva.
---

Este skill te convierte en el orquestador de un pipeline multi-agente que genera un sitio ecommerce funcional. Sigue estos pasos en orden — no te saltes la pausa de aprobación.

## Paso 1 — Reunir el briefing

Si el usuario pasó una ruta a un JSON de briefing como argumento, léelo con `Read`. Si no, pregúntale (o infiere de la conversación) estos campos como mínimo:

- `name` — nombre del negocio
- `industry` — rubro
- `tone` — tono de marca (ej. "cálido y artesanal", "minimalista y tech")
- `colors` — 2-3 colores sugeridos (hex o descripción)
- `products` — lista de productos (nombre, descripción breve, precio) — si el usuario no tiene una lista, dile que puedes inventar un catálogo coherente con la industria y sigue adelante

`briefing.example.json` en la raíz del proyecto es la plantilla de referencia del shape esperado.

No avances al paso 2 sin al menos `name`, `industry` y `tone` — pregunta lo que falte.

## Paso 2 — Generar los 4 specs en paralelo

Invoca el tool `Agent` **cuatro veces en un solo mensaje** (para que corran en paralelo), una por cada subagente, pasando el briefing completo como contexto a cada uno:

- `ecommerce-architect` → debe producir `specs/architecture.md`
- `ecommerce-designer` → debe producir `specs/design-tokens.json`
- `ecommerce-content` → debe producir `specs/content.md` y `specs/products.json`
- `ecommerce-integrations` → debe producir `specs/integrations.md`

Espera a que los 4 terminen antes de continuar.

## Paso 3 — Pausa de aprobación (obligatoria)

Lee los 4 archivos generados en `/specs` y muéstrale al usuario un resumen breve (no el contenido completo) de cada uno: rutas principales, paleta de colores, tono del copy, y el modelo de pagos/envíos elegido. Pídele aprobación explícita antes de continuar, dejando claro que puede editar cualquiera de los archivos en `/specs` directamente antes de aprobar.

**No invoques al implementador hasta que el usuario confirme.** Si pide cambios, aplícalos (tú mismo con `Edit`, o re-invocando el subagente correspondiente) y vuelve a pedir aprobación.

## Paso 4 — Implementación

Una vez aprobados los specs, invoca el tool `Agent` con `ecommerce-implementer`. Este subagente lee todo `/specs`, genera (o actualiza) el sitio Next.js completo en `/site`, y corre `npm install`/`npm run build` para verificar que compila antes de terminar.

## Paso 5 — Revisión y loop de corrección

Invoca el tool `Agent` con `ecommerce-reviewer`. Este subagente audita `/site` contra los specs y escribe `reports/review.md` con veredicto `PASS` o `FAIL`.

- Si `PASS`: el sitio queda listo. Dile al usuario que puede correrlo con `cd site && npm run dev`, y resume brevemente qué se generó (rutas principales, catálogo).
- Si `FAIL`: vuelve a invocar `ecommerce-implementer` — este subagente lee automáticamente `reports/review.md` y corrige lo señalado — y luego vuelve a invocar `ecommerce-reviewer`. Repite este ciclo hasta un máximo de **3 iteraciones**.
- Si tras 3 iteraciones sigue en `FAIL`: detente, muéstrale al usuario el último `reports/review.md` y pregúntale cómo quiere continuar (más iteraciones, corrección manual, o aceptar el sitio con los problemas conocidos). No sigas loopeando sin confirmación.

## Notas para el orquestador

- Los subagentes de spec (`ecommerce-architect`, `ecommerce-designer`, `ecommerce-content`, `ecommerce-integrations`) solo escriben dentro de `/specs` — no deberían tocar `/site`.
- `ecommerce-implementer` solo debería tocar `/site` (y leer `/specs` y `/reports`).
- `ecommerce-reviewer` solo debería tocar `/reports` (y leer `/specs` y `/site`).
- Si el usuario pide generar un sitio nuevo cuando ya existe contenido en `/specs`, `/site` o `/reports` de una corrida anterior, confírmale si quiere sobrescribir antes de empezar.
