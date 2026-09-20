# wled-midi — Bome MIDI Translator pack

**WLED, from a MIDI Translator user's point of view:** it's open-source firmware for addressable
LED strips and bulbs that listens on a **local HTTP JSON API** (no cloud). You never touch that
API here — you just send WLED the **right MIDI**, and a small open convention converts each
message to the WLED JSON it means (note → colour, CC → brightness/effect, Program Change →
preset). This pack makes **[Bome MIDI Translator Pro](https://www.bome.com/products/miditranslator)**
emit that right MIDI from **any** controller, with **no code**.

> **New to WLED? Understand it in 60 seconds** — what it is, its vocabulary, and the whole
> MIDI→WLED map on one page: the **[WLED + MIDI quick reference »](https://github.com/openlamp/openlamp-spec-midi/blob/main/docs/quick-reference.md)**.
> Full spec: **[openlamp/wled-midi »](https://github.com/openlamp/openlamp-spec-midi)**.

A controller sends whatever MIDI its firmware sends. This pack **re-labels** that MIDI into the
[wled-midi](https://github.com/openlamp/openlamp-spec-midi) convention (the right notes / CC / Program
Change) and emits it on a virtual MIDI port your wled-midi implementation listens to — merge,
split and route between hardware, virtual ports and DAWs, then speak wled-midi out the other side.

> ⚠️ **Disclaimer — alpha, not yet qualified.** This pack is an early **alpha**. The translators
> are written to the spec but have **not been properly qualified** — not verified end-to-end on a
> real Bome MIDI Translator install, nor against a physical WLED device. Treat it as a starting
> point, not a finished product: **test with ShowMIDI before wiring any lights**, expect rough
> edges, and please report anything that misbehaves. No warranty (see [LICENSE](LICENSE)).

## What's in the box

| File | What it is |
|---|---|
| [`wled-midi.generic.txt`](wled-midi.generic.txt) | **Generic template** — one translator per wled-midi action (looks, util, modifiers, CC 1–8, Program Change), each with a *capture-me* placeholder on the incoming side. Controller-agnostic: works with anything once you capture your buttons. |

> **Controller-specific presets?** The generic template maps to *any* controller in a few minutes
> via Capture (below), so this pack ships only the generic one. If you build a solid preset for a
> specific controller, a PR to add it here (with the "verify with ShowMIDI" caveat) is welcome.

> **Why text, not a `.bmtp`?** A `.bmtp` project file is a *signed*, INI-based format that breaks if
> hand-edited (Bome's own docs warn against it). So this pack ships translators as **plain text you
> paste into Bome** — the format Bome produces when you copy a translator. It's diff-friendly,
> reviewable, and can't ship you a corrupt project. Build once, then `File → Save As` your own `.bmtp`.

## Setup (5 minutes)

### 1. Create the virtual MIDI port your implementation listens on

wled-midi implementations open a virtual input port (the [engine](https://github.com/openlamp/openlamp-engine-python)
calls it **`OpenLamp`**). Bome sends *to* that port.

- **macOS** — Audio MIDI Setup → MIDI Studio → double-click **IAC Driver** → tick *Device is online* →
  add a port named e.g. `OpenLamp`. (No install; built in.)
- **Windows** — [loopMIDI](https://www.tobias-erichsen.de/software/loopmidi.html) (Tobias Erichsen):
  create a port named `OpenLamp`.

### 2. Import the translators into Bome

1. Open Bome MIDI Translator Pro. `Preset → New Preset`, name it `wled-midi`.
2. Open the pack file (e.g. [`wled-midi.generic.txt`](wled-midi.generic.txt)) in a text editor,
   select a translator block, copy it.
3. In Bome, select the preset and **paste** (`Ctrl/Cmd-V`) — the translator appears. Repeat per block,
   or paste the whole file's blocks at once.
4. Set the project **MIDI routing**: In port = your controller; Out port = `OpenLamp` (the virtual port).

### 3. Capture your controller (generic template only)

Each generic translator has a placeholder incoming message. Click the translator → **Incoming** tab →
**Capture** → press the button/move the control on your hardware → Bome fills in the real MIDI. The
outgoing wled-midi message is already set. Done.

### 4. Verify with ShowMIDI

Route Bome's `OpenLamp` output into [ShowMIDI](https://github.com/gbevin/ShowMIDI) (or watch the engine
log) and press a control — confirm the **right wled-midi note/CC** comes out before wiring the lamps.

## The wled-midi target map (the outgoing side)

Every translator in this pack emits one of these (channel 1 = all lamps; change the channel nibble to
target a group). See the [SPEC](https://github.com/openlamp/openlamp-spec-midi/blob/main/SPEC.md).

| Action | Outgoing MIDI (hex) |
|---|---|
| **Looks** black/red/orange/yellow/green/cyan/blue/magenta/white/effect | `90 3B 7F` … `90 44 7F` (notes 59–68) |
| **Util** off/on/toggle/blackout/restore/solid | `90 30 7F`,`90 32 7F`,`90 34 7F`,`90 35 7F`,`90 37 7F`,`90 38 7F` (48/50/52/53/55/56) |
| **Modifiers** beat / flash | `90 48 7F` / `90 49 7F` (72/73) |
| **CC** bri/cct/hue/sat/fx/sx/ix/pal | `B0 01 pp` … `B0 08 pp` (CC 1–8, value pass-through) |
| **Program Change** → preset n+1 | `C0 pp` |

## Adapt to any controller

- **Pads → looks**: capture each pad on a *Looks* translator.
- **Knobs/faders → CC 1–8**: capture each on a *CC* translator (value passes through as `pp`).
- **A bank/mode button → target channel**: add a translator that sets a Bome global variable, then
  make the outgoing channel nibble a variable — one controller drives multiple lamp groups.
- **Return feedback** (WLED state → controller LEDs): a natural next preset — incoming = the engine's
  `OpenLamp Feedback` port, outgoing = your controller's LED addresses. (Roadmap.)

## Credits

Built on [Bome MIDI Translator Pro](https://www.bome.com/products/miditranslator) (not affiliated).
Part of the [OpenLamp](https://github.com/openlamp) / [wled-midi](https://github.com/openlamp/openlamp-spec-midi)
project. MIT licensed — adapt freely.

---

**Two open standards, one bridge.** This implements the open [**wled-midi**](https://github.com/openlamp/openlamp-spec-midi) convention — the agreed dictionary between [**MIDI**](https://midi.org) (the MIDI Association) and [**WLED**](https://kno.wled.ge). Free for anyone to build on: see the convention's [openness & patent policy](https://github.com/openlamp/openlamp-spec-midi/blob/main/SPEC.md) (§14) and the [licensing note](https://github.com/openlamp/openlamp-spec-midi/blob/main/docs/licensing.md). Part of [OpenLamp](https://github.com/openlamp).
