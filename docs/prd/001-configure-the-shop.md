---
id: 001
title: Configure the shop
status: approved
owner: nayan.jagetiya@allego.com
created: 2026-09-07
updated: 2026-09-07
related: ["docs/feature/raw-features/EPICS.md#e1--configure-the-shop", "docs/feature/raw-features/DECISIONS.md"]
grilled: true
---

# Configure the shop

> Epic **E1**. Stage 2 of the pipeline in `docs/pipeline.md`. Stage 1 (`/grill-me`)
> completed 2026-09-07 over three rounds; its output is **D27–D40** in
> [DECISIONS.md](../feature/raw-features/DECISIONS.md), which this document is
> written against. Where this PRD and a D-decision disagree, the D-decision wins
> and this document is wrong.
>
> **Stage 3 (adversarial gate) passed 2026-09-07.** Seven findings, all applied;
> they produced **D41–D45**. Notable outcomes: the at-risk mechanism (§5.11)
> unified four rules into one, D42 deleted a pricing branch and the defect
> inside it, and E1's independence claim was corrected. `grilled: true`,
> `status: approved` — safe to implement against.

## 1. Summary

E1 lets a salon owner define their salon — services, staff, who can perform
what, hours, booking modes and prices — and publishes that definition as a
public page that anyone can read without logging in. The target user is the
**salon owner** (very often a solo operator who also cuts hair), and the primary
outcome is twofold: it removes the largest category of routine phone call
("what do you charge", "are you open", "do you do beard colour"), and it gives
every downstream epic a real capacity and configuration model to schedule
against.

Nothing in E1 books, queues, or bills. It is the substrate the other four epics
stand on.

## 2. Problem Statement

Salon staff cannot answer the phone. The source problem list names five
conditions where a specialist physically cannot take a call — wet hands during a
shampoo, facial, dye, beard or waxing service; peak hours; off days; night-time
advance enquiries; and back-to-back queued calls. A meaningful share of those
calls are not bookings at all. They are enquiries: opening hours, whether a
service is offered, what it costs, whether the shop is open today.

Two consequences follow:

1. **The salon loses time and customers to questions a web page could answer.**
   The owner's own framing: anything a salon will say on a call, it will
   publish. Today there is nowhere to publish it.
2. **Every salon operates differently.** Modes, services, prices, staffing
   structure and hours all vary salon to salon. A fixed schema cannot represent
   them, and retrofitting configurability later is a rewrite (D2). This is the
   single largest architectural constraint in the product.

There is also a structural dependency: without a configured catalog, staff
roster, capability mapping and hours model, **nothing downstream can function**.
E2 cannot compute availability, E3 cannot estimate a queue, E4 has no prices to
report revenue against.

Evidence is qualitative — the owner's problem list and narrative
(`docs/Initial raw problem List.md`, problems 1, 3 and 5). There is no
production system and therefore no metrics.

## 3. Goals and Non-Goals

### Goals

1. A salon can be defined end to end — catalog, staff, roles, hours, modes,
   prices — with **no developer involvement**.
2. A visitor with **no account** can answer the three most common phone
   questions (what services, at what price, open when) from a public page.
3. Configuration is **per salon**, and the three extensible axes (booking mode,
   specialist-selection mode, cancellation rule) accept a new developer-authored
   type **without a schema change** (D9, D28).
4. A brand-new salon is usable **within minutes**, not an afternoon, via a seed
   catalog it edits rather than authors (D36).
5. Downstream epics receive a capacity model precise enough to schedule against:
   who can perform what, how long it takes them, when they work, and what it
   costs.
6. Permissions work for a salon with **one** person and a salon with **five**
   distinct roles, with no code change between the two (D27, D32).

### Non-Goals

- **A rule-definition UI.** Salons do not author new mode types; developers do,
  and salons configure the result (D9). No rule builder, no validation UI.
- **Separate branch entities.** A multi-branch salon is one tenant (D28/D38).
- **Skill levels or ratings** on the specialist↔service mapping. The mapping is
  a boolean (D28).
- **Payment configuration.** No payments in v1 (D18); cancellation charges are
  recorded as owed and settled offline (D20/D24).
