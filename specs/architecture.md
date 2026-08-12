# Architecture — Aroma Cafetería

## 1. Routes (App Router)

| Route | Renders |
|---|---|
| `/` | Home: hero, brand intro, featured products, category links |
| `/products` | Full catalog grid, filterable by category via query param (`?category=`) |
| `/products/[slug]` | Single product detail page (description, price, add to cart) |
| `/cart` | Cart summary: line items, quantities, subtotal, checkout CTA |
| `/checkout` | Mocked checkout form (shipping info) + order summary, no real payment |
| `/checkout/confirmation` | Static mocked order confirmation page |

## 2. Data Model

```ts
// lib/types.ts

export type CategorySlug = "granos" | "equipo" | "bebidas";

export interface Category {
  slug: CategorySlug;
  name: string;
}

export interface Product {
  id: string;
  slug: string;
  name: string;
  description: string;
  price: number;
  category: CategorySlug;
  image?: string;
  featured?: boolean;
}
```

- `id` and `slug` are derived at build/seed time from `name` (not present in raw briefing data, must be generated when producing `data/products.json`).
- `image` is optional (briefing has no images; placeholder used if absent).
- `featured` optional, used to pick items for the home page.

## 3. Folder Structure (`/site`)

```
site/
  app/
    page.tsx                        # /
    products/
      page.tsx                      # /products
      [slug]/
        page.tsx                    # /products/[slug]
    cart/
      page.tsx                      # /cart
    checkout/
      page.tsx                      # /checkout
      confirmation/
        page.tsx                    # /checkout/confirmation
    layout.tsx
    globals.css
  components/
    Header.tsx
    Footer.tsx
    ProductCard.tsx
    ProductGrid.tsx
    CategoryFilter.tsx
    CartItem.tsx
    CartSummary.tsx
    AddToCartButton.tsx
  lib/
    types.ts
    cart-context.tsx                # React context provider for cart state
    products.ts                     # helpers: getAllProducts, getProductBySlug, getCategories
  data/
    products.json
```

No `db/`, `api/`, or `auth/` folders — out of scope for this MVP.

## 4. Data Source

- Products load from static `data/products.json`, imported/parsed by `lib/products.ts` at build time (no backend, no database).
- Cart state is kept client-side via a React Context (`lib/cart-context.tsx`) wrapping the app in `layout.tsx`, persisted to `localStorage` so the cart survives page reloads without any server — simplest option that meets the mocked-checkout requirement without introducing state-management dependencies.
