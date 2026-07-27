# EXPAND — Locked Technical Decisions

> Purpose: stop the agent from re-litigating architecture every session. If a decision needs to change, update it here first, then proceed.

## Roles & Auth
- Single `users` table with a `role` enum column (`customer` | `store_owner` | `sponsor`). No separate tables per role.
- A single account can only hold one role for MVP (no multi-role accounts yet).
- Auth via Supabase Auth (email/password + optionally magic link). No custom auth server.

## State Management
- Server/async state: TanStack Query only.
- Local/UI state (bottom sheet open, active tab, etc.): React state/context — no Redux, no Zustand, unless a specific cross-cutting UI state problem justifies it later.

## Folder Structure (Next.js App Router)
```
/app
  /(customer)
  /(store)
  /(sponsor)
  /api
/components
  /ui          -> design system primitives
  /shared      -> composed, reusable across roles
/lib
  /supabase
  /payments    -> PaymentProvider abstraction lives here
  /validation  -> zod schemas
/hooks
/types
/docs          -> SPEC.md, PHASES.md, SCHEMA.md, FLOWS.md, DECISIONS.md
```

## Payment Abstraction
- Define `PaymentProvider` interface in `/lib/payments/provider.ts` with methods like `createPaymentIntent`, `getStatus`, `confirmManualPayment`.
- MVP implementation: `ManualQrisProvider` (store-uploaded QRIS + WhatsApp confirmation + manual admin verification).
- Future gateways (Midtrans/Xendit) implement the same interface — checkout UI never talks to a provider directly, only through this interface.

## Database
- Supabase Postgres. See `SCHEMA.md` for tables.
- Row Level Security enabled on all tables from day one (not deferred).

## Styling
- Tailwind CSS + shadcn/ui components as the base layer; Radix for primitives needing accessibility behavior (dialogs, popovers).
- No CSS-in-JS libraries.

## Offline/PWA
- Cache strategy: stale-while-revalidate for product/catalog data, network-first for cart/checkout/order status.
- IndexedDB used for offline cart persistence only, not as a general data cache.

## Language
- All copy in Bahasa Indonesia goes through a central `/lib/i18n/strings.ts` (or similar) rather than being hardcoded inline per component, so tone/wording stays consistent and is easy to audit.