- **Live queue data on the public page.** Queue length and current wait are
  produced by E3 and appear when E3 lands (D31).
- **Offers, coupons and broadcast configuration.** That is E5.
- **Per-service enablement of booking modes.** Modes are salon-level (D35).
- **Capability authoring by salons.** The ten capabilities are fixed (D32).

## 4. User Stories

### Salon setup

1. As an **owner**, I want to create my salon and have a starter catalog already
   present, so that I am editing a list rather than staring at an empty screen.
   - Seed catalog ships with typical services and durations; **prices are
     blank** (D36).
2. As an **owner**, I want to add, rename, and remove services, so that the
   catalog matches what I actually sell.
3. As an **owner**, I want to set a price against each service, so that
   customers stop phoning to ask.
4. As an **owner**, I want to set how long a service takes, so that the system
   can estimate a day honestly.
5. As an **owner**, I want to set an optional buffer after a service, so that
   cleanup and chair reset are accounted for and the day does not run
   progressively late (D39).
6. As an **owner**, I want a service I no longer offer to disappear from the
   public page without destroying the history of times it was performed.

### Staff and permissions

7. As an **owner**, I want to add a staff member by name and phone and have them
   invited automatically, so that I do not have to create credentials for anyone
   (D33).
8. As a **staff member**, I want to receive an invite and set my own password,
   so that my work log is genuinely mine (D25, D33).
9. As an **owner**, I want to define the roles my salon actually has — anywhere
   from just me, to owner plus manager plus receptionist plus stylists — so that
   the system fits my shop rather than the other way round (D27).
10. As an **owner**, I want to build each role out of specific capabilities, so
    that I can let a receptionist book on someone's behalf without also letting
    them change prices (D32).
11. As an **owner who also cuts hair**, I want to hold both the owner role and
    the specialist role, so that I appear in the queue like any other stylist
    (D27).
12. As an **owner**, I want to be certain I cannot accidentally remove my own
    ability to administer the salon, so that I can never lock myself out.
    - The owner role holds every capability and cannot be deleted or stripped
      (D32).
