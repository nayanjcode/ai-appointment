# F06 — Customer profile & CRM

**From:** problems 7 (know the customer), 6 (retention analytics)

Per-customer intelligence. Distinguished from F07 by audience: F06 answers
"who is this person in front of me", F07 answers "how is the business doing".

## Capabilities

- Customer record: visit history, services used, spend
- Last visit: when, what, with whom, at what price
- Preference and trend detection — what they typically buy
- Upsell hints: "what more can be sold" to this specific customer
- Relationship context — source phrases it as "what are the relations of this
  person with me"
- New-customer tracking: who arrived this month/year, and retention follow-up
  so they "are not lost"
- Communication guidance — "barber knows how to communicate", pitch strategy
  for converting a new customer to a regular

## Dependencies

F01 (booking history), F09 (spend), F04 (service catalog).

## Open questions

- **"Relations of this person with me" is undefined.** Family grouping,
  referral chain, or loyalty tier? This also underpins F01's VIP-for-relatives
  rule, so it cannot stay vague.
- **Upsell hints imply a recommendation engine.** Rule-based (co-occurrence) or
  model-based? Cold start with no history?
- Consent and data retention are not mentioned anywhere in the source. Storing
  behavioural profiles of named individuals carries obligations regardless of
  whether the source raised them.
- How is a walk-in customer identified — phone number, name, nothing?
