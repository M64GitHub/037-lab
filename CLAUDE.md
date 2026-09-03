# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Local extension (read first)

If `local/CLAUDE-MD-EXTENSION.md` exists, read it before doing anything else and treat its
instructions as high priority — where it and this file differ, it wins. The `local/` folder is
gitignored and holds local-only working notes, plans and session protocols for developing this
app. If the file does not exist, ignore this section.

## What this is

**0 3 7 — music theory lab for tracker people.** A single-file browser app (`037-lab.html`) that teaches music theory in semitones (never staff notation) for chiptune/tracker musicians, and answers the question in its name: the tracker arp formula `0 3 7` is *not* the chord for every scale degree. Everything — CSS, markup, JS, and all the prose content — lives in that one file.

## Commands

There is no build, lint, or test step. Open `037-lab.html` directly in a browser (`open 037-lab.html`) and reload after edits. The only network dependency is the JetBrains Mono Google Font; everything else works offline via `file://`.

Audio requires a user gesture to start — the first click on anything audible creates the `AudioContext`. When checking audio changes, click a key/button rather than expecting sound on load.

## Layout of the file

`<style>` (lines ~10–233) → markup with a sticky header holding the **global controls** (root, scale, waveform, arp rate, volume) and four `<section class="tab">`s → one `<script>` divided by `/* ===== name ===== */` banners in this order: data, state, audio, tabs, global controls, keyboard, chord chips + degree cards, progressions, melody lab, url state + undo, demos, techniques content, theory content, learn renderer, init.

Add new code inside the matching banner section; the init block at the bottom fixes the render order.

## Architecture

### Everything is a semitone offset
Pitches are MIDI numbers; scales, chords and intervals are arrays of semitone offsets (`iv`). `tname(midi)` formats tracker-style names (`C-4`, `A#3`). `NOTE_NAMES` are sharps only.

### Data tables drive the UI
- `SCALES` — `iv` offsets + a `mood` string shown as the tab hint.
- `CHORDS` — `iv` is the raw tracker formula; `deg` is *which scale steps the chord stacks* (0-based). `deg` is what makes snapping work for any scale length.
- `PROGS` — progressions as 0-based scale degrees (`degs`), assumed 7-note but wrapped with `% L` for pentatonics/blues.
- `TECH` / `THEORY` — `{t: title, b: HTML body, d?: DEMOS key}`; rendered as collapsible cards by `renderLearn`. A `d` key adds a "Hear it" button that calls `DEMOS[d]()`.

### `fitChord()` is the core idea
`fitChord(degIdx, chord, snap)` builds a chord on a scale degree. With `snap=true` it stacks by `chord.deg` through the scale (diatonic result); with `snap=false` it uses the raw `chord.iv`. It returns `rel` (the numbers to type in a tracker), `adj` (digit differs from the raw formula → rendered **amber**), `out` (pitch class not in scale → rendered **red**), `midis`, and `rootNote`. The degree cards, progressions, and melody lab chords all go through it. Progressions and the melody lab always pass `snap=true` with a triad, so they are always diatonic.

`rootMidi()` places the root around C3–B3 but shifts C/C#/D up an octave so chords sit in a comfortable register for all 12 keys. Roman numerals (`romanFor`) only apply to 7-note scales.

### State and rendering
`S` is the single global state object (root, scale, chord, snap, wave, arpRate, vol). Rendering is imperative and rebuilds DOM from scratch: changing root/scale calls `renderScaleTab()` (keyboard, scale strip, chips, degree cards, progressions) plus `fillMelodyProg()`. Changing the chord shape only re-renders chips + degree cards. `$`/`$$` are `querySelector` shorthands.

**Persistence (url state + undo section).** `appState()` collects the key and header controls (`r s w a`), every Melody Lab control (`MCTL`, keyed by element id without the `m`), the seed and the pattern (`pat`, from `encodeCells`: `.` rest, `-` hold, two base-36 chars per note, `_vv` slide) into one object. `writeState()` puts it in the URL hash via `history.replaceState` and in localStorage (`LS_KEY`); `saveState()` is the debounced version and runs at the end of `renderPattern()` and on every control change. Init reads the hash, then localStorage (`readState`), applies the globals before `fillGlobal()` and the melody controls after `fillMelodyProg()`, then restores the pattern (`restorePattern`) or generates from the seed. **Every new control must be added to `MCTL` (or `appState()`)** so links and reloads restore it. Undo/redo (`MEL.hist` / `MEL.redo`) hold JSON snapshots of cells, bass and seed; call `pushUndo()` before any mutation of the pattern. Chords are not part of the snapshot or the link yet; they are recomputed from the current key and progression.

