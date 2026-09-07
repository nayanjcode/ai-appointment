# Decisions

Answers from the owner, 2026-09-06. These resolve open questions raised in the
feature files. **Not yet grilled** — recorded as stated, not stress-tested.

## D1 — Problem 8 is demand capture, not a marketplace

**Supersedes the marketplace concern in SCOPE-AND-CONFLICTS.md.** Problem 8 was
read here as customer-side cross-salon discovery. The intent was the *salon's*
view of the same event: knowing it lost someone.

Three mechanisms:

1. **Lapsed-customer detection** (F07) — who was a regular, stopped coming, and
   why.
2. **Walk-ins are bookings.** The owner always creates an appointment for a
   walk-in. A walkout is therefore recorded as a **cancellation**, which makes
   it measurable.
3. **"Wanted to book" button** (F01) — capture demand that could not be served.
   Shown **conditionally only**, e.g. queue full with limited hours remaining.
   Trigger condition is deliberately narrow and still to be defined.

**Consequence:** this substantially repairs the F07 gap flagged earlier. Walkouts
and turned-away demand become observable. Missed *phone* calls remain invisible
(call agents are out of scope), but they are no longer the only loss signal.

**Scope unchanged:** single-salon B2B. No cross-salon search, no public
marketplace.

## D2 — Configurability is the architecture, not a settings page

Every salon operates differently. The system must let each salon configure:

- **Which booking modes exist** — one salon wants next-available only; another
  wants next-available + slot; another adds VIP
- **Price per mode** — varies per salon
- **New modes of any kind** — specialist-selection modes, cancellation rules,
  and others not yet enumerated
- Services, durations, hours

Stated requirement: introducing a new mode should be **easily doable in the
system**.

**Consequence:** this is the rules-engine end of the spectrum, not fixed-schema.
It is the single largest architectural constraint in the product and must be
designed in from day one — retrofitting configurability is a rewrite.
See open question O1 below: it is not yet settled whether a salon *owner*
authors a new mode through the UI, or whether "easily" means a low-effort dev
change.

## D3 — Orders require artist confirmation

A new booking is **not accepted until the artist confirms it**. Salons have a
fixed time window for this.

**Consequence:** booking gains a `pending` state. This is new and touches F01,
F03, and F05.

## D4 — Artist shop-open control

The artist gets a dedicated **start / stop** control indicating the shop has
opened, or that they are available in the shop today.

## D5 — Live estimation is explicitly mutable

Resolves the "VIP vs. ETA integrity" contradiction: ETAs are **not promises**.
The schedule reshuffles and estimates update. Reshuffle triggers include:

- A service starts
- A service ends
- A specialist goes on break, and returns
- A VIP booking is inserted

"Live estimation time" is defined as this continuous reshuffle.

## D6 — Service duration: seeded per specialist, then learned

Duration is entered **per hair specialist** at setup, then the system adjusts it
using that specialist's own history. Resolves the F02 cold-start question:
manual seed first, AI refinement after.

## D7 — No booking outside operating hours

Customers cannot book on off days or off hours. The **owner can change shop
timings** at any time — to open capacity, or to stop taking orders in an
emergency.

## D8 — Notifications go over SMS or WhatsApp

Resolves the F02 / F08 channel question. Not app-push-dependent, which matters
given the web-first, mobile-first scope.

## D9 — New modes are authored by developers (resolves O1)

Salons do not build their own modes. Configurability (D2) is delivered through
**clean extension points plus per-salon configuration values**, not a
rule-definition UI.

**Consequence:** materially smaller than the alternative. No rule builder, no
validation UI, and the voice assistant only ever meets a known, finite mode
vocabulary. Adding a genuinely novel mode is a small dev change plus config.

## D10 — Booking state machine (resolves O2)

```
Pending ──▶ Booked ──▶ In Progress ──▶ Completed
   │           │
   └──▶ Cancelled ◀┘
```

- **Pending** is the state after a customer books, before artist confirmation.
- **Auto-accept criteria** and **auto-reject criteria** both exist — the specific
  rules are still to be defined.
- **Timeout is bounded by office hours.** A pending booking that reaches the end
  of the working day is **rejected**.

## D11 — Payment by mode (partially resolves O3)

