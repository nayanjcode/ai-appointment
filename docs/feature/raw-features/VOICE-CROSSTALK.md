# Voice cross-talk — analysis

The problem from the source: multiple specialists in one room, one person's
voice command captured by another's session.

## Why the proposed mitigations don't close it

Three were proposed: voice-command confirmation; only act if the speaker is
marked available; per-artist authentication to start the day.

All three are **authorization** controls. They answer *"is this command
permitted?"* Cross-talk is a **capture** problem — *"whose voice was that?"*

The case they miss is the common one:

> Artist A and Artist B are both authenticated, both on duty, both mid-service,
> in the same room. B says "mark complete." A's session hears it. **Every
> authorization check passes** — B is a valid, available, authenticated user.
> The command lands on A's queue.

Availability gating only helps when the interfering speaker is *off* duty, which
in a busy salon is the rare case. It is worth having — it shrinks the blast
radius — but it is not the fix.

Confirmation is genuinely valuable and should stay: it converts a silent wrong
write into a visible wrong prompt. But it fires on A's screen, about B's words,
while A's hands are wet. Under load, people confirm reflexively.

## Options that address capture

Ordered by leverage per unit of cost.

### 1. Near-field microphone per artist — highest leverage

A boom or bone-conduction mic at 5 cm has a very large signal-to-noise advantage
over a room mic at 3 m. This is the standard fix in noisy shared environments
(warehouses, kitchens, drive-throughs) and it attacks the physics rather than
patching the software. Roughly $20–50 per artist. Also mitigates the separate
salon-acoustics risk — dryers, clippers, music.

### 2. Device-per-artist session binding

Each artist's session lives on their own phone or tablet. Natural pairing with
(1), and already implied by per-artist authentication. Removes any notion of a
shared room mic.

### 3. Push-to-talk or wake word

The device listens only on demand. Nearly eliminates ambient capture.

- **Push-to-talk** conflicts with the wet-hands problem — unless it is a foot
  pedal or a wearable button, which sidesteps it entirely.
- **Wake word** scoped per artist keeps hands free but is weaker alone; paired
  with (1) it is reasonable.

### 4. Voice enrollment / speaker verification — defer

Each artist enrolls a voiceprint; commands not matching the session owner are
rejected. Directly targets the problem, but accuracy degrades exactly where it
is needed — background noise, similar voices, raised speech — and it adds
latency to every command. Expensive to get right. Not a v1 control.

### 5. Confirmation before commit — keep, as the last layer

Already decided. Effective as defence-in-depth, ineffective as the primary
control.

## Decided (D15)

**No external hardware.** Phone audio for both customers and staff, plus:
distinct per-artist wake word, clarify-and-reject when a stray command lands,
confirmation before commit, availability gating, per-artist auth.

**Customers cannot start or stop a service** — after booking they may only
cancel or read. Cross-talk can therefore only affect staff writes, of which
there are four.

### Why this is defensible

The wake word is the load-bearing part. It moves the system from "always
listening to the room" to "listening only when addressed", so B's unprefixed
"mark complete" never reaches A. That is capture-level, not authorization —
which is what the earlier analysis said was missing.

### Two residual risks

**Phone position.** A phone mic is near-field only if the phone is near the
mouth. On a counter 2 m away with wet hands it behaves as a room mic, and the
wake word then has to survive salon noise. A Bluetooth earbud paired to the
phone the artist already owns restores near-field *and* hands-free for ~$15–20 —
no new system, no new device to manage. Worth a trial before concluding the bare
phone is sufficient.

**Alert fatigue.** Each cross-talk event costs two people: a spurious prompt for
A, a repeat for B. If frequent, A confirms reflexively and the backstop stops
working. Wake words should keep this rare — instrument it and check.

### Deferred

Voice biometrics (option 4 below) remains a v2+ consideration. Near-field
hardware is not adopted, but the earbud path stays open if wake-word accuracy
disappoints in a real salon.
