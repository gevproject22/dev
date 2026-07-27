# EXPAND — Build Phases

> Rule of thumb: one phase (or even one sub-task within a phase) per chat session with the agent. Start a fresh session per phase to avoid dragging a huge context history into every request. Reference SPEC.md sections instead of re-explaining requirements.

## Phase 0 — Foundation
- Init Next.js + TypeScript + Tailwind project
- Install shadcn/ui, Radix UI, Framer Motion, TanStack Query, React Hook Form, Zod
- Set up Supabase project + client
- Set up folder structure (see DECISIONS.md)
- Set up ESLint/Prettier/tsconfig strict mode
- **Done when:** `npm run dev` runs a blank app with Supabase client connected and no type errors.

## Phase 1 — Design System
- Tokens: color, spacing, radius, elevation, typography, motion (SPEC.md §8)
- Base components: Button, Card, Input, Chip, Tag, Badge, Avatar
- Overlay components: Bottom Sheet, Modal Sheet, Dialog, Toast, Snackbar
- State components: Skeleton, Empty State, Error State, Success State, Loading State
- Build a `/design-system` internal route to visually QA every component
- **Done when:** every component above is rendered and reviewable on `/design-system` with no ad hoc styling needed later.

## Phase 2 — Auth & Navigation Shell
- Supabase Auth (login/register/session)
- Role handling (customer / store owner / sponsor)
- Native bottom navigation + safe-area insets + edge-to-edge layout shell
- **Done when:** a user can register, log in, and land on a role-appropriate home shell with working nav.

## Phase 3 — Customer Core Flow
- Product browse, search, filter
- Product detail page
- Wishlist
- Cart (optimistic UI)
- **Done when:** a customer can find a product and get it into cart with zero page reloads.

## Phase 4 — Checkout & Manual Payment
- Full checkout flow per SPEC.md §6
- Manual QRIS display
- WhatsApp confirmation deep link generation
- Order status pipeline UI
- Payment abstraction layer (`PaymentProvider` interface) even though only manual QRIS is implemented
- **Done when:** a customer can complete a full order end-to-end and see status update after admin verifies.

## Phase 5 — Store Owner Dashboard
- Product/inventory management
- Order management + payment verification screen
- Sales analytics
- Promotions/coupons
- Store customization (incl. QRIS image upload)
- **Done when:** a store owner can manage products and verify a real test payment from Phase 4.

## Phase 6 — Tournament / Sponsor Module
- Tournament creation (sponsor side)
- Tournament listing + detail + registration (customer side)
- Digital ticket + QR check-in
- Certificate generation/download
- **Done when:** a customer can register for a tournament and receive a ticket with QR.

## Phase 7 — PWA Layer
- Manifest, service worker, offline page, caching strategy
- Custom install card (SPEC.md §5)
- Push notifications, background sync
- **Done when:** app installs on Android/iOS home screen and works offline for cached routes.

## Phase 8 — Performance & Accessibility Pass
- Lighthouse audit against SPEC.md §9 targets
- Fix layout shift, bundle size, image optimization
- Accessibility audit (contrast, touch targets, screen reader, keyboard nav)
- **Done when:** all four Lighthouse targets are met on production build.

---
### Credit-efficiency notes per phase
- Ask the agent to plan file structure for the phase first, review it, then implement — cheaper than fixing a wrong large implementation.
- For edits within a phase, use targeted diffs, not full-file regeneration.
- Don't open Phase N+1 in the same session as Phase N unless Phase N is fully closed out.
