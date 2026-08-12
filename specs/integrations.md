# Integrations — Aroma Cafetería (MVP mocks)

Todo lo descrito aquí es mockeado en el cliente. No hay backend, base de datos ni credenciales reales. El objetivo de este documento es que la implementación sea explícita y que una futura integración real (v2) sea un reemplazo acotado, no una reescritura.

## 1. Pagos (mock)

No se procesa ningún pago real. No hay llamadas a ningún proveedor de pagos.

**`/checkout` — formulario:**
- Datos de contacto/envío: `name`, `email`, `address` (calle, ciudad, código postal — un solo textarea o campos simples, a discreción del implementador).
- Datos de tarjeta (solo visual, sin validación real contra ningún proveedor): `cardNumber`, `expiry`, `cvc`. Puede aplicarse validación de formato básica en cliente (largo, solo dígitos) pero **nunca** se envía a ningún servicio ni se verifica que sea una tarjeta válida.
- Botón "Confirmar pedido".

**Al confirmar:**
- No hay llamada a red ni persistencia (ni localStorage, ni backend).
- Se genera un número de orden client-side (ej. `ORD-` + timestamp o random alfanumérico) y se navega a `/checkout/confirmation` pasando los datos necesarios (vía state/query/context — a discreción del implementador).
- `/checkout/confirmation` muestra: número de orden generado, resumen de items comprados, total, y un mensaje de éxito. Al recargar la página esta información puede perderse (no hay persistencia) — es aceptable para el MVP.

**Nota v2:** aquí (en el submit de `/checkout`) es donde se integraría Stripe Checkout / Payment Intents, reemplazando el formulario de tarjeta mock por un redirect o Elements embebido.

## 2. Envíos

Tarifa plana simple, sin cálculo por zona, peso ni carrier real:

- Envío estándar: **$5.00** flat, aplicado en `/cart` y `/checkout` como línea aparte del subtotal.
- Envío gratis si el subtotal es **≥ $50**.
- Regla mostrada en el resumen del carrito (ej. "Envío gratis en compras sobre $50 — te faltan $X" cuando aplica).
- No hay selección de método de envío, no hay estimación de fecha de entrega, no hay integración con carrier.

## 3. Inventario

`architecture.md` no define un campo de stock en `Product`; se usa el default `inStock: boolean` indicado en este spec.

- Campo opcional en `data/products.json` (y en `lib/types.ts`): `inStock?: boolean`. Si el campo está ausente, se asume `true` (disponible).
- **UI:**
  - `ProductCard` y la página de detalle (`/products/[slug]`) muestran un badge "Agotado" cuando `inStock === false`.
  - El botón `AddToCartButton` se deshabilita (y muestra texto "Agotado" en vez de "Agregar al carrito") cuando `inStock === false`.
  - No hay decremento de stock al agregar al carrito o completar el checkout — el valor es estático, definido a mano en `products.json`.
- No hay cantidades por SKU (`stock: number`) en este MVP; es un flag binario.

## 4. Fuera de alcance explícito (no incluido en el MVP)

- Cuentas de usuario / login / autenticación.
- Historial de pedidos persistente (nada se guarda tras la confirmación).
- Notificaciones por email (confirmación de orden, envío, etc.).
- Webhooks de pago (Stripe u otro proveedor).
- Procesamiento real de pagos, cálculo de impuestos, y validación de tarjeta.
- Cálculo de envío por zona/peso/carrier real.
- Decremento real de inventario / gestión de stock por cantidad.

Estos puntos no deben marcarse como bugs o incompletos en revisión — son mocks intencionales del MVP.