### Audio
`audio()` lazily creates one `AudioContext` and the chain `voice → gain → compressor → lowpass → master → destination`, and builds the three pulse waveforms (`pulse50/25/125`) as `PeriodicWave`s from a Fourier series of the duty cycle. `voice(midi, opts)` is the *only* synth primitive — envelope, optional glide (`glideTo/glideT`), optional delayed vibrato (`vib`), per-voice `wave` override. `playNote`, `playChordSim`, `playArp` wrap it. `playArp` cycles chord tones at `S.arpRate` notes/sec (the chip-arp trick itself). All scheduling uses AudioContext time via the `t` option — never `setTimeout` for note timing. Started oscillators are tracked in `live[]` so `stopAll()` can cut them; demos call `stopAll()` first.

### Melody Lab
State in `MEL`. Generation is seeded: `generateMelody(seed)` reseeds `RNG` (mulberry32 from `makeRng`) and every random choice goes through `rnd`/`chance`, never `Math.random`, so seed + settings reproduces the melody, bass and slides; `MEL.seed` is shown in the Seed field. A pattern is a flat array of 16 cells per bar, each `{type:'note', midi, len}`, `{type:'hold'}` (note still ringing) or `{type:'rest'}`; displayed as hex row numbers like a tracker. Generation per bar: pick a rhythm template from `RHY[density]`, then `walkPitch` chooses pitches — chord tones on strong steps (0/4/8/12), mostly stepwise motion, occasional leaps, from `melodyPool()` (scale notes across the chosen octave range). Bar chords come from the selected progression. Phrase form depends on length: 4 bars = A B A B′, 2 bars = A A′, 1 bar = cadenced; `varyBar` rerolls a fraction of onsets and, with `cadence`, pulls the last note to root or fifth. Click a note cell to reroll it, double-click to toggle rest. `scheduleMelody` schedules one whole loop iteration (lead + optional chord arp on `pulse125` + triangle bass); looping re-schedules via a `setTimeout` fired just before the iteration ends, while a `requestAnimationFrame` tick highlights and auto-scrolls the current row. Space bar toggles play on the Melody tab. `MEL.bass` is a parallel cell array in the same format (the octave-bass figure, built from the bar chords by `genBass`); it is what the **Bass** column shows and what `scheduleMelody` plays — the bass is data, not improvised at schedule time, so it can be exported.

**Clipboard export.** `exportText(cells, 'it'|'swm')` writes a column in the OpenMPT (`ModPlug Tracker  IT`, 11-char cells) or DUET SID-Wizard (`DUET SW`, 9-char cells) clipboard text; the header buttons call `exportColumn(col, fmt)`. Conventions: note names are shifted one octave (`itName`) because IT calls middle C `C-5` while the app shows `C-4`; the first rest row after a note becomes `===` (note-off) in IT only; the stored SID-Wizard `slide` value is written verbatim as `03xx`, and converted for IT with `itSpeed()` assuming speed 6 and tempo = BPM (`Gxx` = xx/16 semitone per tick).

**Slides (tone portamento).** `#mSlide` (Off/Some/Lots) is a *generation* option: `placeSlides()` runs as a post-pass over the finished cells and sets `slide: vv` on note cells (peak of the bar, cadence note, leaps ≥4 st, and for Lots some stepwise moves; never on step 0, after a rest, or on a 16th). `vv` is a SID-Wizard `03vv` speed derived by `slideVal()` from the interval and the note length at the current BPM (`slideSecs()` is the forward formula); it renders as an amber `3xx` fx column. `#mHearSlide` is the *playback* A/B toggle: with it on, `scheduleMelody` merges a note and the following contiguous `slide` notes into one `voice()` call with a `ramps` list (linear-in-Hz frequency ramps, no retrigger); with it off every note is voiced separately.
