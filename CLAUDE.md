# CLAUDE.md — openlamp/bome

A **no-code adapter pack** for [Bome MIDI Translator Pro](https://www.bome.com/products/miditranslator):
maps any controller's MIDI onto the [wled-midi](https://github.com/openlamp/openlamp-spec-midi) convention.

## What this repo is (and isn't)

- **Is**: plain-text translator files you paste into Bome, + a README recipe. No runtime, no build.
- **Isn't**: a `.bmtp` binary. Bome's `.bmtp` is a *signed* INI file that corrupts if hand-edited
  (Bome's docs warn against it), so we ship the **copy-text** form of translators instead — reviewable,
  diff-friendly, and impossible to ship corrupt. Users build once in Bome and `Save As` their own `.bmtp`.

## The contract that must stay correct

The **outgoing** side of every translator IS the wled-midi convention and must match
[SPEC.md](https://github.com/openlamp/openlamp-spec-midi/blob/main/SPEC.md) exactly:

- Looks = notes 59–68 (`90 3B 7F` … `90 44 7F`), Util = 48/50/52/53/55/56, Modifiers = 72/73.
- CC 1–8 = bri/cct/hue/sat/fx/sx/ix/pal (`B0 01 pp` … `B0 08 pp`), value pass-through.
- Program Change → preset (`C0 pp`). Channel nibble = target lamp group.

The **incoming** side is controller-specific and meant to be re-captured — never assert a controller's
factory MIDI numbers as fact unless verified (they vary by scene/firmware). Frame them as "documented
default, verify with ShowMIDI".

## Untested

No Bome install in the dev loop here → the paste round-trip and import are **unverified on a machine**.
The GUI-recipe path in the README is the guaranteed fallback. Verify before claiming it imports clean.
