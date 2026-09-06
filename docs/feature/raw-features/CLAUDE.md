# Raw feature breakdown

> Auto-loaded when Claude reads anything in this directory.

Derived from `docs/Initial raw problem List.md` (9 problem sections) plus the
owner's feature narrative. **Input was problems, not features** — this document
is the first pass at converting one into the other.

Status: **raw**. Not grilled, not specified. Feeds stage 0 (`/slice`) of the
pipeline in `CLAUDE.md`.

## Features

| ID | Feature | From problems |
| --- | --- | --- |
| [F01](F01-booking-scheduling-core.md) | Booking & scheduling core | 1, 3, 9 |
| [F02](F02-live-queue-eta.md) | Live queue & ETA | 2 |
| [F03](F03-voice-ai-assistant.md) | Voice-first AI assistant | 1 |
| [F04](F04-shop-configuration.md) | Shop configuration & catalog | 1, 3, 5 |
| [F05](F05-specialist-workspace.md) | Specialist workspace | 1, 2, 4 |
| [F06](F06-customer-crm.md) | Customer profile & CRM | 6, 7 |
| [F07](F07-business-analytics.md) | Business analytics | 4, 6 |
| [F08](F08-marketing-broadcast.md) | Marketing & broadcast | 5 |
| [F09](F09-billing-payments.md) | Billing & payments | 1 |
| [F10](F10-identity-roles.md) | Identity, roles & permissions | 1 |

See [SCOPE-AND-CONFLICTS.md](SCOPE-AND-CONFLICTS.md) for what was ruled out,
what contradicts, and what is unresolved.

## How 9 problems became 10 features

The mapping is not 1:1. Three collapses and one split:

- **Problems 1, 3, 9 → F01.** All three are booking: #1 is the *channel*
  (phone), #3 is *availability lookup*, #9 is a *booking mode* (VIP). One
  feature with mode variants, not three features.
- **Problems 6, 7 → F06 + F07.** "Business analyst" (#6) and "software that
  knows the customer" (#7) share a data substrate but split by audience:
  per-customer intelligence (F06) vs. per-business intelligence (F07).
- **Problem 1 → F03, F09, F10.** #1 is the largest section and is really three
  things: replace the phone channel (F03), absorb the receptionist's payment
  duty (F09), and absorb their delegation duty (F10 roles).
- **Problem 8 has no feature.** See SCOPE-AND-CONFLICTS.md — it implies a
  different product.

## Reading note

Every "Open question" below is a real gap in the source, not a placeholder.
They are the input to stage 1 (`/grill-me`), and none should reach a PRD
unresolved.
