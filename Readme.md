# E-Commerce Stack — Engineer's Cheat Sheet

## 1. Glossary (one line each)

| Term | Definition |
|---|---|
| **Ecommerce Platform** | All-in-one SaaS: hosting + storefront + checkout + admin (Shopify, BigCommerce). |
| **Ecom Builder** | Drag-and-drop UI to assemble a store without code (Shopify themes, Wix, Webflow Ecom). |
| **Headless** | Frontend decoupled from backend; you build UI, backend exposes API only. |
| **WooCommerce** | Open-source ecom plugin on top of WordPress. Self-hosted, PHP/MySQL. |
| **Whop** | Digital-product / membership commerce platform, creator-economy focused (licenses, Discord roles, SaaS access). |
| **Stripe** | Payment infrastructure (API-first): charges, subscriptions, payouts, tax, fraud. |
| **Ecommerce Engine** | The backend core: catalog, cart, pricing, inventory, order logic. |
| **Ecommerce API** | Interface exposing engine functions (products, cart, checkout) to any frontend. |
| **Storefront** | Customer-facing site/app that consumes the engine/API. |
| **Landing Page** | Single-purpose acquisition page (ad → conversion), not full catalog. |
| **Product Page (PDP)** | Page for one SKU: images, price, variants, add-to-cart. |
| **CRM** | Customer data + lifecycle system: emails, segments, LTV, support history. |

---

## 2. Architecture A — Monolithic vs Headless

```mermaid
flowchart LR
    subgraph Monolithic["Monolithic Platform"]
        M1[Shopify / WooCommerce] --> M2[Theme = Storefront]
        M2 --> M3[Built-in Checkout]
        M3 --> M4[Built-in Admin/CRM]
    end

    subgraph Headless["Headless / Composable"]
        H1[Ecommerce Engine] -->|API| H2[Storefront: Next.js / Remix]
        H1 -->|API| H3[Mobile App]
        H1 -->|API| H4[Landing Pages: Webflow]
        H2 --> H5[Stripe / Payment Layer]
        H1 --> H6[CRM: HubSpot / Klaviyo]
    end
```

**Rule of thumb:** Monolithic = fast to launch, hard to customize. Headless = slow to launch, unlimited customization.

---

## 3. Stack Combos (pick one)

| Combo | Engine | Storefront | Payments | CRM | Best for |
|---|---|---|---|---|---|
| **1. Turnkey** | Shopify | Shopify theme | Shopify Payments / Stripe | Shopify + Klaviyo | Small biz, fast launch |
| **2. Open-source** | WooCommerce | WordPress theme | Stripe/PayPal plugin | Mailchimp/WP CRM | Budget, content+shop hybrid |
| **3. Headless Composable** | Shopify/Commerce Layer/Medusa (API) | Next.js custom frontend | Stripe | Segment → HubSpot | Scale, custom UX, multi-channel |
| **4. Creator/Community** | Whop | Whop-hosted storefront | Stripe/Whop payments | Whop native + Discord | Digital goods, memberships, SaaS licenses |

---

## 4. Alternative Business Design System — Creator/Access Commerce (Whop-style)

Not a "product catalog" model — an **access/license** model. Different mental model from classic ecom:

```mermaid
flowchart TD
    A[Landing Page] --> B[Product Page = Offer Page]
    B --> C{Purchase Type}
    C -->|One-time| D[License Key / File Delivery]
    C -->|Subscription| E[Recurring Access via Stripe]
    D --> F[Access Gate]
    E --> F[Access Gate]
    F --> G[Discord Role / App Login / Course Portal]
    F --> H[CRM: churn, renewal, upsell tracking]
```

Key diff vs classic storefront:
- No shipping/inventory concept.
- "Product" = access rights, not physical SKU.
- CRM tracks **access lifecycle** (active/expired/refunded), not shipping status.

---

## 5. Request Flow — Storefront to Engine

```mermaid
sequenceDiagram
    participant U as User
    participant LP as Landing Page
    participant PDP as Product Page
    participant API as Ecommerce API
    participant PAY as Stripe
    participant CRM as CRM

    U->>LP: Click ad
    LP->>PDP: Redirect
    PDP->>API: GET /product/:id
    API-->>PDP: price, stock, variants
    U->>API: POST /cart
    U->>API: POST /checkout
    API->>PAY: create payment intent
    PAY-->>API: success
    API->>CRM: sync order + customer
    API-->>U: order confirmation
```

