# F07 — Business analytics

**From:** problems 4 (peak staffing), 6 (business analyst)

Owner-facing intelligence. Source frames it ambitiously — "works as a business
analyst" — but every stated question is answerable from F01/F02/F05/F09 data.
No external data source is implied.

## Capabilities

**Staffing:**
- Peak hours and peak days
- "Which days do I need an extra member"
- "Which days can I take off without burning the pocket" — off-day cost model

**Utilisation:**
- Where the time goes
- Time per service, per specialist
- Break patterns (see F05 caveat)

**Revenue and loss:**
- Sales reporting
- Lost deals, with reasons
- Growth-area identification
- New-customer volume per month/year

## Dependencies

Everything transactional. F07 is read-only over other features' data — it should
ship *after* them, not alongside.

## Open questions

- **"Lost deals due to xyz reasons" cannot be measured for the main loss
  channel.** The source's own loss cases are missed calls and customers who
  walked out — both invisible to a system that never sees the phone. Once call
  agents are out of scope, the largest category of lost deal is unobservable.
  Analytics can only report abandoned in-app bookings and cancellations. This
  gap should be stated plainly rather than papered over.
- "If the customer is satisfied and finds new shop more good" (#6) implies churn
  detection or satisfaction capture. Neither has a data source. Survey? Absence
  of return visits?
- Is this a dashboard, a scheduled report, or a voice query surface?
- What volume of history makes any of this statistically meaningful? A single
  salon generates thin data.