| Mode | Amount blocked at booking | Cancellation charge |
| --- | --- | --- |
| Slot booking | **50%** | Yes |
| Next available | None | **None** |
| VIP | *undecided* | Yes (stated earlier) |

Blocking 50% on slot bookings makes the slot cancellation charge enforceable —
this was the gap flagged in O3. Next-available stays frictionless, which is
right for the default path.

**Still open:** VIP is not covered. It was stated earlier to always carry a
cancellation charge, which is unenforceable without a block or card on file.

## D12 — Voice cross-talk mitigations (partial — see VOICE-CROSSTALK.md)

Accepted controls:

- Voice command confirmation before any state change
- Commands only processed for a specialist the system shows as available — not
  off for the day, not on break
- Per-artist authentication; the artist signs in as themselves and starts the day

**These are authorization controls and do not solve capture.** Two authenticated,
on-duty artists in one room is the collision case, and every check above passes.
See [VOICE-CROSSTALK.md](VOICE-CROSSTALK.md) for options that address capture and
for the recommendation to consider scoping voice to customers in v1.

## D13 — Slot bookings are protected capacity

A 5–6 PM slot booking must not be pushed back by delays accumulating in the
next-available queue. Slot bookings are **reserved capacity**; next-available
work flows around them.

**Consequence:** this is a real scheduling constraint, not a preference. The
queue engine must treat slots as fixed points and fit next-available work into
the gaps — including refusing to start a next-available service that would
overrun into a reserved slot. Affects F01 and F02 structurally.

## D14 — Button semantics are UI-only

The start/stop and confirm controls are the **standard UI** surface. In the
chatbot everything is voice, and the assistant must **proactively surface these
capabilities** so users know they exist and can use them.

