# 0 3 7 — music theory lab for tracker people

A one-file browser app that teaches music theory **in semitones** — no staff notation, nothing
you'd have to translate before typing it into a tracker. Built for chiptune / tracker musicians,
and named after the question that started it:

> Is the chip-arp formula `0 3 7` the right chord on *every* note of the scale?

**No.** It's the minor triad. Stack thirds through the scale and the formula changes with the
degree — that's the whole idea in one table (C major):

| degree | I       | ii      | iii     | IV      | V       | vi      | vii°    |
| ------ | ------- | ------- | ------- | ------- | ------- | ------- | ------- |
| type   | `0 4 7` | `0 3 7` | `0 3 7` | `0 4 7` | `0 4 7` | `0 3 7` | `0 3 6` |

Digits that were bent to stay in scale show **amber** in the app; anything that leaves the scale
shows **red**. Every chord, every scale, every key.

## Run it

    open 037-lab.html

No build, no server, no dependencies — works from `file://` and offline (the only network request
is the JetBrains Mono web font, and it degrades gracefully). Sound starts on your first click;
everything is synthesised in the Web Audio API: pulse, triangle and saw waves plus a shift-register
noise for the drums. Chrome, Firefox and Safari work; Web MIDI needs Chrome, Edge or Opera.

## The five tabs

The header holds the **root, scale, waveform, arp rate and volume** for everything below, a
**MIDI** button where the browser supports it, and **Reset**. The address bar always holds a link
that restores the key, every setting, the seed and the exact pattern (**Copy link** puts it on the
clipboard), and the last state comes back when you reopen the page. **Reset** forgets all of that
and reloads with the defaults (A minor, a fresh pattern); the Ear tab's scores are kept.

### Scale & Chords
- **24 scales** in four families (the modes; harmonic, melodic, Hungarian, double harmonic and
  other minors; pentatonic, blues, hirajoshi, in-sen, yo, Ryukyu; whole tone and both diminished),
  each with a one-line mood. A row of chips lists the keys that share the same notes (A minor =
  C major = D Dorian …) and the parallel key; click one to switch.
- **A keyboard** that lights the scale. Play it with the mouse, with the computer keyboard the way
  trackers do (`Z`–`/` one octave from C-3, `Q`–`P` the octave above, `[` `]` move it), or from a
  MIDI keyboard after pressing **MIDI**.
- **Every chord shape** (triads, sus, power, 7ths, 6ths, add9) on every scale note, as the digits
  you type into the arp. *Snap to scale* shows the raw formula vs the diatonic one. Every card lists
  its inversions; pick a **Top note** and the inversion that puts it on top lights up in every card,
  since a fast arp is heard by its highest note.
- **Common progressions** as fast arps, one bar each, plus *Your progression* from the Melody Lab.
- **Finder** — the reverse question. Type a formula you copied from someone's song (`0 5 9`), note
  names (`C E G`) or tracker notes (`C-4 E-4 G-4`), or tick *Pick on keyboard* and play the notes:
  the left pane names the chord, its inversion and root-position formula, lists close chords when
  nothing matches, and shows which keys hold it and as which degree. The right pane is the scale
  finder: type all the notes of a song and it ranks every scale that holds them by fewest unused
  notes, names the unused ones, and **Use** sets root and scale.

### Melody Lab
A melody generator that thinks like a tracker: 16 rows per bar, hex row numbers, `C-4` notes, `|`
for a note still ringing, `···` for a rest. Pick length (1–8 bars), tempo, density, range, rests
and leaps; it writes chord tones on the strong rows, walks stepwise between them and shapes phrases
(A B A B′; longer forms repeat and end on a cadence). Every Generate has a **seed**: same seed,
same settings, same melody. **Undo / Redo** cover everything. Space plays.

- **Columns**: Melody, **Arp** (the chord channel: one note per bar plus `Jxy`, the tracker
  arpeggio command with the chord's offsets; *Voicing* picks the inversion, *auto* keeps the top
  note close from bar to bar), **Bass** (a triangle octave bass), **Drums** (a chip noise kit —
  kick, snare, hat, open hat from a 15-bit shift register; six *Drum styles*, a fill in the last
  bar) and **Chord** (the chord per bar; click it for alternatives).
- **Progression** — type your own, one chord per bar, up to eight; changing it re-chords the pattern
  under the existing melody:

  | you type | you get |
  | --- | --- |
  | `1 5 6 4` | scale degrees, snapped: whatever triad the scale gives on that degree |
  | `i VI III VII` · `I V vi IV` | roman numerals mean what they say: upper = major, lower = minor |
  | `57` · `V7` · `2m7` · `1maj7` · `4sus4` · `7dim` | a suffix forces the shape: `m maj dim aug sus2 sus4 5 7 m7 maj7 m7b5 dim7 6 m6 add9` |
  | `b6 b7` · `#4dim` · `bVI` | `b`/`#` shift the root a semitone — borrowed chords; digits that leave the scale show **red** |
  | `1 - 5 -` | `-` holds the previous chord for another bar |

