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

- **Largely resolved by D1.** Walk-ins are booked, so walkouts record as
  cancellations; the "wanted to book" button captures turned-away demand; and
  lapsed-regular detection covers churn via absence of return visits. Missed
  *phone* calls remain unobservable, but they are no longer the only signal.
- Still open: is churn inferred purely from absence, or is satisfaction captured
  directly (survey, rating)? D1 implies the former; F01's "best artist" mode
  (O5b) would need the latter.
- Is this a dashboard, a scheduled report, or a voice query surface?
- What volume of history makes any of this statistically meaningful? A single
  salon generates thin data.