---

## 6.1 Design System — Next.js + Shopify (Headless)

```mermaid
flowchart TB
    subgraph Frontend["Next.js App"]
        LP[Landing Page - SSG]
        PDP[Product Page - ISR]
        CART[Cart - Client State]
    end

    subgraph Shopify["Shopify Backend"]
        SA[Storefront API - GraphQL]
        ADM[Admin - Catalog/Orders]
        CHK[Shopify Checkout]
    end

    LP --> PDP
    PDP -->|fetch products| SA
    CART -->|fetch cart| SA
    CART -->|redirect| CHK
    CHK -->|webhook: order created| ADM
    ADM --> CRM[CRM / Klaviyo]
    SA -.->|revalidate on webhook| PDP
```

- Frontend = pure presentation, no business logic.
- Shopify keeps: inventory, tax, checkout, payments.
- Trade-off: you don't own checkout UX (unless Shopify Plus + Checkout Extensibility).

---

## 6.2 Design System — WordPress + WooCommerce

```mermaid
flowchart TB
    subgraph WP["WordPress Monolith"]
        TH[Theme - PHP templates]
        WC[WooCommerce Core]
        DB[(MySQL)]
        PL[Plugins: SEO, Reviews, Shipping]
    end

    U[User] --> TH
    TH -->|render PDP/LP| WC
    WC --> DB
    WC --> PL
    WC -->|checkout| GW[Payment Gateway Plugin - Stripe/PayPal]
    GW --> WC
    WC --> EM[Email/CRM Plugin - Mailchimp/WP Fusion]
```

- Everything server-rendered, single codebase, single DB.
- Fast to extend via plugins, slow at scale (plugin bloat = perf risk).
- No native API-first mindset unless you enable WooCommerce REST/GraphQL.

---

## 6.3 Design System — Whop (Access Commerce)

```mermaid
flowchart TB
    LP[Whop Landing/Offer Page] --> OF[Offer: Plan + Price]
    OF -->|checkout| PAY[Stripe via Whop]
    PAY --> LIC[License/Membership Created]
    LIC --> GATE{Delivery Channel}
    GATE --> DIS[Discord Role Sync]
    GATE --> APP[App/SaaS Login Access]
    GATE --> FILE[File/Course Unlock]
    LIC --> DASH[Whop Dashboard: churn, MRR, renewals]
    DASH --> CRM[Export to CRM / Zapier]
```

- No inventory. "Product" = a plan definition, not a SKU.
- Delivery = access grant, not shipment.
- Ideal for SaaS resellers, communities, digital licenses.

---

## 6.4 Design System — Supabase + Stripe + Next.js (DIY Headless Engine)

```mermaid
flowchart TB
    subgraph FE["Next.js Storefront"]
        LP[Landing Page]
        PDP[Product Page]
        CART[Cart / Checkout UI]
    end

    subgraph BE["Supabase Backend"]
        AUTH[Auth - users/sessions]
        PG[(Postgres - products, orders, inventory)]
        RLS[Row Level Security]
        EDGE[Edge Functions - business logic]
        STOR[Storage - product images]
    end

    LP --> PDP
    PDP -->|query via supabase-js| PG
    CART --> AUTH
    CART -->|call edge function: create-checkout| EDGE
    EDGE -->|create session| STRIPE[Stripe Checkout]
    STRIPE -->|webhook: payment success| EDGE
    EDGE -->|insert order, decrement stock| PG
    PG --> RLS
    PDP --> STOR
    PG --> CRM[Sync to CRM via Edge Function]
```

- You build the "engine" yourself: catalog, cart, inventory logic = your Postgres schema + Edge Functions.
- Full control, zero platform fees, but you own bugs, security (RLS), and scaling.
- Best when: custom business logic Shopify/Whop can't express (marketplaces, custom pricing rules, complex access tiers).

---

## 7. Decision Table — Which Stack?

| Need | Use |
|---|---|
| Ship in 1 week, non-technical team | Ecom Builder (Shopify/Wix) |
| WordPress content site + shop | WooCommerce |
| Full custom UX, multi-frontend (web+app) | Headless + Ecommerce API |
| Digital products, memberships, no shipping | Whop |
| Just need to charge money, build everything else | Stripe only |
