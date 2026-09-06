# F08 — Marketing & broadcast

**From:** problem 5 (scheme/coupon advertise)

Thinnest section in the source — one line. Expanded only by the owner narrative
("advertisements and offers can be managed and broadcasted via this").

## Capabilities

- Create and manage offers, schemes, coupons
- Broadcast to customers
- Apply a coupon to a booking or bill
- Target by segment, using F06 data

## Dependencies

F06 (audience), F09 (redemption), F04 (pricing interaction).

## Open questions

- **Channel is unspecified.** In-app only, or SMS/WhatsApp/email? This is the
  whole feature — a broadcast nobody receives is not a broadcast.
- Consent and opt-out are not mentioned. Unsolicited commercial messaging is
  regulated in most jurisdictions.
- Do coupons interact with booking-mode pricing (F01/F04)? A discount on a VIP
  slot with a mandatory cancellation charge needs a defined precedence rule.
- Is this iteration-1 scope at all? It is the least connected to the stated core
  pain (phone calls and waiting).
