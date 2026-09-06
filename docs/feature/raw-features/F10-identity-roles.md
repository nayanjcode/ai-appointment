# F10 — Identity, roles & permissions

**From:** problem 1 (receptionist delegation duties), owner narrative (role layers)

Stated roles: **admin, receptionist, hair specialist, customer** — plus owner,
who appears throughout the narrative as distinct from admin.

## Capabilities

- Authentication for each role
- Role-scoped permissions
- Delegation: booking on behalf of another person — the receptionist's core duty
- Specialist ↔ service capability mapping (which specialist can do what) — F02
  depends on this to avoid the "free specialist who cannot cut my hair" problem
- Multi-tenant isolation: staff and customers belong to a salon (F04)

## Dependencies

Upstream of everything.

## Open questions

- **Is owner a separate role from admin?** The narrative uses both. Five roles
  or four?
- **The receptionist role is contradictory.** The product's stated value is
  eliminating the receptionist, yet receptionist is a listed role layer.
  Presumably: support it where one exists, make one unnecessary elsewhere. Needs
  confirming — it decides whether receptionist is a v1 role or never built.
- Customer identity: account required, or phone-number-only / guest booking?
  Guest booking conflicts with F06 (history) and F09 (cancellation charges).
- Can one person hold two roles (owner who also cuts hair)? Common in this
  segment; the source's "single person manager" case is exactly this.
