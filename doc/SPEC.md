# EXPAND — Product Spec (Master Reference)

> This file is the single source of truth. Reference specific sections when assigning tasks to the AI agent instead of re-pasting the whole spec. Do not duplicate this content into chat prompts — point to it.

## 1. Vision

Expand is a premium, mobile-first Progressive Web App (PWA) marketplace for fishing gear and angling communities. It must feel indistinguishable from a high-end native Android/iOS app: no page reloads, no desktop-style layouts, no HTML-default form styling.

Priority order when tradeoffs arise: **UX > performance > consistency > scalability > accessibility > maintainability > visual flourish.**

## 2. Language Rule

- All user-facing strings: natural, professional **Bahasa Indonesia** (avoid literal translation — use terms common in Indonesian mobile apps: Masuk, Daftar, Beranda, Keranjang, Kategori, Toko, Promo, Pesanan Saya, Tambah ke Keranjang, Bayar, Simpan, Batalkan, Konfirmasi).
- Code, variables, API routes, DB schema, comments, folder names: **English only**.
- Never mix languages within a single UI string.

## 3. Roles

| Role | Capabilities |
|---|---|
| Customer | browse, search, filter, wishlist, cart, checkout, order history, tracking, reviews, notifications, tournament registration, profile |
| Store Owner | dashboard, product/inventory management, orders, customers, sales analytics, promotions, coupons, store customization, notifications |
| Fishing Spot Sponsor | publish sponsored tournaments (banner, description, organizer, location/Google Maps, fee, quota, schedule, rules, sponsors, prizes, gallery, digital certificate) |

Tournament participants get: digital ticket, QR check-in, event reminder, certificate download.

## 4. Native Mobile Experience Requirements

Implement: smooth/shared-element transitions, pull-to-refresh, swipe gestures, native bottom nav, FAB, bottom sheet, modal sheet, native dialog, toast, snackbar, skeleton loading, optimistic UI, ripple effect, blur effect, haptic feedback (where supported), sticky header, safe-area insets, native scrolling, edge-to-edge layout, 60fps animation.

Never: desktop layout, HTML-looking forms, full page refresh, UI flicker.

## 5. PWA Requirements

Installable, service worker, offline support + offline page, cached assets, push notifications, background sync, app shortcuts, adaptive icon, splash screen, standalone mode, auto-updates.

**Install prompt UX:** after onboarding, show a custom floating install card (not the raw browser prompt):
> "Pasang Expand di layar utama untuk pengalaman yang lebih cepat dan nyaman."
Buttons: **Pasang** / **Nanti**. "Pasang" triggers the native install prompt. Never force it repeatedly.

## 6. Payment Strategy (MVP — no payment gateway)

Do **not** integrate Midtrans, Xendit, Stripe, PayPal, Duitku, or iPaymu. Architect the payment layer so a gateway can be added later without changing the checkout flow (i.e., abstract behind a `PaymentProvider` interface — see FLOWS.md).

### Checkout flow
Keranjang → Alamat Pengiriman → Metode Pengiriman → Ringkasan Pesanan → Pembayaran → Konfirmasi Pembayaran → Status Pesanan

### Manual QRIS
- Store owner uploads a static QRIS image via Store Admin Panel.
- Shown at checkout with: Nama Toko, Total Pembayaran, QRIS image (responsive, scannable), Nomor Pesanan, Petunjuk Pembayaran, Copy Order ID button, optional payment countdown.

### WhatsApp Confirmation
CTA: **"Konfirmasi Pembayaran via WhatsApp"** → opens `https://wa.me/62895414700070` with a URL-encoded prefilled message:

```
Halo Admin Expand,

Saya telah melakukan pembayaran.

Nomor Pesanan:
{{ORDER_ID}}

Nama:
{{CUSTOMER_NAME}}

Nama Toko:
{{STORE_NAME}}

Total Pembayaran:
{{TOTAL_PAYMENT}}

Mohon dilakukan verifikasi pembayaran.

Terima kasih.
```

### Admin Verification
Store owner can: view pending payments, verify, reject, mark as paid, view payment history. Customer is auto-notified after verification.

### Order status pipeline
Menunggu Pembayaran → Menunggu Verifikasi → Diproses → Dikirim → Selesai

## 7. UI Design Direction

Premium, clean, elegant, minimal, modern, mobile-first, one-handed-use comfortable. Avoid: AI-generic look, excessive gradients, visual clutter, inconsistent spacing.

Inspiration only (never copy): Material Design 3, Apple HIG, Linear, Shopify Mobile, Tokopedia, Apple Wallet, Google Wallet, Stripe, Cash App.

## 8. Design System Scope

Typography, color tokens, radius, elevation, spacing, grid, motion, icons, buttons, cards, inputs, chips, tags, badges, navigation, bottom sheet, dialog, snackbar, toast, avatar, empty/loading/skeleton/error/success states. All components reusable and consistent — build this once, early (see PHASES.md Phase 1).

## 9. Performance & Accessibility Targets

- Lighthouse: Performance ≥95, Accessibility ≥95, Best Practices =100, SEO ≥90
- Techniques: lazy loading, code splitting, image optimization, minimal JS, fast first paint, zero unwanted layout shift, 60fps
- WCAG AA: large touch targets, proper contrast, semantic HTML, keyboard access, screen reader support

## 10. Tech Stack

Next.js, TypeScript, Tailwind CSS, shadcn/ui, Radix UI, Framer Motion, Supabase, IndexedDB, TanStack Query, React Hook Form, Zod, PWA tooling.

## 11. Non-Negotiables Checklist

- [ ] No page reloads anywhere in the app
- [ ] No payment gateway integrated in MVP
- [ ] All user-facing text in natural Bahasa Indonesia
- [ ] Payment layer abstracted for future gateway swap
- [ ] Design system built before feature screens
- [ ] Lighthouse targets met before final delivery

---
See also: `PHASES.md` (build order), `SCHEMA.md` (database), `FLOWS.md` (detailed user/data flows), `DECISIONS.md` (locked technical decisions).
