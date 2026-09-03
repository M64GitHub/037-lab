# 0 3 7 — music theory lab for tracker people

A one-file browser app that teaches music theory **in semitones** — no staff notation, no
sheet music, nothing you'd have to translate before typing it into a tracker. Built for
chiptune / tracker musicians, and named after the question that started it:

> Is the chip-arp formula `0 3 7` the right chord on *every* note of the scale?

**No.** It's the minor triad. Stack thirds through the scale and the formula changes with
the degree — that's the whole idea of the app in one table (C major):

| degree | I     | ii    | iii   | IV    | V     | vi    | vii°  |
| ------ | ----- | ----- | ----- | ----- | ----- | ----- | ----- |
| type   | `0 4 7` | `0 3 7` | `0 3 7` | `0 4 7` | `0 4 7` | `0 3 7` | `0 3 6` |

Digits that were bent to stay in scale show up **amber** in the app; anything that leaves the
scale shows **red**. Every chord, every scale, every key.

## Run it

```
open 037-lab.html
```

That's it. No build, no server, no dependencies — works from `file://` and offline (the only
network request is the JetBrains Mono web font, and it degrades gracefully). Sound starts on
your first click; everything is synthesised with pulse / triangle / saw waves in the Web Audio API.

## What's inside

Global controls up top — **root, scale, waveform, arp rate, volume** — drive all five tabs.

**Scale & Chords** — twenty-four scales in four families (the modes; minor and exotic flavours
like harmonic, Hungarian and double harmonic; pentatonic, blues and the Japanese hirajoshi, in-sen,
yo and Ryukyu; the symmetric whole-tone and diminished scales), each with a one-line mood. A
keyboard that lights up the scale, then every chord shape
(triads, sus, power chords, 7ths, 6ths, add9) built on every scale note, shown as the digits you
type in your arp. Toggle *snap to scale* to see the raw formula vs. the diatonic one. Every card
lists its inversions with the note each one puts on top; pick a **Top note** and the inversion
that puts it there lights up in every card, since a fast arp is heard by its highest note.
Under the scale title a row of chips lists the keys that share the same notes (A minor = C major =
D Dorian …, the modes) and the parallel key on the same root; click one to switch. Common
progressions play as fast arps, one bar each, in the current key. At the bottom, the **Finder**
answers the reverse question. Type an arp formula you copied from someone's song (`0 5 9`), note
names (`C E G`) or tracker notes, or tick *Pick on keyboard* and click keys: the left pane names
the chord, the inversion and the root-position formula, lists close chords when nothing matches
exactly, and shows which keys hold that chord and as which degree (*Set key* switches). The right
pane is the scale finder: type all the notes of a song and it ranks every scale that holds them by
fewest unused notes, shows which notes are unused, and *Use* sets root and scale with one click.

The on-screen keyboard also plays from your computer keyboard the way trackers do: `Z`–`/` is one
octave from C-3 (Z S X D C V G B H N J M , L . ; /), `Q`–`P` the octave above, `[` and `]` move
it, while the Scale tab is open and no text field has focus. In browsers with Web MIDI (Chrome,
Edge, Opera) a **MIDI** button in the header connects your MIDI keyboard, velocity and all. Both
feed the Finder's *Pick on keyboard* mode, so play a chord and it gets named.

**Melody Lab** — a melody generator that thinks like a tracker: 16 rows per bar, hex step numbers,
`C-4`-style notes, `|` for a note still ringing, `···` for a rest. Pick length (1–8 bars), tempo,
density, range, rests and leaps; it writes chord tones on the strong steps, walks stepwise in
between, and shapes phrases (A B A B′, longer forms repeat and end on a cadence). Space plays. Click any
cell in the Melody, Bass or Drums column to select it and edit it from the keyboard, the way a
tracker does:

  | key | does |
  | --- | --- |
  | `↑` `↓` | move the note through the scale (bass: through the bass range; drums: cycle kick, snare, hat, open hat) |
  | `Shift ↑↓` · `Alt ↑↓` | an octave · a semitone (a note outside the scale turns **red**) |
  | `R` | reroll the note |
  | `Del` / double-click | mute it to a rest |
  | `Enter` | start a note on a rest, or split a ringing note at that row |
  | `+` `−` | lengthen or shorten the note |
  | `S` | toggle a slide into the note |
  | `←` `→` `Tab` `Esc` | move around the columns, deselect |
  | `Ctrl+Z` `Ctrl+Shift+Z` | undo, redo (buttons too) |

  Hand-edited bass and drums survive a progression change and travel in the share link.