**Note:** this changes the interface, not the underlying dependency — the system
still needs the events. A proactive assistant can help by nudging ("you have not
marked your last service complete"), which is a genuine mitigation for the
forgot-to-press risk.

## D15 — Cross-talk: wake word + phone audio + clarify-and-reject

Decided: **no external hardware.** Voice runs on the phone's own audio, for both
customers and staff.

Accepted protocol:

- **Distinct per-artist wake word.** Ambient speech is ignored unless prefixed.
- When A's phone does capture B's command, **A's phone asks what to complete**;
  A rejects it, B repeats.
- Confirmation before any state change (D12).
- Availability gating and per-artist auth (D12).

**Customers cannot start or stop a service.** After booking they can only cancel,
or read. This bounds the damage: cross-talk can only ever affect staff writes,
and staff writes are four controls.

**Assessment.** The wake word is doing most of the work — it moves the system
from "always listening to the room" to "listening only when addressed," so B
saying "mark complete" without A's wake word never reaches A at all. That is a
genuine capture-level control, not just authorization.

Two residual risks, both worth watching rather than blocking on:

1. **Phone position.** A phone mic is only near-field if the phone is near the
   mouth. On a counter 2 m away with wet hands, it is a room mic. A cheap
   Bluetooth earbud paired to the phone they already own restores near-field
   *and* hands-free for ~$15–20 — no new device, no new system. Worth trialling
   before assuming the bare phone is enough.
2. **Alert fatigue.** Every cross-talk event costs two people: a spurious prompt
   for A, a repeat for B. If it happens often, A starts confirming reflexively —
   which is the original failure. Wake words should keep frequency low; measure
   it rather than assume it.

## D16 — Slot protection: shift later and warn (resolves the D13 admission rule)

When next-available work threatens a reserved slot:

- **Next-available bookings shift later.** The slot holds. Delay is absorbed by
  the flexible queue, which is what makes it flexible.
- **Warn the affected customer** on their schedule card as the slot approaches.

## D17 — VIP payment is per-salon configurable

VIP may carry no payment, or an extra charge, at the salon's discretion.
Consistent with D2 and D9: developers build the option, salons configure it.

## D18 — No payments in v1

Payment handling moves to a later version.

**This contradicts D11 and must be reconciled — see C1 below.**

## D19 — Auto-accept / auto-reject criteria (APPROVED 2026-09-06)

Delegated with "define smartly what you think is relevant", then **approved as
written**. Every threshold is per-salon configurable (D2).

**Auto-accept** when all hold:

- Artist has started their day (D4) and is not on break
- Requested time falls inside the artist's configured working hours
- The artist is qualified for the requested service
- No conflict with an existing booking or reserved slot (D13/D16)
- Booking is more than *N* minutes ahead — default 60

**Auto-reject** when any holds:

- Artist is off, on leave, or has not started the day
- The artist does not offer that service
- Hard conflict with an existing booking or reserved slot
- Still pending at end of office hours (D10, already decided)

**Require manual artist confirmation** — the residual cases:

- Booking is imminent (inside the *N*-minute window); the artist should actually
  see it before it lands
- VIP request
- New customer combined with a long or high-value service
- Requested duration materially exceeds that artist's learned average (D6)

Rationale: auto-accept should cover the boring majority so the artist is not
interrupted mid-service, which is the whole point of D3. Manual confirmation is
reserved for cases where a human would plausibly say no.

## D20 — C1 resolved: record charges as owed, settle offline (option 2)

Cancellation charges are **recorded as owed** and settled offline — typically
cash at the next visit. No payment integration in v1 (D18). Needs a small
ledger; keeps the policy real without taking money.

Applies to slot bookings (D11) and to VIP where a salon configures a charge
(D17). The 50% *block* does not happen in v1 — there is nothing to block with.

*Note: option 3 (no-show tracking) was recommended; the owner chose option 2.*

## D21 — This is a demo build

Iteration 1 is for demonstration. **Maximise feature breadth**; mock external
integrations that cannot be built for real.

**Scope is demo-sized; code quality is production-grade.** The real product
starts from this codebase with no or minimal change. Mocks go behind replaceable
interfaces; domain logic is written for keeps.

**Consequence — reframes several "blocking" concerns:** payment processing, real
SMS/WhatsApp delivery, and AI duration learning (D6) are all mockable. Concerns
raised earlier about compliance, deliverability, and provider integration are
**deferred, not solved** — they return in a production iteration.

**Not licensed by this:** configurability (D2/D9) is architectural and must be
real. Retrofitting it is a rewrite, demo or not.

## D22 — Login required to book; viewing is open

- **Booking requires login.** No guest booking.
- **Viewing without login is the public salon view only** — queue length,
  current wait, availability, services and prices. **No personal data is
  reachable without login** (D23).

Resolves O7 and O8.

## D23 — Unauthenticated access is public data only (resolves O8)

Without login, a visitor sees the **public salon view**: queue length, current
wait, availability, services, prices. Nothing personal.

**No personal data without login.** Reading your own appointments requires
authentication, consistent with D22 requiring it to book. No magic links, no
tokenised URLs.

## D24 — Owed charges are booking fields, not a ledger entity

Approved 2026-09-07.

A cancellation that incurs a charge (D11 slot, D17 VIP) records it **on the
booking itself**: `charge_owed` and `settled_at`. A settle action on the
customer card marks it paid offline (D20). No separate ledger entity in v1.

**Rationale:** the event that creates a debt owns its lifecycle. Charges are
created in E2 (a cancellation *is* a booking event) and read and settled in E4,
which is otherwise pure read-only projection — a write-owning ledger there would
break that property and make E4 expensive to build last.

## D25 — Forgot-to-press: three mechanisms (resolves open item j)

Approved 2026-09-07.

1. **Serialization.** A service cannot start while the previous one is still
   open. The next customer's arrival forces the close. Structural, not a
   reminder.
2. **Elapsed-time nudge.** When a service's expected end passes with no status
   change, the voice assistant asks for the status. It repeats **3 times over
   5 minutes**, then **reports to the admin**. There is **no auto-complete** —
   auto-completing would fabricate a pay record.
3. **Per-service income.** Specialist earnings depend on services recorded, so
   the specialist has their own reason to keep the log accurate. **Specialists
   are responsible for keeping their work logs clear.**

**Timestamp correction.** Serialization guarantees *state*, not *timestamps* — a
batch close at end of day yields correct states and wrong durations, which would
poison D6's learned durations. Correction path: **the specialist requests a time
change from the admin**, quoting the approximate real time. Not self-service.

**Consequences:**

- The work log is a **pay record**, not analytics. Disputes are real, and the
  admin correction path is the resolution mechanism.
- The escalation target makes open item **e** (owner vs. admin — four roles or
  five?) load-bearing: something now routes to "admin".
- Alert fatigue is an **accepted risk**. The nudge fires on elapse with no grace
  period. Measure frequency during the demo rather than pre-tuning it.

## D26 — Unequal work distribution is accepted

Approved 2026-09-07.

Because pay is per service (D25), "fastest turn" assignment is also an income
allocation. Distribution will be unequal. **This is accepted** — it already
happens in salons and is the reality of the segment. No fairness rule goes into
the assignment algorithm.

Per-specialist counts are visible (E3/E4) so specialists can check their own
numbers, but the system does not equalise.

## D27 — The role set is per-salon configuration (resolves e, g, n)

Approved 2026-09-07. Stage-1 grill of E1.

There is **no fixed role enum**. A salon has anywhere from **1 to 5 roles** and
composes its own: a solo owner doing everything; owner + staff; owner +
receptionist + stylist; owner + manager + receptionist + stylist; and so on.
**One person may hold several roles** — the owner who also cuts hair is a
first-class case, not an edge case.

**Consequence — permissions attach to capabilities, not role names.** Nothing in
the code may test for "admin", because no salon is guaranteed to have one. E1
defines a fixed set of capabilities (edit catalog, confirm bookings, settle
charges, receive escalations, perform services, view business analytics, …) and
a salon composes named roles out of them. This is D2's configurability applied
to people rather than modes.

**Resolves open item n.** D25's "reports to the admin" becomes *reports to
whoever holds the `receives_escalations` capability*, **falling back to the
owner** when nobody does. The owner capability always exists — it is the one
thing a salon cannot configure away.

## D28 — E1 configuration answers

Approved 2026-09-07. Stage-1 grill of E1.

- **VIP pricing: build all three.** Surcharge, multiplier, and replacement price
  are all implemented; the owner picks one per salon. Consistent with D9 —
  developers build the options, salons configure.
- **Three extensible axes** (resolves gap 4): booking mode, specialist-selection
  mode, and cancellation rule are all extension points under D9.
- **Multi-branch is one tenant.** No separate branch entity in v1.
- **Service mapping is a simple boolean** — a specialist either performs a
  service or does not. No skill level, no rating.
- **Durations are visible to everyone.** A default may be supplied, but every
  salon must set its own; for genuinely new service entries a default is not
  meaningful.

**Knock-on — open item b ("best artist") has no data.** A boolean mapping cannot
rank anyone. That specialist-selection mode must either drop from v1, or be
redefined as an explicit owner-authored ranking rather than an inferred one.
**Carried to E2's stage 1.**

## D29 — Mid-day closure does not auto-cancel

Delegated then **approved 2026-09-07**.

When an owner closes hours mid-day (D7):

- New bookings in the closed window stop **immediately**.
- Bookings **already committed** inside that window are **not auto-cancelled**.
  They are flagged *at risk* and the owner is shown the short list with one
  decision per booking: **notify and cancel**, or **keep**.
- Cancellations made this way are **salon-initiated**: no `charge_owed` against
  the customer (D24 charges are for customer-initiated cancellations), and they
  are tagged distinctly so E4's lost-demand reporting does not record them as
  customer churn.

**Rationale.** Silent auto-cancel destroys trust with the exact customers who
already committed. Silently keeping them means the barber arrives at an
emergency closure to find customers turning up. Forcing one explicit choice over
a short list is the honest middle, and it is cheap to build. The distinct tag is
what stops an emergency from polluting the churn numbers.

## D30 — The committed price is honoured

Delegated then **approved 2026-09-07**.

Price is **captured onto the booking at commit time** and honoured at service
time. Catalog and public-page price changes apply to **new bookings only**.

**Rationale.** Charging a different price than the one quoted is the fastest way
to lose a customer, and F04's "live-accurate" requirement is about the *public
page*, not about repricing existing commitments. It is also cheap — one
denormalised field — and E4 needs the as-charged price anyway for accurate
historical spend.

## D31 — The public page ships in E1; E3 fills the queue slot

Delegated then **approved 2026-09-07**.

E1 ships the public page with what E1 owns: services, prices, hours, open/closed.
Queue length and current wait (D23) render as a **section that appears when E3
lands**, and are absent until then.

**Rationale.** The page is E1's surface; the live queue is a widget on it. This
keeps the epic boundary clean without changing what finally ships — and given
the whole build is a week, this is build order rather than a real boundary
question.

## D32 — Capability vocabulary is fixed; salons compose roles from it

Approved 2026-09-07. Stage-1 grill of E1, round 2.

Ten capabilities, authored by developers (the D9 principle applied to people):

`manage_catalog`, `manage_staff`, `manage_hours`, `manage_modes`,
`confirm_bookings`, `perform_services`, `book_on_behalf`, `settle_charges`,
`view_analytics`, `receive_escalations`.

A salon creates named roles ("Receptionist") as bundles of these. Salons may
**not** invent capabilities.

- **The owner role always exists, holds every capability, and cannot be deleted
  or stripped.** It is D27's escalation fallback and the guarantee that a salon
  cannot lock itself out.
- `perform_services` is the structurally significant one: holding it is what
  puts a person in the queue and gives them service mappings. The owner who also
  cuts hair is simply an owner who also holds it (D27).
- **No code may test for a role name.** Every check is against a capability.

## D33 — Staff are created by the owner and invited

Approved 2026-09-07.

The owner creates the staff record (name + phone); the system sends an invite
over SMS/WhatsApp (mocked, D21); the staff member sets a password on first open.

No self-signup — a salon cannot have strangers claiming to be its stylists. No
login-less staff records either: D25 makes the specialist personally accountable
for their own work log, which requires them to have an account.

## D34 — Hours model, and what "open" means

Approved 2026-09-07.

- **Salon hours are the outer bound; per-specialist availability sits inside
  them** and is validated against them. A specialist cannot be available while
  the shop is shut.
- **Booking availability uses *configured* hours** (D7; D19's auto-accept
  already tests "inside the artist's configured working hours").
- **The live queue uses *actual* starts** (D4).
- **The public page shows both, and does not collapse them to one boolean** —
  "Open · 2 stylists in" versus "Open · nobody in yet". Collapsing is how the
  customer who arrives at 9:05 finds an empty shop.

## D35 — Modes are salon-level and reference a cancellation rule

Approved 2026-09-07.

- **Booking modes are enabled per salon, not per service.** No stated need for
  per-service modes, and it multiplies the configuration surface for no gain.
- **A mode holds a reference to one cancellation-rule type plus that rule's
  parameters** — slot → `percentage_charge` at 50%; next-available →
  `no_charge`. The three extensible axes (D28) therefore stay genuinely
  independent, and adding a cancellation rule requires touching no mode.

## D36 — E1 ships a seed catalog

Approved 2026-09-07.

A default catalog (haircut, beard trim, shave, colour, facial, waxing, …) with
typical durations and **blank prices**. The owner edits rather than authors.

Prices are deliberately blank: a wrong default price could go live on the public
page, whereas a wrong default duration only nudges an ETA that D6 corrects from
history. Turns setup from twenty minutes into two.

## D37 — Per-specialist price override, and the price-confirmation window

Approved 2026-09-07. Owner asked for the override; the window mechanism is
theirs, refined for two collisions found during the grill.

**Price:** base price per service, with an **optional per-specialist override**.

**The collision:** with *fastest turn* the system picks the specialist, so the
system picks the price — and under D30 the price is captured at commit, but for
next-available the specialist may be unknown until service time.

**Resolution, in three parts:**

1. **Named specialist** — price known upfront, no window.
2. **Any/fastest turn on a mode with no cancellation charge** — assign, disclose
   the price, then a **10-minute confirmation window capped at the time until
   their turn**. Silence confirms. This is safe *because* next-available
   cancellation is free (D11): an auto-confirmed customer who dislikes the price
   simply cancels.
3. **Any/fastest turn on a charge-bearing mode** (slot, and VIP where a salon
   configures a charge) — **quote the salon base price and honour it**, whoever
   performs the service. The salon absorbs the premium. Timeout-confirms is
   never applied where cancelling costs money.

**No new state in D10.** The booking is created in Pending immediately but held
out of the artist's confirmation queue for the window via an `artist_visible_at`
timestamp. Cancel inside the window and the artist never saw it; confirm or
lapse and it enters the normal D19 flow. This preserves D3's actual intent — do
not interrupt someone mid-service for a booking that may evaporate — for the
cost of one field.

**Knock-on for open item b.** Per-specialist pricing reintroduces the seniority
dimension D28 removed: a salon charging more for one stylist is signalling
exactly that. "Best artist" could be defined as the highest-priced qualified
specialist, with no new data. **Carried to E2's stage 1** as a candidate
resolution, not a decision.

## D38 — Multi-tenant model, global customer identity

Approved 2026-09-07.

- **Multi-tenant in the model; one salon seeded for the demo.** D2 forces
  per-salon configuration everywhere, so tenant scoping exists whether or not it
  is named — better named than discovered.
- **Customer identity is global; the customer↔salon relationship is per-salon.**
  One login, one profile per salon for history and CRM. F06's history is
  per-salon regardless, and global auth sits alongside D2 as a
  retrofit-is-a-rewrite item.

## D39 — Services carry an optional buffer

Approved 2026-09-07.

One optional buffer per service, set at salon level, added **after** the service
when scheduling.

D6's learned durations cannot capture it — start/complete brackets the service,
not the sweeping and chair reset after it. Without a buffer every ETA runs
slightly optimistic and compounds across the day, and E3's "will this overrun
the reserved slot" check (D16) is dishonest.

## D40 — Off days, holidays and leave are two mechanisms

Approved 2026-09-07.

- **Recurring weekly off days** live in the hours model, at salon and specialist
  level (D34).
- **One-off dated closures are exception entries that override it**, at either
  level. "The shop is shut on Diwali" and "Ravi is off next Tuesday" are the
  same feature.

**Consequence:** D7's live mid-day change is just an exception created for today,
which means D29 already defines what happens to bookings caught inside one.

## D41 — Pay-relevant config changes are attributable

Approved 2026-09-07. Stage-3 gate on PRD 001.

Changes to **service durations** and **per-specialist price overrides** record
the **actor and a timestamp**.

Deliberately narrow: two columns on two kinds of change, **not** a general audit
log — that stays out of scope. The justification is D25, which makes the work
log a pay record; a pay dispute with no record of who changed what is
unresolvable.

*Origin: this was inferred by the PRD author and asserted in §6 without a
decision behind it. The gate forced it to be decided or dropped.*

## D42 — Slot, VIP and prebook assign the specialist at booking time

Approved 2026-09-07. Stage-3 gate on PRD 001.

**Only next-available leaves the specialist unknown at request time.** For slot,
VIP and prebook the specialist is assigned when the booking is made — that is
what D13's "protected capacity" reserves: a named person's time, not an
anonymous chair.

**Two consequences:**

1. **D37's confirmation window applies only to next-available.** Everywhere else
   the price is known before commitment and D30 captures it.
2. **The charge-bearing pricing branch is deleted.** An earlier draft had the
   salon quote base price and absorb any premium on charge-bearing modes. It is
   unnecessary — and it carried a defect: it only considered the assigned
   specialist being *more* expensive, so it silently overcharged whenever they
   were cheaper than base.

## D43 — One at-risk mechanism for config changes that invalidate bookings

Approved 2026-09-07. Stage-3 gate on PRD 001.

Four rules were one rule in disguise. **Any configuration change that
invalidates a committed booking** produces the same behaviour:

Triggers: mid-day or dated closure (D7/D29/D40); salon hours narrowed; service
deactivated; last specialist mapped to a service removed.

Behaviour: new bookings stop immediately; committed bookings are **never
auto-cancelled**; the owner gets an at-risk list and decides **per booking**
(notify-and-cancel, or keep); salon-initiated cancellations raise **no
`charge_owed`** (D24) and are **tagged distinctly** so E4 never counts them as
customer churn (D29). Default is keep; cancel-all is one action.

**Also settled:** narrowing salon hours **clamps** specialist availability to
the new bounds and reports what changed — it does **not** block the save.
Blocking contradicts D7, whose whole point is that the owner can close the shop
at will, including mid-emergency.

*Supersedes the bespoke warnings previously specified separately for service
deactivation and specialist-mapping removal. The gate found one path guarded and
its twin unguarded.*

## D44 — A Pending booking consumes capacity during the confirmation window

Approved 2026-09-07. Stage-3 gate on PRD 001.

During D37's 10-minute price-confirmation window the booking exists in Pending
and **holds capacity**. An availability calculation that ignores Pending
double-books.

Grief risk is accepted: D22 requires login, so there is an identity behind every
hold, and the window self-clears in ten minutes.

This is E2's definition of Pending, recorded here so E1's PRD does not
contradict it.

## D45 — E1 specifies the at-risk mechanism; E2 delivers it

Approved 2026-09-07. Stage-3 gate on PRD 001.

The at-risk mechanism (D43) reads bookings, which are E2's entity — so E1 is not
as independent as EPICS.md claimed.

**Resolution: separate specification from delivery.** The mechanism is specified
in PRD 001 because E1 config changes trigger it, and **delivered with E2**. E1
ships the configuration writes; the at-risk list arrives with the epic that owns
bookings. E1's dependency *for delivery* remains none.

An event bus (E1 emits `config.changed`, E2 subscribes) is the cleaner
architecture and was rejected as the wrong call for a one-week build — it buys
decoupling with no second consumer to justify it.

**Residual risk:** if E2 slips, E1 ships with config changes that can silently
orphan bookings and no list to catch them. Until E2 lands, the triggering
changes should warn generically that committed bookings may be affected.

---

# Contradictions to resolve

## ~~C1~~ — RESOLVED by D20 (option 2)

*Kept for context. Original analysis below.*

### ~~C1 — D18 (no payments in v1) breaks D11 and D17~~

D11 blocks 50% on slot bookings and charges on cancellation. D17 allows a VIP
charge. **D18 removes the ability to do either.**

With no payment integration in v1:

- The 50% slot block cannot happen
- No cancellation charge is collectible
- Cancellation policy becomes informational only

That may be fine, but the consequence should be chosen, not inherited. Options:

1. **Drop charges from v1 entirely.** Simplest. Cancellation is free everywhere;
   revisit with payments in v2.
2. **Record the charge as owed, settle offline.** The salon collects in cash at
   the next visit. No integration, keeps the policy real, needs a small ledger.
3. **Substitute a non-monetary deterrent.** Track no-shows; restrict repeat
   offenders from slot or VIP booking. No money, still enforceable, and it feeds
   F06/F07.

Option 3 is the closest thing to enforcement without taking money, and the data
it produces is useful independently.

---

# Still open

## Blocking

**Nothing.** All structural questions are resolved — `/slice` is unblocked.

The items below are per-feature detail. They belong in per-epic `/grill-me`
passes at stage 1, not in a single up-front sweep.

## Needed before a PRD

| # | Question | Feature |
| --- | --- | --- |
| a | Who grants VIP eligibility — owner allowlist, paid tier, both? | F01 |
| b | "Best artist" — by what measure? D28 removed the ranking data, but **D37 restores a proxy**: per-specialist price signals seniority. Candidate = highest-priced qualified specialist | E2 |
| c | Late arrival: the 10-min-early rule is stated; the penalty is not | F01/F02 |
| d | Notification lead time — "X minutes/hours" has no value; SMS/WhatsApp is metered (D8) | F02 |
| ~~e~~ | ~~Owner vs. admin: four roles or five?~~ **Resolved by D27** — no fixed enum, 1–5 per-salon | E1 |
| ~~f~~ | ~~Guest booking, or account required?~~ **Resolved by D22** — login required, no guest booking | F10 |
| ~~g~~ | ~~One person holding two roles~~ **Resolved by D27** — yes, first-class | E1 |
| h | "Relations of this person with me" — family, referral, or loyalty tier? | F06 |
| i | Consent, retention, and messaging opt-out — live now that D8 picks SMS/WhatsApp | F06/F08 |
| ~~j~~ | ~~Forgot-to-press behaviour for the three controls~~ **Resolved by D25** | E3 |
| k | Exact trigger condition for the "wanted to book" button (D1) | F01 |
| l | Can VIP preempt an **in-progress** service, or only insert ahead in queue? | F01/F02 |
| m | Can a next-available service be refused if it would overrun a reserved slot? (D13) | E3 |
| ~~n~~ | ~~Nudge escalation target~~ **Resolved by D27** — `receives_escalations` capability, falling back to owner | E3 |
| o | Time-change request/approve flow — where it lives, and whether the original timestamp is retained (D25) | E3 |
