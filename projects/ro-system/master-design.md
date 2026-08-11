# Under-Sink RO System — Master Design Spec

*Living reference document. Phase 1 = physical build. Phase 2 = drift-monitoring data pipeline (ESP32 → Google Apps Script → Google Sheet, no local hub or dedicated computer required). Phase 1 includes physical hooks for Phase 2 so nothing needs re-plumbing later.*

---

## Source Water Profile

- Chlorinated public groundwater (not private well)
- Very soft (~6 ppm hardness) — remineralization stage is doing real work, not decorative
- DBPs present (TTHM, HAA5) but well under EPA MCL — confirms active chlorination, chlorine residual level unknown
- No coliform/E. coli in recent testing
- Iron/manganese untested — check with cheap test strips before finalizing prefilter media; no evidence of need currently
- **To confirm before finalizing parts:** actual feed pressure (psi) via hose-bib gauge. Note: with the feed isolation subsystem's flow restrictor in place, worst-case leak volume is bounded regardless of house pressure — this check now matters for a different reason, RO membrane rejection/production performance, which requires adequate driving pressure independent of the restrictor's leak-safety role. Rule of thumb: booster pump warranted below ~40 psi.

---

## Phase 1 — Piping Tree (locked)

```
House feed ─[Solenoid, N.C.]─[Flow restrictor]─[TAP-A: pressure + raw TDS, capped]─V0─ Sediment ─V1─ Carbon ─V2─ RO membrane
                    ▲                                                                                                │
                    │                                                                              [TAP-B: product TDS, capped]
          (controlled by timer                                                                                       │
           relay, see Feed                                              ─V3─ Permeate pump/Tank ─V4─ Post-carbon ─V5─ Remin ─V6─ UV ─V7─[Flow switch]─ Faucet
           Isolation Subsystem                                                                                                              │
           below)                                                                                                          triggers timer relay
```

- 8 valves (V0–V7), full independent per-stage isolation — every stage serviceable without disturbing or depressurizing any other stage
- All fittings 1/4" push-to-connect (John Guest–style), one consistent fitting language start to finish
- Waste/concentrate line: membrane → auto-flush valve (flow-restrictor + shutoff-triggered flush combo, non-electric) → check valve → drain saddle valve → P-trap
- Permeate pump provides: faster tank recovery, reduced membrane back-pressure/scaling stress (longer membrane life), and built-in auto-shutoff function (no separate ASO valve needed — confirmed via manufacturer literature)
- Feed isolation subsystem (solenoid + flow restrictor + dispense-line flow switch) — see dedicated section below for full design rationale

### Stage-by-stage

| Stage | Component | Notes |
|---|---|---|
| — | Feed adapter / saddle valve | Taps cold supply, 1/4" OD output |
| — | V0 | Ball valve, isolation |
| 1 | Sediment filter, 5-micron | Quick-change cartridge format preferred |
| — | V1 | |
| 2 | Carbon block | **Silver-impregnated/bacteriostatic** — suppresses biofilm growth in the media itself (documented mechanism, not marketing) |
| — | V2 | |
| — | RO membrane | 50–75 GPD sufficient for this demand profile |
| — | Auto-flush valve | On waste line — flushes concentrate at shutoff, prevents stagnant scaling; non-electric, ties to your own usage |
| — | V3 | |
| — | Permeate pump | Non-electric (Aquatec-style diaphragm), no power needed |
| — | Accumulator tank | 3.2-gallon nominal (delivers ~1.6–1.9 gal usable) — correctly sized to ~1.5 gal draw need, not oversized. NSF-listed sanitary butyl bladder. |
| — | V4 | |
| 3 | Post-carbon + Remin combo cartridge | Silver-impregnated for consistency; lower priority risk than pre-membrane carbon but cheap to match |
| — | V5 | |
| — | V6 | |
| 4 | UV sterilizer | **Terminal stage**, immediately pre-faucet — sterilizes every draw regardless of upstream dwell time. Needs power. |
| — | V7 | |
| — | Faucet | Non-air-gap style, 3/8"–1/2" hole in stainless sink deck (new drilled hole) |
| — | Drain check valve | Inline on drain tube, prevents backflow — required specifically because non-air-gap style chosen |

