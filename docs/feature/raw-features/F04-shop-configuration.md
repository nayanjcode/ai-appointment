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

- **Answered (D2): the rules-engine end.** Salons configure which modes exist,
  their prices, and can introduce new modes — of booking, specialist selection,
  cancellation, and more. Must be designed in from day one.
- **Still open (O1): who authors a new mode?** Owner via a rule-definition UI,
  or a developer making a low-effort change. Different products.
- Who edits config — owner only, or admin role too? (See also O5e: is admin
  even a distinct role from owner?)
- Owner can change shop timings live, including to stop taking orders in an
  emergency (D7). What happens to bookings already made in a window that is
  then closed?
- Is service duration a single number, or per-specialist?
- Multi-branch salons: one tenant or many?