- **Slides** — *Some* or *Lots* places tone portamento where it belongs (up into the peak, down into
  the cadence, across leaps). The amber `3xx` column is the value to type, timed like SID-Wizard's
  calculated slides; **Hear slides** A/Bs the same melody without them.
- **Edit like a tracker** — click a cell in the Melody, Bass or Drums column, then:

  | key | does |
  | --- | --- |
  | `↑` `↓` | move the note through the scale (bass: the bass range; drums: kick → snare → hat → open hat) |
  | `Shift ↑↓` · `Alt ↑↓` | an octave · a semitone (outside the scale turns **red**) |
  | `R` · `Del` · `Enter` | reroll · mute to a rest (double-click too) · start a note, or split a ringing one |
  | `+` `−` · `S` | lengthen / shorten · toggle a slide into the note |
  | `←` `→` `Tab` `Esc` · `Ctrl+Z` `Ctrl+Shift+Z` | move around, deselect · undo, redo |

  Hand-edited bass and drums survive a progression change and travel in the share link.
- **Export** — every column header has **IT** and **SWM** buttons that copy the column to the
  clipboard in OpenMPT's `ModPlug Tracker  IT` text (11-character cells, `===` note-off, `Gxx`
  slides, `Jxy` arps) or DUET's `DUET SW` text (9-character cells, `03xx` slides, `07xx` chord
  select with the chord table listed for you). Drums export as a fixed note with instruments
  01–04. *Copy as text* gives a plain readable dump.
- **Paste from your tracker** — the other direction. Copy a channel in OpenMPT (IT or XM), Schism
  or DUET and press **Paste from tracker** (or the ⇩ button in the Bass or Drums header). Notes,
  note-offs, held notes and tone portamento come back as cells and slides; other effects are
  counted and ignored. If the browser will not hand over the clipboard, a text box appears.
- **Suggest chords** — harmonizes whatever is in the Melody column: scores every chord of the scale
  per bar by how well it fits the notes (strong rows and long notes weigh more, a hand-made bass
  counts as the root), then picks the sequence that moves well and lands on the tonic. The result
  becomes the progression text; the **Chord** column offers alternatives per bar. Pasted melodies
  get this automatically.

### Ear
Ear training in semitones. **Intervals** (easy `3 4 5 7 12`, medium, or all twelve; up, down,
together or mixed), **Chords** (triads; plus sus and power; plus 7ths; as a chord or an arp),
**Scales** (the common six or all 24) and **Degrees**: the tonic chord of the current key, then one
note — name the scale step, the skill that writes melodies. Answer by click or keyboard (`1`–`9`
`0` `-` `=` are 1–12 semitones or degrees, or the n-th chip), `R` replays, `Enter` asks the next
one. A wrong answer plays the right one and then yours. Scores per mode stay in the browser.

### Techniques
The chip tricks that make three channels sound like a band, most with a **Hear it** button: chord
arps and where the arp rate comes from (ticks, tempo, multispeed), inversions for the top note,
fake echo, delayed vibrato, slides, hard restart, octave bass, noise drums, detune, PWM, sidechain
pump, one channel doing two jobs, secondary dominants, borrowed chords, the semitone-up key
change, instrument design, arrangement in 4s and 8s.

### Theory
Semitones as the only unit, scales as a palette, the circle of fifths as counting by 7, why chords
aren't always `0 3 7`, inversions, progressions, tension and release, writing melodies, harmonizing
a melody, rhythm & groove, and a **cheat sheet** with every formula, the progression syntax, the
exported tracker commands and every keyboard shortcut.

## Conventions

Everything is a semitone offset. Notes are MIDI numbers under the hood and tracker names on screen
(`C-4`, `A#3` — sharps only; the IT export calls the same note `C-5`, as OpenMPT does). Scales and
chords are arrays like `[0, 3, 7]`. Roman numerals only appear for 7-note scales. Colours: cyan =
action, amber = adjusted to fit the scale, red = outside the scale, faint = holds and rests.

## Hacking on it

It really is one file: `<style>`, the markup, then a `<script>` split by `/* ===== name ===== */`
banners. The interesting bits are data tables — `SCALES` (offsets, family, mood), `CHORDS` (the raw
formula plus which scale steps it stacks, which is what makes snapping work for any scale length),
`PROGS`, `DRUMS` and `DRUM_PATS`, `COLS` (the pattern columns), `EAR_MODES`, and `TECH` / `THEORY`
(the cards; a `d:` key wires a *Hear it* button to `DEMOS`). `fitChord()` is the heart of it;
`voice()`, `noise()` and `kick()` are the synth primitives, and all note timing runs on AudioContext
time — never `setTimeout`. `CLAUDE.md` maps every subsystem. Edit, reload, done.