- **Paste from your tracker** — the other direction. Copy a channel in OpenMPT (IT or XM), Schism
  or DUET and press **Paste from tracker** (or the ⇩ button in a column header to aim it at the
  Bass or Drums column). Notes, note-offs, held notes and tone portamento (`Gxx`, `3xx`, `03xx`)
  come back as cells and slides; other effects are counted and ignored; more than one channel in
  the text uses the first. Drums map by instrument number 01–04, or by pitch (C kick, D snare, F#
  hat, A# open hat) when there is none. If the browser will not hand over the clipboard, a text box
  appears to paste into. Your melody then gets chords suggested from its notes, plus a generated
  bass, arp and drums — edit, play and export it back.
- **Suggest chords** — harmonizes whatever is in the Melody column: for every bar it scores the
  chords of the scale by how well they fit the notes (strong steps and long notes weigh more, a
  hand-made bass line counts as the root), then picks a sequence that moves well and lands on the
  tonic. The result becomes the progression text. Click any chord in the **Chord** column to see the
  best alternatives for that bar and swap one in; Ctrl+Z takes it back. Chord arps and a triangle octave bass come along for the ride, each in its own column so you
see exactly what plays under the melody, and each column has **IT** / **SWM** buttons in its
header that copy it to the clipboard in OpenMPT's `ModPlug Tracker  IT` text or DUET's `DUET SW`
format, slides included (`Gxx` / `03xx`), ready to paste into DUET, OpenMPT or Schism. *Copy as
text* still gives you a plain readable dump.

- **Arp column** — the chord channel the way a tracker plays it: one note per bar plus the
  arpeggio command on every row, `J47` for `0 4 7`, `J37` for `0 3 7` — the app's namesake, in the
  clipboard at last. **Voicing** picks the inversion (root, 1st, 2nd, or *auto*, which keeps the
  top note close from bar to bar). The IT export writes `Jxy` on all sixteen rows of each bar and
  tells you the tick rate it runs at; the SWM export writes SID-Wizard's chord select (`07xx`) on
  the note row and lists the chords to enter in the chord table, since the clipboard cannot carry
  them.
- **Progression** — type your own, one chord per bar, up to eight. The presets are just a
  starting point. Changing it re-chords the pattern on screen (bass and arp follow, the melody
  stays), and the Scale tab shows it as *Your progression*, playable like the others.

  | you type | you get |
  | --- | --- |
  | `1 5 6 4` | scale degrees, snapped to the scale: whatever triad the scale gives on that degree |
  | `i VI III VII` · `I V vi IV` | roman numerals mean what they say: upper = major, lower = minor, whatever the scale |
  | `57` · `V7` · `2m7` · `1maj7` · `4sus4` · `7dim` | a suffix forces the shape: `m maj dim aug sus2 sus4 5 7 m7 maj7 m7b5 dim7 6 m6 add9` |
  | `b6 b7` · `#4dim` · `bVI` | `b`/`#` shift the root a semitone — borrowed chords; digits that leave the scale show **red** |
  | `1 - 5 -` | `-` holds the previous chord for another bar |

- **Drums column** — a chip noise kit: kick (triangle pitch drop), snare (noise burst plus a short
  tone), closed and open hat (short and long high-passed noise), all synthesised from a 15-bit
  LFSR like the NES noise channel, no samples. Pick a **Drum style** (four on the floor, boom bap,
  breakbeat, half time, drum & bass, rock); the last bar gets a fill. One drum per row, as on a
  single noise channel. The export writes a fixed note with instrument numbers 01–04, so make a
  kick, snare, hat and open hat instrument in your tracker and paste.

- **Slides** — set *Some* or *Lots* and the generator places tone portamento where it belongs:
  sliding up into the peak of a phrase, down into the cadence, bending across leaps. The amber
  `3xx` column is the value to type; speeds follow SID-Wizard's calculated-slide timing, so the
  number is tempo-aware and realistic. Untick **Hear slides** to A/B the same melody without them.
  Playback is real legato — the running note bends, nothing retriggers.
- **Seed, share, undo** — every Generate has a seed (the *Seed* field). The same seed with the same
  settings gives the same melody, so type one in to reproduce it, or press ↻ to hear the same seed
  under new settings. The address bar always holds a link that restores the key, every setting, the
  seed and the exact pattern, edits included; **Copy link** puts it on the clipboard, and the last
  state comes back when you reopen the page. **Undo / Redo** (Ctrl+Z, Ctrl+Shift+Z) cover Generate,
  Variation, rerolls and muted notes.

**Ear** — ear training in semitones. *Intervals* (easy 3 4 5 7 12, medium, or all twelve; up, down,
together or mixed), *Chords* (triads, plus sus and power chords, plus 7ths; played as a chord or as
an arp), *Scales* (the common six or all 24) and *Degrees*: the tonic chord of the current key, then
one note, and you name the scale step, which is the skill that writes melodies. Answer by click or
from the keyboard (`1`–`9` `0` `-` `=` are 1–12 semitones or degrees, or the n-th chip), `R`
replays, `Enter` asks the next one. A wrong answer plays the right one and then yours. Scores and
streaks per mode stay in the browser.

**Techniques** — the chip tricks that make three channels sound like a band, most with a
*Hear it* button: chord arps, inversions for the top note, fake echo, delayed vibrato, slides,
octave bass, detune, PWM, sidechain pump, one channel doing two jobs, arrangement in 4s and 8s.

**Theory** — semitones as the only unit, scales as a palette, why chords aren't always `0 3 7`,
inversions, progressions, writing melodies that work, rhythm & groove, and a cheat sheet.

## Conventions

Everything is a semitone offset. Notes are MIDI numbers under the hood and tracker names on
screen (`C-4`, `A#3` — sharps only). Scales and chords are arrays like `[0, 3, 7]`. Roman
numerals only appear for 7-note scales; pentatonics and blues just count degrees.

## Hacking on it

It really is one file: `<style>`, the markup, then a `<script>` split by `/* ===== name ===== */`
banners. The interesting bits are data tables:

- `SCALES` — offsets plus a *mood* line; adding a scale is one line.
- `CHORDS` — the raw formula (`iv`) plus which scale steps it stacks (`deg`), which is what makes
  snapping work for any scale length.
- `PROGS` — progressions as 0-based scale degrees.
- `TECH` / `THEORY` — the cards; give one a `d:` key and it gets a *Hear it* button wired to `DEMOS`.

`fitChord()` is the heart of it, `voice()` is the only synth primitive, and all note timing runs
on AudioContext time — never `setTimeout`. Edit, reload, done.
