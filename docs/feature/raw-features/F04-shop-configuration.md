# F04 — Shop configuration & catalog

**From:** problems 1 (prices/enquiry), 3 (holidays), 5 (schemes)

The source states a hard architectural constraint: **everything salon-facing
must be configurable per salon.** Modes, services, and prices "can differ from
salon to salon". Treat this as a multi-tenant configuration requirement, not a
settings page.

## Capabilities

- Service catalog: name, duration, price, which specialists can perform it
- Per-salon pricing — must be live-accurate; source insists price changes are
  reported immediately
- Booking modes enabled/disabled per salon, each with its own price and
  cancellation policy
- Working hours, off days, holidays, per-specialist availability
- Public info page answering what customers currently phone to ask
  (services, prices, hours) — source argues explicitly that anything a salon
  will say on a call, it will publish

## Dependencies

Nothing. F04 is upstream of F01, F02, F07, F08, F09.

## Open questions

- **How generic is "generic"?** A fixed schema with per-tenant values is a
  week. A user-defined schema (salons invent their own modes and rules) is a
  rules engine. Source says "very very generic" without bounding it. This one
  answer changes the build size more than any other in the document.
- Who edits config — owner only, or admin role too?
- Is service duration a single number, or per-specialist?
- Multi-branch salons: one tenant or many?