13. As an **owner**, I want to say which services each specialist performs, so
    that nobody is offered a service they cannot deliver (F02's "free specialist
    who cannot cut my hair" problem).
14. As an **owner**, I want to set a duration per specialist for the same
    service, so that a faster stylist is not estimated as if they were the
    slowest (D6).
15. As an **owner**, I want to charge more for a more senior stylist, so that
    pricing reflects the shop's reality (D37).
16. As a **staff member**, I want to see the durations recorded against me, so
    that I know what the system expects of me (D28).

### Hours and closures

17. As an **owner**, I want to set the salon's weekly opening hours, so that
    customers cannot book when the shop is shut (D7).
18. As an **owner**, I want to set each specialist's own working days and hours
    inside the salon's hours, so that part-time staff are represented correctly
    (D34).
19. As an **owner**, I want to mark a one-off closure like a public holiday, so
    that a single date overrides the weekly pattern (D40).
20. As an **owner**, I want to mark an individual's leave, so that one person
    being away does not close the shop (D40).
21. As an **owner**, I want to close the shop right now in an emergency, so that
    no further bookings are taken today (D7).
22. As an **owner** closing mid-day, I want to see exactly which committed
    bookings fall inside the window I just closed and decide each one, so that I
    neither strand a customer silently nor turn people away who I could still
    serve (D29).
23. As a **customer whose booking was cancelled by an emergency closure**, I want
    it to be clear the salon cancelled and not me, so that I am not charged and
    not recorded as a no-show (D29, D24).

### Booking modes and pricing rules

24. As an **owner**, I want to choose which booking modes my salon offers, so
    that a next-available-only shop is not forced to expose slot booking (D2,
    D35).
25. As an **owner**, I want each mode to carry its own price and its own
    cancellation rule, so that a slot booking can carry a charge while
    next-available stays frictionless (D11, D35).
26. As an **owner**, I want to choose how VIP is priced — a surcharge, a
    multiplier, or a flat replacement price — so that it matches how I actually
    treat priority customers (D28).
27. As a **developer**, I want to add a new booking mode, specialist-selection
    mode, or cancellation rule without a schema migration, so that "easily
    doable in the system" (D2) is true in practice (D9, D28).

### The public page

28. As a **visitor with no account**, I want to see the salon's services and
    prices, so that I do not have to phone to ask (D23).
29. As a **visitor with no account**, I want to see opening hours and whether
    the shop is open right now, so that I do not travel to a closed shop (D23).
30. As a **visitor**, I want "open" to distinguish between *the shop's hours say
    open* and *there is actually a stylist in*, so that I do not arrive at 9:05
    to an empty chair (D34).
31. As a **visitor with no account**, I want to see **no personal information** —
    not customer names, not bookings — because I have not logged in and have no
    right to it (D23).
32. As an **owner**, I want a price change to appear on the public page
    immediately, so that nobody is quoted an out-of-date figure (F04).
33. As a **customer with an existing booking**, I want the price I was quoted to
    be the price I pay, even if the salon changes its catalog before my
    appointment (D30).

### Permission and failure cases

34. As a **staff member without `manage_catalog`**, I want the catalog to be
    read-only for me, so that I cannot change prices by accident.
35. As an **owner**, I want to be stopped from making a specialist available
    outside the salon's own opening hours, so that the schedule cannot describe
    an impossible day (D34).
36. As an **owner**, I want to be stopped from removing the last specialist who
    can perform a service that has upcoming bookings, or at least warned loudly,
    so that I do not silently orphan a committed appointment.
37. As an **owner**, I want to be warned before publishing a service with no
    price, so that the public page never shows a blank figure (D36).

## 5. Functional Requirements

### 5.1 Tenancy

- The system is **multi-tenant**. A salon is the tenant boundary; every
  configuration entity below belongs to exactly one salon (D38).
- A **multi-branch salon is a single tenant**. There is no branch entity, and
  branches are not separately configurable (D28/D38).
- **Customer identity is global; the customer↔salon relationship is per salon.**
  One login works across salons; history and CRM are held per salon (D38).
- The demo seeds **one** salon. The scoping must nonetheless be real, because
  D2 requires per-salon configuration everywhere.

### 5.2 Capabilities and roles

- There are exactly **ten capabilities**, authored by developers and not
  editable by salons (D32):
  `manage_catalog`, `manage_staff`, `manage_hours`, `manage_modes`,
  `confirm_bookings`, `perform_services`, `book_on_behalf`, `settle_charges`,
  `view_analytics`, `receive_escalations`.
- A **role is a salon-defined, named bundle of capabilities.** A salon may
  define between one and five roles, or any number in that range; there is no
  fixed role enum (D27).
- **A person may hold more than one role.** Their effective permissions are the
  union of their roles' capabilities (D27).
- **The owner role always exists, holds every capability, and can be neither
  deleted nor stripped** (D32). It is the guaranteed fallback for D27's
  escalation routing and the guarantee a salon cannot lock itself out.
- **No authorization check may test a role name.** Every check is against a
  capability. A salon with no role called "admin" must still work (D27, D32).
- `perform_services` is structurally significant: holding it is what makes a
  person a specialist — it puts them in the queue and entitles them to
  service mappings and durations (D32).
- Authorization is enforced **server-side**. Hiding a control in the UI is not
  an access control.

### 5.3 Staff provisioning

- A user with `manage_staff` creates a staff record from **name and phone**
  (D33).
- The system sends an **invite** over SMS or WhatsApp (D8; delivery is mocked
  per D21). The staff member sets their own password on first open.
- **Self-signup as staff is not permitted.** A salon cannot have strangers
  claiming to be its stylists (D33).
- **Login-less staff records are not permitted.** D25 makes each specialist
  personally accountable for their own work log, which requires an account.
- An invite that is never accepted leaves the staff record in a **pending**
  state: visible to the owner, not offered to customers, not in the queue.

### 5.4 Service catalog

- A service has: name, base duration, base price, optional buffer, and active
  flag.
- **Buffer is applied after the service** when scheduling, and is set at salon
  level per service (D39). Learned durations (D6) cannot capture it, because
  start/complete brackets the service and not the cleanup that follows.
- Deactivating a service removes it from the public page and from new bookings,
  and **preserves historical records** of the times it was performed.
- Deactivating a service that has **upcoming bookings** triggers the at-risk
  mechanism (§5.11). Previously this hazard was guarded only on the
  specialist-mapping path; it applies identically here (D43).
- A service with **no price** may exist in draft but must not appear on the
  public page; the owner is warned (D36).

### 5.5 Specialist↔service mapping

- The mapping is a **boolean**: a specialist either performs a service or does
  not. There is no skill level and no rating (D28).
- The mapping row carries **that specialist's duration** for that service (D6).
  Absent an entry, the service's base duration applies.
- The mapping row may carry **that specialist's price override** for that
  service (D37). Absent an entry, the service's base price applies.
- Durations are **visible to every staff member**, not owner-only (D28).
- Removing the last specialist mapped to a service that has upcoming bookings
  triggers the at-risk mechanism (§5.11), not a bespoke warning (D43).

### 5.6 Pricing

- **Base price per service**, with an **optional per-specialist override**
  (D37).
- **The price is captured onto the booking at commit time and honoured at
  service time.** Catalog price changes apply to new bookings only (D30).
- Public-page and catalog price changes take effect **immediately** for new
  bookings (F04).
- Where the customer chooses a **named specialist**, the applicable price is
  known before commitment and quoted directly (D37).
- **A slot booking assigns the specialist at booking time** (D42). That is what
  D13's "protected capacity" reserves — a named person's time, not an anonymous
  chair. The price is therefore known before commitment and D30 captures it.
  The same holds for VIP and prebook.
- **Only next-available leaves the specialist unknown at request time**, and
  only there does the confirmation window apply (D37, D42): assign, disclose the
  resulting price, then open a **confirmation window of 10 minutes, capped at
  the time remaining until the customer's turn**. Silence confirms. This is safe
  only because cancellation on next-available is free (D11) — the auto-confirmed
  customer who dislikes the price simply cancels at no cost.
- **There is no charge-bearing variant of this rule.** An earlier draft
  specified quoting the base price and having the salon absorb the premium on
  charge-bearing modes. D42 removes the need: on those modes the specialist, and
  so the price, is known at commit. It also removed a defect — that rule
  silently overcharged whenever the assigned specialist was *cheaper* than base.
- **During the confirmation window the Pending booking consumes capacity.**
  This is E2's definition of Pending, recorded here as a dependency rather than
  an E1 requirement: an availability calculation that ignores Pending
  double-books (D44).
- E1 **stores** these prices and rules. Applying them at booking time is E2's
  responsibility; E1's obligation is to make the rule unambiguous and
  representable, and to expose price resolution as one shared module so the two
  epics cannot diverge (§7).

### 5.7 Hours, off days and closures

- **Salon hours are the outer bound.** Per-specialist availability sits inside
  them and is validated against them; a specialist cannot be available while the
  shop is shut (D34).
- **Narrowing salon hours clamps specialist availability to the new bounds and
  reports what changed. It does not block the save** (D43). Blocking is the
  wrong behaviour when D7's entire purpose is letting the owner close the shop
  at will, including in an emergency — a validation error is not what anyone
  needs at that moment.
- **Recurring weekly off days** live in the hours model, at salon and specialist
  level (D40).
- **One-off dated closures are exception entries that override the weekly
  pattern**, at either salon or specialist level. "The shop is shut on Diwali"
  and "Ravi is off next Tuesday" are the same mechanism (D40).
- **Two distinct notions of open** exist and must not be conflated (D34):
  - *Configured* hours govern **booking availability** (D7; D19's auto-accept
    already tests "inside the artist's configured working hours").
  - *Actual* starts, via the artist's start-the-day control (D4), govern the
    **live queue**.
- **A live mid-day closure is an exception entry created for today** (D7, D40).
  On creating one:
  - New bookings inside the closed window stop **immediately**.
  - Bookings **already committed** inside that window are **not auto-cancelled**.
    They are flagged *at risk*, and the owner is presented with the list and one
    decision per booking: **notify and cancel**, or **keep** (D29).
  - Cancellations arising this way are **salon-initiated**: no `charge_owed` is
    raised against the customer (D24), and they are **tagged distinctly** so that
    E4's lost-demand reporting does not record them as customer churn (D29).

### 5.8 Booking modes, selection modes and cancellation rules

- Three independent, developer-extensible axes (D28):
  **booking mode**, **specialist-selection mode**, **cancellation rule**.
- Adding a new type on any axis is a **developer change plus configuration
  values**, and must not require a schema migration (D9).
- **Booking modes are enabled per salon, not per service** (D35).
- Each enabled mode carries **its own price** and **a reference to one
  cancellation-rule type plus that rule's parameters** — e.g. slot →
  `percentage_charge` at 50%; next-available → `no_charge` (D35, D11).
  Referencing rather than embedding is what keeps the axes independent: adding a
  cancellation rule must require touching no mode.
- **VIP pricing supports all three shapes** — surcharge, multiplier, and flat
  replacement price — and the salon chooses one (D28).
- VIP may carry a cancellation charge or none, at the salon's discretion (D17).

### 5.9 Public salon page

- Reachable **without authentication** (D22, D23).
- Shows: services, prices, opening hours, and open/closed state (D23, D31).
- **E1 shows the configured open/closed state only.** The "how many stylists
  are actually in" line depends on D4's start-the-day signal, which is an E3
  control, so it ships in the E3 section alongside queue length — not in E1
  (D31). D34's "show both, never collapse to one boolean" remains the target
  end state; it is simply not reachable within E1's delivery.
- Shows **no personal data of any kind** — no customer names, no bookings, no
  staff contact details (D23).
- **Queue length and current wait are produced by E3** and appear as a section
  on this page when E3 lands. They are absent until then (D31).

### 5.10 Seed catalog

- A new salon starts from a **default catalog** — haircut, beard trim, shave,
  colour, facial, waxing and similar — carrying **typical durations** and
  **blank prices** (D36).
- Prices are blank deliberately: a wrong default price could go live on the
  public page, whereas a wrong default duration only nudges an ETA that D6
  corrects from history.
- The seed is **editable in full**: rename, remove, add.

### 5.11 Config changes that invalidate committed bookings — the at-risk mechanism

Four separate rules in earlier drafts were one rule in disguise: **a
configuration change has invalidated something already committed.** They are
unified here (D43).

**Triggers** — any of the following, where committed bookings fall inside the
affected set:

| Trigger | Section |
| --- | --- |
| Mid-day or dated closure (D7, D29, D40) | §5.7 |
| Salon hours narrowed | §5.7 |
| Service deactivated | §5.4 |
| Last specialist mapped to a service removed | §5.5 |

**Behaviour**, identical in every case:

1. New bookings against the affected configuration stop **immediately**.
2. Bookings **already committed** are **never auto-cancelled**.
3. The owner is shown the at-risk list and makes **one decision per booking**:
   **notify and cancel**, or **keep**.
4. Cancellations arising this way are **salon-initiated**: no `charge_owed` is
   raised against the customer (D24), and they are **tagged distinctly** so
   E4's lost-demand reporting never records them as customer churn (D29).
5. The default is **keep**, and cancel-all is available as a single action. An
   owner closing the shop urgently will not work through a list item by item.

**Delivery boundary.** This mechanism reads bookings, which are E2's entity.
It is **specified here because E1 config changes trigger it, and delivered with
E2** (D45). E1 ships the configuration writes; the at-risk list arrives with the
epic that owns bookings. E1's own dependency for *delivery* remains none.

### 5.12 State and validation summary

| Rule | Enforcement |
| --- | --- |
| Specialist availability ⊆ salon hours (on staff edit) | Reject on save (D34) |
| Salon hours narrowed | Clamp staff availability, report; never block (D43) |
| Owner role holds all capabilities | Not editable (D32) |
| Authorization by capability, never role name | Server-side (D32) |
| Service on public page has a price | Warn, and exclude until priced (D36) |
| Any config change invalidating a committed booking | At-risk mechanism, §5.11 (D43) |
| Slot / VIP / prebook specialist | Assigned at booking time (D42) |
| Pending booking during confirmation window | Consumes capacity (D44) |
| Committed price | Immutable after commit (D30) |
| Duration or price-override change | Records actor and timestamp (D41) |

## 6. Non-Functional Requirements

- **Configuration changes are visible immediately.** A price or hours change
  must be reflected on the public page on the next request, with no cache that
  outlives the change (F04's live-accuracy requirement).
- **Authorization is server-side and capability-based.** A UI that hides a
  control is not an access control; every mutation re-checks the capability
  (D32).
- **The public page exposes no personal data.** This is a correctness
  requirement, not a preference — D23 makes it the boundary of unauthenticated
  access.
- **Configurability is not mockable.** D21 permits mocking payment, message
  delivery and duration learning. It explicitly does **not** permit faking D2/D9
  extensibility, because retrofitting it is a rewrite.
- **Pay-relevant configuration changes are attributable** (D41). Changes to
  **durations** and **per-specialist price overrides** record the actor and a
  timestamp. This is deliberately narrow — it is two columns on two kinds of
  change, not the general audit log that §8 rules out. The justification is
  concrete: D25 makes these numbers a pay record, and a pay dispute with no
  record of who changed what is unresolvable.
- **Scale is deliberately small.** One seeded salon, single-digit staff,
  low-tens of services. No performance engineering is warranted beyond avoiding
  obvious N+1 reads on the public page.
- **Degradation:** if the catalog cannot be loaded, the public page must show an
  honest error rather than an empty catalog that implies the salon offers
  nothing.

## 7. Implementation Notes and Architecture

**There is no existing code.** This is a greenfield repository — the only
contents today are documentation and skills. Nothing below ties back to
existing classes, endpoints or tables, because none exist. This section is a
starting shape, not a description of what is there.

### Entities

```
Salon (tenant)
├── Role            name + capability set          (salon-defined, D27)
├── StaffMember     user, roles[], invite state    (D33)
├── Service         name, duration, price, buffer, active   (D39)
├── ServiceMapping  (staff × service) → duration?, price?   (D28, D6, D37)
├── HoursRule       weekly pattern, salon or staff level    (D34, D40)
├── HoursException  dated override, salon or staff level    (D40)
└── ModeConfig      mode type + price + cancellation-rule ref + params (D35)

Capability          fixed enum of 10, developer-authored    (D32)
User                global identity, spans salons           (D38)
```

### The three extension points

The one part of E1 that cannot be mocked. Each axis is a registry of
developer-authored types; a salon's configuration is a **selection from the
registry plus parameter values**, never a rule definition (D9):

| Axis | Registered types (initial) | Salon configures |
| --- | --- | --- |
| Booking mode | next-available, slot, VIP, prebook | which are enabled, price each |
| Specialist selection | fastest turn, named specialist | which are enabled |
| Cancellation rule | `no_charge`, `percentage_charge`, `flat_charge` | which rule per mode, and its parameters |

Adding a fourth booking mode must be: register the type, ship it, salons enable
it. No migration, no change to `ModeConfig`'s shape.

### Consumers

E1 is upstream of everything, so its model is a contract:

- **E2** reads catalog, mappings, hours, modes and prices to compute
  availability and apply D37's pricing rules.
- **E3** reads durations, buffers and mappings to estimate the queue, and the
  actual-start signal to decide the live open state.
- **E4** reads prices for revenue and the work log as a pay record.
- **E5** reads prices to compute discounts.

### Modules worth isolating for testing

- **Capability resolution** — given a person and a salon, produce the effective
  capability set. Pure, and every authorization decision depends on it.
- **Hours resolution** — given a salon, a person and a date, produce the working
  window after applying weekly rules and dated exceptions. Pure, and both E2's
  availability and E3's queue depend on it being right.
- **Price resolution** — given a service, a mode and optionally a specialist,
  produce the applicable price under D37's rules. Pure, and the collision
  handling makes it the most error-prone logic in E1.

## 8. Out of Scope

Deferred deliberately, with the decision that put them here:

- Rule-definition UI for authoring mode types — developers author them (D9).
- Branch entities and per-branch configuration (D28/D38).
- Skill levels, ratings or seniority on service mappings (D28).
- Payment configuration, card capture, and the 50% slot block (D18, D20).
- Live queue length and current wait on the public page — E3 (D31).
- Offers, coupons, segments and broadcast configuration — E5.
- Per-service enablement of booking modes (D35).
- Salon-authored capabilities (D32).
- Consent, retention and messaging opt-out configuration — open item **i**,
  belongs with E4/E5.
- Full audit history of configuration changes. The **narrow** pay-relevant
  attribution of D41 is in scope (§6) — actor and timestamp on duration and
  price-override changes only. A general change log over all configuration is
  not.

## 9. Risks and Open Questions

### Risks

1. **The extension-point design is the one thing that cannot be corrected
   later.** D2 says retrofitting configurability is a rewrite, and D21's
   permission to mock explicitly excludes it. If the registry shape is wrong,
   all four downstream epics inherit it. *Mitigation: prove it by adding a
   fourth mode type as an exercise before E2 starts.*
2. **Capability checks are easy to implement cosmetically.** If any check tests
   a role name, or if the server trusts the client, D27's flexible role model
   silently becomes a fixed one. *Mitigation: capability resolution as an
   isolated, tested module.*
3. **Seed durations could be quietly wrong.** An owner who accepts the seed
   without editing gets plausible-looking but inaccurate estimates, which
   degrades E3's ETAs before D6 has any history to learn from. *Mitigation:
   surface seeded-and-unedited durations to the owner during setup.*
4. **Per-specialist pricing is the most intricate logic in E1** and its rules
   live half in E1 (storage) and half in E2 (application). A divergence between
   the two produces a customer quoted one price and charged another. *Mitigation:
   price resolution as a shared pure module, not reimplemented in E2.* **Reduced
   by the stage-3 gate:** D42 established that only next-available leaves the
   specialist unknown at commit, which deleted the charge-bearing branch
   entirely — and with it a defect that overcharged whenever the assigned
   specialist was cheaper than base.
5. **The at-risk flow (§5.11) is a human-in-the-loop step in an emergency.** An
   owner closing the shop urgently may not work through a list. *Mitigation:
   default to keep, cancel-all as a single action, never auto-decide — now
   specified in §5.11 rather than left as an intention.*
6. **E1 is not as independent as EPICS.md claimed.** The at-risk mechanism reads
   bookings. Specification and delivery are now separated (D45), but the
   sequencing risk is real: if E2 slips, E1 ships with configuration changes that
   can silently orphan bookings and no list to catch them. *Mitigation: until E2
   lands, the triggering config changes should warn generically that committed
   bookings may be affected, even without the ability to enumerate them.*

### Open questions

**Blocking: none.** E1's stage-1 frontier is empty.

Non-blocking, carried elsewhere:

| # | Question | Where it belongs |
| --- | --- | --- |
| b | "Best artist" by what measure? D28 removed the ranking data; D37's per-specialist price is a candidate proxy | E2 stage 1 |
| i | Consent, retention, messaging opt-out | E4/E5 stage 1 |

### Fragile assumptions

- **That ten capabilities are enough.** The set was derived from the roles named
  in the source. A salon structure nobody described may need an eleventh.
- **That a salon-level buffer per service is sufficient.** A slower stylist
  plausibly needs a longer buffer, exactly as they need a longer duration. If
  that proves true, buffer moves onto the mapping row alongside duration.
- **That one seeded salon is enough to exercise multi-tenancy.** Tenant leaks
  are usually found by having two tenants, not one.
- **That "the owner can never lock themselves out" holds under multi-role.** The
  guarantee is stated (D32); it needs a test, not just an intention.

## 10. Success Metrics

Demo-appropriate and directional — there is no production traffic to measure
(D21).

**Setup:**

1. A salon goes from created to fully configured — catalog priced, staff added,
   hours set, modes enabled — in **under ten minutes** starting from the seed.
2. Adding a salon requires **zero code changes**.

**Extensibility (the load-bearing one):**

3. Registering a **fourth booking mode type** requires a developer change plus
   configuration only — **no schema migration**, and no change to how existing
   modes are stored. This is the pass/fail test for D2 and D9.

**Public page:**

4. The page answers all three of the common enquiry questions — what services,
   at what price, open when — without login.
5. The page exposes **zero** personal data fields. Pass/fail, not directional.

**Downstream readiness:**

6. E2 can compute a correct availability window for a given service, date and
   specialist using only what E1 stores — no supplementary configuration
   invented in E2.
