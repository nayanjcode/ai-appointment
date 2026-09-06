# Epics — the unit of work

> Output of stage 0 (`/slice`), 2026-09-07. **This supersedes F01–F10 as the
> unit of work.** The F-files remain the problem-domain reference; the epics
> below are what gets a PRD.

F01–F10 was a taxonomy of the *problem*, sliced by noun (booking, queue, voice,
billing). Nouns overlap, because they all read and write the same schedule —
slot protection was specified twice, F05 was F02's only data source, F03 said
outright it was not a system of its own, and F09 had dissolved into two fields.

These epics are sliced by **verb**: *configure* → *commit* → *run* → *know* →
*reach out*. Each is a distinct thing a person does, which is why they don't
overlap.

| ID | Epic | Absorbs | Depends on |
| --- | --- | --- | --- |
| [E1](#e1--configure-the-shop) | Configure the shop | F04, staff/role model from F10 | none |
| [E2](#e2--commit-an-appointment) | Commit an appointment | F01, auth from F10, charge creation from F09 | E1 |
| [E3](#e3--run-the-day) | Run the day | F02, F05 | E2 |
| [E4](#e4--know-the-customer-and-the-business) | Know the customer and the business | F06, F07, charge settlement from F09 | E2, E3 |
| [E5](#e5--reach-out) | Reach out | F08 | E1, E4 |

**Voice is not an epic.** It is a capability inside E2 and E3 — the system works
without voice; voice cannot work without the system. See
[Voice as a capability](#voice-as-a-capability).

---

## E1 — Configure the shop

**When** I'm an owner setting up, or changing prices and hours mid-week, **I want**
to define what my salon offers and who can do it **so** customers stop phoning to
ask, and the rest of the system has a real capacity model to schedule against.

**Ships when:** An owner defines services, staff, hours and modes; a stranger
with no login sees services, prices, hours and open/closed on a public page.

**Acceptance criteria:**

- [ ] Services with duration and price; duration is per specialist (D6 seed)
- [ ] Specialists defined, each mapped to the services they can perform
- [ ] Working hours, off days, holidays — editable live, including closing to new
      orders mid-day (D7)
- [ ] Booking modes enabled per salon, each with its own price and cancellation
      policy (D2)
- [ ] Public page, no login, shows services, prices, hours, open/closed (D23)
- [ ] A price change is visible publicly immediately
- [ ] A new mode type added by a developer becomes available as configuration
      with no schema change (D9)

**Depends on:** none.

**Risk / learning:** Proves D2 is architecture, not a settings page. The one
thing D21 says you may **not** mock. If the extension-point model is wrong here,
every later epic inherits it and it is a rewrite.

---

## E2 — Commit an appointment

**When** I want a haircut and would otherwise phone the salon, **I want** to see
real availability and book it myself **so** I get a confirmed appointment without
anyone picking up a phone.

**Ships when:** A logged-in customer books a service, it reaches Booked (auto or
artist-confirmed), and they can see, cancel and reschedule it. The artist sees
incoming requests and confirms.

**Acceptance criteria:**

- [ ] Login required to book; public view stays open (D22/D23)
- [ ] Availability shows only genuine openings against hours and capacity
- [ ] Booking mode (minimum: next-available + slot) and specialist mode
      (fastest turn / named)
- [ ] D19 auto-accept / auto-reject applied; residual cases route to artist
      confirmation
- [ ] Artist confirms or rejects within the salon's window; still pending at end
      of office hours → rejected, customer told (D10)
- [ ] A slot booking reserves capacity; next-available placement flows around it
      (D13) — this is the *placement* rule, distinct from E3's admission rule
- [ ] Cancel applies the per-mode policy; a slot cancellation writes
      `charge_owed` on the booking (D24)
- [ ] Reschedule
- [ ] My appointments: current, upcoming, history
- [ ] "Wanted to book" capture when the salon cannot serve the request (D1)

**Depends on:** E1.

**Risk / learning:** The state machine plus slot reservation is the structural
core — get it wrong and E3 has nothing sound to execute. First real test of
D19's thresholds: do they cover the boring majority, or does the artist still
get interrupted mid-service?

**Note on size:** this epic is heavy for one PRD. That is a document-size
problem, not a boundary problem — stages 4 and 5 (`/slice-the-spec`,
`/story-splitting`) exist for it. Do not re-cut the epic to make the PRD shorter.

---

## E3 — Run the day

**When** I'm mid-service with wet hands, or I'm a customer sitting in the shop
with no idea when my turn comes, **I want** the day's actual progress to be
visible and self-updating **so** nobody has to ask, and nobody has to tell.

**Ships when:** The artist marks their day started and drives services with
start/pause/complete; every waiting customer sees their position and an ETA that
moves with reality, and gets told before their turn.

**Acceptance criteria:**

- [ ] Artist marks shop open / self available; off-duty mode (D4)
- [ ] start / pause / resume / complete, plus break start and return
- [ ] **A service cannot start while the previous one is still open** (D25.1)
- [ ] Queue position, live ETA, and a "who is being served now" view
- [ ] ETA recomputes on every D5 trigger
- [ ] Only specialists qualified for *my* service count toward my ETA
- [ ] A next-available service that would overrun a reserved slot is not started;
      next-available shifts later and the affected customer is warned as the slot
      nears (D16) — the *admission* rule
- [ ] Elapsed-end nudge: assistant asks for status, repeats 3× over 5 minutes,
      then reports to the admin. **No auto-complete** (D25.2)
- [ ] Specialist can request a timestamp correction from the admin (D25)
- [ ] Notification fires ahead of turn over SMS/WhatsApp (mocked, D21), with
      suppression so reshuffles do not spam
- [ ] Walk-in entered as a booking (D1)
- [ ] Durations adjust from that specialist's own history (D6)
- [ ] Specialist sees their own service count for the day (D25.3 — the incentive
      cannot act without visibility)

**Depends on:** E2.

**Risk / learning:** Where the product is won or lost. Booking is commodity;
a trustworthy ETA is the differentiator. Everything here rests on artists
recording services — mitigated by D25, not eliminated.

**Sequence within the epic:** bare controls plus a naive sum-the-queue ETA
first; learned durations and notification suppression after. Observe the
recording behaviour in week one, not week four after building estimation maths
on an input that may not arrive.

---

## E4 — Know the customer and the business

**When** I'm about to serve someone I half-remember, or I'm deciding whether to
staff Saturday, **I want** the system to tell me what it already knows **so** I'm
not guessing from memory.

**Ships when:** An artist opens a customer and sees history, preferences and
what's owed; an owner opens a dashboard and sees peaks, utilisation, revenue and
lost demand.

**Acceptance criteria:**

- [ ] Customer card: visit history, services, spend, last visit (when, what, with
      whom, at what price)
- [ ] Preference and upsell hints from co-occurrence
- [ ] Owed balance shown; settle action marks it paid offline (D20/D24)
- [ ] Lapsed-regular detection (D1)
- [ ] Owner dashboard: peak hours and days, time per service per specialist,
      revenue
- [ ] Lost demand: cancellations including walkouts, "wanted to book" captures,
      auto-rejections
- [ ] Thin data reports "not enough data" rather than a fabricated trend
- [ ] Work log presented as a **pay record**, not analytics (D25)
- [ ] Per-specialist service counts visible to owner and specialist. The system
      does **not** equalise distribution (D26)
- [ ] Break data is owner-visible and specialists are told it is recorded

**Depends on:** E2, E3.

**Risk / learning:** Low technical risk, real product risk. One salon generates
thin data; this tests whether the "works as a business analyst" claim survives
that or quietly becomes a chart of noise.

---

## E5 — Reach out

**When** I'm looking at a quiet Tuesday, **I want** to push an offer at the right
customers **so** I fill the gap instead of watching it.

**Ships when:** An owner creates an offer, targets a segment, broadcasts it, and
a customer redeems it against a booking at the discounted price.

**Acceptance criteria:**

- [ ] Create offer/coupon with validity window and discount
- [ ] Target a segment drawn from E4 (all / lapsed / new / by service)
- [ ] Broadcast over SMS/WhatsApp (mocked, D21)
- [ ] Redemption applies to a booking and the price reflects it
- [ ] Defined precedence against mode pricing and cancellation charges
- [ ] Opt-out honoured and recorded

**Depends on:** E1, E4.

**Risk / learning:** Lowest risk, loosest coupling. The epic to cut if breadth
pressure hits — furthest from "no phone calls, visible queue".

---

## Voice as a capability

Confirmed 2026-09-07: **the system works without voice; voice cannot work
without the system.** Voice is therefore not an epic.

- Voice **intents** are specified inside each epic's PRD (E2 and E3 carry them;
  E1 is read-only to voice).
- The voice **protocol** is a cross-cutting design doc, not a PRD — the same
  status as D2's configurability. It covers: per-artist wake word (D15),
  confirm-before-write (D12), clarify-and-reject (D15), availability gating
  (D12), fallback to the classic UI, and proactive surfacing of controls (D14).

---

## What dissolved

| Was | Now |
| --- | --- |
| **F01 + F02** | Split by verb instead: **commit** (E2) and **execute** (E3). Slot protection appears in both, but as two rules that were previously conflated — *placement* at booking time, *admission* at start time. |
| **F03 voice** | Capability inside E2/E3, plus one cross-cutting protocol doc. |
| **F05** | Into E3 — it was F02's only data source, so they could never ship apart. |
| **F06 + F07** | Merged into E4. Same event substrate, two audiences. |
| **F09 billing** | Two fields on a booking (D24). Created in E2, settled in E4. |
| **F10 identity** | Staff and role model → E1. Login/session → E2. Booking-on-behalf → E2. Role-scoped permissions cross-cut. Open items **e**, **g** move to E1/E2 stage 1. |

## Sequencing rationale

The order is largely forced: E1 defines the capacity model E2 schedules against,
and E2 produces the appointments E3 executes. What is *not* forced is depth —
E1 and E2 should ship at the minimum that lets the next one start, because the
learning lives in E3. E1 goes first despite being unglamorous, because D2
configurability cannot be retrofitted and D21 forbids mocking it. E4 follows E3
by necessity (no events, no analytics) and is cheap once the event stream
exists. E5 is genuinely optional.

**If work stopped after E3**, the salon takes bookings without a phone and shows
an honest live queue — the whole of the stated "done", with only the
intelligence layer missing.