---

## Phase 1 — Automatic Feed Isolation Subsystem

### Design goal
Limit the duration house-line pressure is present on the feed side of the system (sediment through membrane), so that the worst-case volume from an undetected fitting failure is bounded to a short window rather than running unattended indefinitely.

### Mechanism
- **Normally-closed solenoid valve** at the very start of the feed line, before the flow restrictor and before the sediment stage
- **Passive inline flow restrictor** immediately after the solenoid — caps the maximum possible flow rate through the entire downstream chain regardless of any other component's state; a physical backstop requiring no power or logic
- **Flow switch** (mechanical paddle/vane type, Normally Open configuration — not a pulse-output flow meter) mounted on the dispense line near the faucet, downstream of the tank
- **Retriggerable delay timer relay**: the flow switch closing serves as the trigger input; each trigger resets the countdown to its full duration; the relay's output energizes the solenoid for the length of the countdown; countdown reaching zero de-energizes the solenoid, which fails closed by design

### Sequence of operation
1. System idle: solenoid closed, feed line unpressurized past the solenoid. Tank holds its own stored pressure independently and is unaffected.
2. Faucet opened: dispense draws immediately from the tank's stored pressure — zero delay, entirely independent of solenoid state.
3. Dispense flow trips the flow switch → timer relay triggers → solenoid opens → house pressure enters the feed line, produces through sediment/carbon/membrane, refills the tank.
4. Continued or repeated draws keep retriggering the timer, extending the open window for as long as needed.
5. Draws stop → timer counts down from the last trigger → reaches zero → solenoid closes → feed line isolated from house pressure again.

### Alternatives considered and why they were not used
- **Sensing directly on the feed line** (a pressure or flow switch positioned before the solenoid): rejected — creates a dependency loop where the sensor requires flow to trigger the valve that is itself required for flow to exist in the first place.
- **Sensing via tank pressure/level as the sole trigger**: rejected as a standalone mechanism — cannot distinguish "tank refilling normally after a draw" from "tank continuously draining through a downstream leak," since both present identically as sustained low pressure.
- **Low-flow-rated flow switch on the feed line**: technically workable but a specialty product category (switches reliably actuating under ~1 GPM) commands significantly higher cost than standard flow switches. Relocating the sensor to the dispense line avoids this entirely, since post-tank dispense flow rate is close to standard tap flow and falls within commodity flow-switch specifications.
- **A small secondary buffer tank immediately after the prefilters**, sized to shrink normal refill time enough to allow a tight backup timer: considered and workable, but adds a second tank and associated hardware. The dispense-line sensing approach achieves a comparable bounded-exposure outcome with fewer components.

### Fail-safe properties
- Solenoid is normally-closed: loss of power, a failed relay, a disconnected wire, or any electrical fault all default to the closed (safe) state
- Flow restrictor operates independently of all electronics — caps worst-case flow rate even if the solenoid fails open or is left open
- A hard backup maximum-duration cutoff, independent of the retriggerable countdown, is recommended as an additional safeguard against a stuck or falsely-triggering flow switch — specific duration value not yet set (open item)

### Component list
| Part | Function | Approx. cost |
|---|---|---|
| Water flow switch, Normally Open, 1/4" | Dispense-line trigger | Standard (non-low-flow) rated part, well under specialty low-flow pricing |
| Retriggerable delay timer relay module, 12V DC | Countdown logic, resettable on each trigger | Low cost, common electronics part |
| Solenoid valve, 12V DC, Normally Closed, 1/4" | Feed line isolation | Common part, same class already used in refrigeration/appliance water lines |
| 12V DC power supply | Powers relay + solenoid | Low cost |
| Inline flow restrictor, feed line | Passive worst-case flow cap | Low cost, no power required |

