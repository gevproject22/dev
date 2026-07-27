# EXPAND — Key Flows

## Checkout Flow
1. **Keranjang** — review items, adjust quantity, remove item, see subtotal.
2. **Alamat Pengiriman** — select/add shipping address.
3. **Metode Pengiriman** — choose shipping method (flat options for MVP, no live courier API needed unless specified).
4. **Ringkasan Pesanan** — line items, shipping, total, edit-back links to earlier steps.
5. **Pembayaran** — show store's QRIS image, Nama Toko, Total Pembayaran, Nomor Pesanan, Petunjuk Pembayaran, Copy Order ID, optional countdown.
6. **Konfirmasi Pembayaran** — CTA button opens WhatsApp deep link (see SPEC.md §6 for exact message template). Order enters `menunggu_verifikasi`.
7. **Status Pesanan** — customer sees live status; store owner sees it in their pending-payments queue.

Each step is a distinct screen/route (no single giant form) to preserve native feel — use shared element transitions between steps, not full navigation reloads.

## Order Status State Machine
```
menunggu_pembayaran
   -> menunggu_verifikasi   (customer taps "Saya sudah bayar" / sends WA confirmation)
   -> diproses              (store owner verifies payment)
   -> dikirim               (store owner marks shipped, optionally with tracking info)
   -> selesai                (customer confirms receipt OR auto after N days)

menunggu_verifikasi -> menunggu_pembayaran (store owner rejects payment, with reason)
```
Notify customer (push/in-app) on every transition.

## Manual QRIS Payment Flow
1. Store owner uploads static QRIS image in Store Admin → stored on `stores.qris_image_url`.
2. At checkout, app renders that image plus dynamic order details (order ID, total, instructions).
3. Customer pays externally (their own banking/e-wallet app scans the QRIS).
4. Customer taps "Konfirmasi Pembayaran via WhatsApp" — prefilled message per SPEC.md §6 opens in WhatsApp.
5. Store owner manually verifies in their dashboard's Pending Payment queue → marks Verified/Rejected.
6. On Verified: order status → `diproses`, customer notified.
7. On Rejected: order status → back to `menunggu_pembayaran`, customer notified with reason, can resubmit.

## Payment Abstraction (for future gateway swap)
```ts
// /lib/payments/provider.ts
interface PaymentProvider {
  createPaymentIntent(order: Order): Promise<PaymentIntent>;
  getStatus(orderId: string): Promise<PaymentStatus>;
  confirmManualPayment?(orderId: string, proof: ManualProof): Promise<void>; // manual providers only
}
```
- `ManualQrisProvider` implements this for MVP.
- Later, `MidtransProvider` / `XenditProvider` can implement the same interface; checkout UI and order state machine remain unchanged.

## Tournament Registration Flow
1. Sponsor creates tournament (banner, description, location + Maps link, fee, quota, schedule, rules, prizes, gallery).
2. Customer browses tournament list → detail page → registers (pays fee via same manual QRIS flow if fee > 0, or instant registration if free).
3. On confirmed registration: generate QR code + digital ticket.
4. Reminder notification scheduled before `schedule_start`.
5. After event: sponsor/store marks attendance via QR check-in scan; certificate becomes downloadable (PDF) post-event.

## Notification Triggers (push + in-app)
- Order status changes (every transition in the state machine above)
- Payment verified/rejected
- Tournament registration confirmed
- Tournament reminder (X hours/days before event)
- New promo/coupon from followed store (optional, later phase)
