# F05 — Specialist workspace

**From:** problems 1 (wet hands, off days), 2 (queue), 4 (peak staffing)

The specialist-facing surface. Also the **source of truth for F02's live data** —
without start/stop/pause events from here, every ETA is fiction.

## Capabilities

- My queue: who is next, what service, how long it should take
- Who is coming, with prep context from F06 ("if he wants to be prepared for
  anyone")
- Service lifecycle controls: start, pause, resume, complete — these emit the
  events F02 consumes
- Break tracking
- Work log: services performed, time taken
- Off-duty mode — the stated outcome is "off day calls reduced to zero"
- No manual "your turn has come" call or message; the system notifies

## Dependencies

F01, F02 (bidirectional), F06 (prep insights), F10 (role).

## Decided (see DECISIONS.md)

- **Shop start/stop control (D4).** Artist marks the shop open, or themselves
  available for the day.
- **Confirmation duty (D3/D10).** Artist confirms within a fixed window; pending
  bookings are rejected at end of office hours.
- **Buttons are the UI surface only (D14).** In the chatbot these are voice
  commands, and the assistant proactively surfaces them.

## Open questions

- **Break tracking is surveillance.** Source frames it from the owner's side:
  "can know how much break the hair specialist takes." That is a labour-
  relations decision, not a feature toggle. Who sees this data, and do
  specialists know?
- **Three controls now depend on the artist remembering to press something:**
  service start/stop, shop start/stop (D4), and booking confirmation (D3). Each
  has an unspecified failure mode. Forgetting "stop" drifts every ETA;
  forgetting "start" may block bookings; forgetting to confirm strands a
  customer (O2). The product's reliability rests on this and it is undesigned.
- Can a specialist edit the queue directly, or only the system?
- Walk-in customers: who enters them, and where do they land in the queue?