### Basis for periodic feed isolation not affecting filter media or membrane life
- Automatic shutoff valves that stop feed flow to the membrane once the storage tank reaches target pressure are standard, universal equipment in residential RO systems — periodic feed interruption on the timescale of minutes to hours is default industry behavior, not a novel condition being introduced here
- Membrane damage from drying out refers specifically to air exposure (an open or drained housing, typically during long-term decommissioning or membrane removal) — a sealed housing that simply stops receiving new feed water while remaining full of static water is a different condition and is not a documented failure mode
- Documented stagnation-related concerns (taste/quality degradation from standing water) apply to multi-day idle periods, not hours between draws, and are independently addressed by the terminal UV stage and post-carbon polish stage already specified upstream

---

## Phase 1 — Physical Mounting

- **Slide-out sled**, not fixed wall board: bottom-mount, full-extension drawer slides, rated 50–66 lbs minimum (accounts for full tank weight + all housings + pump + UV)
- Composite/plywood board mounted on top of the slide tray as the actual equipment-mounting surface (ready-made trays are general organizers, not plumbing-specific — board provides the real mounting surface)
- Tank rides on the sled with everything else — weight is floor-supported via slides, not wall-cantilevered, so the earlier "don't hang the tank" concern doesn't apply to this mounting method
- Service loops (a few extra feet of coiled 1/4" tubing) at feed/product/drain connection points — sled can slide most/all the way out while still connected for routine service; full disconnect only needed for major work
- Every valve handle oriented facing outward/forward — no reaching behind housings
- Exterior-grade plywood if building the board, sealed all faces/edges

### Noise mitigations (all standard, none exotic)
- Permeate pump on a rubber isolation pad/grommet — reduces thump transmission into board/cabinet
- Water hammer arrestor near feed if banging occurs from valve operation (cheap, add if needed rather than preemptively)
- Tubing secured with adhesive clips to prevent rattle against cabinet walls

---

## Phase 1 — Faucet & Sink Modification

- New hole drilled fresh into stainless sink deck (no spare pre-punched hole confirmed)
- Non-air-gap faucet style chosen — quieter, simpler, smaller hole (3/8"–1/2" vs. 1"–1-3/8" for air-gap)
- Requires check valve on drain line (see table above) — this is the tradeoff for skipping air-gap
- Drilling method: center-punch mark, step/unibit bit, cutting oil, go slow
- **Pick exact faucet model before drilling** — size hole to that model's actual stem diameter

---

## Phase 2 — Data Monitoring (no local hub, push-based)

### Design goal
Passive drift monitoring (is TDS rejection or feed pressure trending in the wrong direction over weeks/months) — not live/real-time monitoring, and explicitly not dependent on any always-on server, hub, or dedicated computer.

### Architecture
```
ESP32 (ESPHome, external-antenna board)
  reads: raw-water TDS (TAP-A), product-water TDS (TAP-B), feed pressure (TAP-A)
       │
       ├─ Long-interval push (baseline "still fine" heartbeat, ~180–240+ min, exact value TBD)
       │
       └─ Event-triggered push (fires on the feed isolation subsystem's flow switch
            transitioning to "flow detected" — i.e., the moments the system is
            actually producing, which are the most informative readings)
       │
       ▼
Google Apps Script Web App (deployed once, free, no subscription)
  validates: shared-secret match, exact expected JSON key shape, per-field type
  and sane-range checks — malformed or unauthorized payloads rejected outright
       │
       ▼
Google Sheet (the actual log — viewable/chartable from phone or laptop anytime,
  no server needed to view it, only to write to it)
```

- No Home Assistant, no local hub, no dedicated always-on computer of any kind
- Push chosen over pull: gapless regardless of whether any phone/laptop happens to be on or nearby; ESP32 itself is effectively always up (small board, plugged in continuously)
- Google Apps Script chosen over IFTTT-based push: IFTTT's free tier does not reliably include webhook support (confirmed as a paid-tier feature on current plans) — Apps Script is genuinely free with no subscription tier, at the cost of a small one-time script (write once, not an ongoing maintenance burden)
- GitHub-repo-based alternative (ESP32 commits to a file via API) considered and rejected: requires read-then-write API calls per update, generates a git commit per data point, and still needs a second hop to actually chart/view trends — no advantage over direct-to-Sheet for this use case
- Endpoint security: shared-secret field required in every payload, checked before any write; strict shape validation (exact expected keys, correct types, sane per-field ranges) rejects malformed or unexpected data; Google's one-time deployment consent (authorizing the script to write to the Sheet) is a manual browser step done once at setup, not a per-request auth burden — the ESP32 itself performs plain unauthenticated POSTs at request time

### Leak detection — decoupled entirely from the ESP32/data pipeline
- Standalone battery or plug-in leak detection puck, self-contained (local audible alarm), sitting in the sled's drip-pan area
- No network connection, no ESP32 involvement, no hub — deliberate simplification, since the feed isolation subsystem already bounds worst-case unattended leak exposure architecturally; a networked/smart leak alarm was judged to add no real value on top of that

### Physical hooks already in Phase 1
- **TAP-A**: capped 1/4" push-connect tee on feed line, pre-V0 — raw-water TDS + feed pressure reference
- **TAP-B**: capped 1/4" push-connect tee on membrane product line, pre-tank — product-water TDS reference (compared against TAP-A = actual rejection rate)
- Spare outlet/power strip capacity under sink beyond UV's needs — for ESP32 board power
- Physical space reserved in the sled's drip-pan area for the standalone leak puck

### Hardware plan
| Component | Choice | Why |
|---|---|---|
| Microcontroller | ESP32 board, **external-antenna variant** (e.g. "-U" suffix boards like ESP32-WROOM-32U) | Under-sink cabinets are effectively Faraday cages (metal sink, plumbing, cabinet walls) — external IPEX/U.FL antenna lets the antenna be relocated toward the cabinet opening for real signal |
| Firmware | ESPHome, browser-based flashing, `http_request` component for push | No compiler/CLI needed for basic setup; config via YAML |
| TDS sensing | Two analog TDS probes, one at TAP-A, one at TAP-B | Compare raw vs. product TDS = real rejection rate over time, not a guess |
| Pressure sensing | Analog/I2C pressure transducer at TAP-A | Diagnoses clogged prefilters by pressure drop over time; also informs membrane performance question above |
| Leak detection | Standalone battery/plug leak puck, self-contained, no network | See above — deliberately decoupled from the data pipeline |

### Physical effort required (honestly assessed)
- Zero soldering if pre-terminated sensor modules are used
- Jumper wire / screw-terminal connections to the ESP32 board — push-fit or screwdriver-tighten, no specialized tools
- Routing thin probe wires from the board to TAP-A and TAP-B, zip-tied/taped in place
- Everything past physical wiring (board config, sensor calibration, Apps Script deployment, ESPHome YAML) is screen work

---

## Open items / not yet locked

- [ ] Confirm actual feed water pressure (psi) — hose-bib gauge test (now a membrane-performance question, not a leak-safety one — see Source Water Profile note)
- [ ] Iron/manganese test strip check on source water
- [ ] Final Phase 1 parts list with real current models/prices, by category
- [ ] Feed isolation subsystem: confirm specific flow switch, timer relay, and solenoid model numbers; cross-check minimum actuation flow rate against actual dispense-line flow rate once faucet model is chosen
- [ ] Feed isolation subsystem: set the hard backup maximum-duration cutoff value
- [ ] Phase 2: set exact long-interval push duration (leaning 180–240+ min baseline, plus event-triggered push on flow detection)
- [ ] Phase 2: decide whether event-triggered push fires once per activation or repeats through the full open window (current lean: once per activation)
- [ ] Final Phase 2 sensor module picks (specific TDS probe, pressure transducer models compatible with ESPHome)
- [ ] Standalone leak puck product pick
- [ ] Standalone variable-temp kettle model (hot water, fully decoupled from plumbing — not yet researched)
- [ ] **Independent design review pending before SKU/parts selection begins** — full plan to be reviewed for objections/gaps before committing to specific purchases

---

*Document generated as a working reference — update as decisions are finalized.*
