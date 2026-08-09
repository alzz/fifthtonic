# Fifthtonic

Type a chord progression, find out what key it is in.

A single self-contained HTML page. No build step, no dependencies, no network
requests — open `index.html` in a browser and it works.

## Usage

```
git clone git@github.com:alzz/fifthtonic.git
cd fifthtonic
open index.html          # or xdg-open, or just double-click it
```

Type chords separated by spaces or commas:

```
C G Am F
Dm7 G7 Cmaj7
Am F C G7 Bb
```

The answer updates as you type.

## What it tells you

- **The key**, with a confidence percentage and a plain-language reason.
- **Possible keys** — the runners-up, as a row of buttons. Click one and the
  whole page is re-read against it; the detected key stays tagged so you can
  get back.
- **Circle of fifths**, with the key's six chords shaded, their scale degrees
  marked, the diminished degree at its root note, and the parallel key outlined
  in dashes.
- **How each chord functions** — a Roman-numeral analysis of what you typed,
  including secondary dominants (`D7` in C major reads `V7/V`) and borrowed
  chords.
- **Chords in this key** and **harmonic function** — every diatonic triad
  grouped as tonic, subdominant, dominant.
- **Modes** built on each degree, ordered darkest to brightest.

## Capo

Set a capo fret and type the shapes you finger. The app transposes them and
analyses what actually sounds:

```
G Em C D  +  capo 3   →   Bb major   (you are fingering G major shapes)
```

Chord names are respelled for whichever key wins, so the IV of Ab major reads
`Db`, not `C#`. Slash chords transpose too: `G/B` at capo 5 becomes `C/E`.

## Chord syntax

| | |
|---|---|
| Roots | `A`–`G` with `#` `b` `♯` `♭` `x` `##` `bb` |
| Triads | major, `m` `min` `-`, `dim` `°`, `aug` `+`, `sus2` `sus4`, `5` |
| Sevenths | `7`, `maj7` `M7` `Δ`, `m7`, `m7b5` `ø`, `dim7` |
| Extensions | `6`, `6/9`, `9`, `11`, `13`, `add9` … |
| Alterations | `b5` `#5` `b9` `#9` `#11` `b13` |
| Slash | `C/E`, `G/B` |

A token it cannot read gets a red chip and an inline message; the rest of the
progression is still analysed.

## How the key is chosen

All 24 keys are scored — 12 major, and 12 minor scored against the union of the
natural, harmonic and melodic forms, since real minor-key music moves freely
between them.

Scoring is weighted evidence, not a count of matching notes:

- A dominant seventh on the fifth degree is the strongest single signal a key
  can give: it carries both the leading tone and a tritone that resolves one way.
- A tonic chord first, and especially last, counts for something.
- `ii–V–I` motion counts for more.
- Notes outside the key are penalised — but only lightly when the chord is a
  recognisable borrowing: a secondary dominant, `bVII`, `iv` in major, a
  Picardy third.

The scores go through a softmax deliberately warm enough that a genuinely
ambiguous progression reports as ambiguous. `Dm7 G7 Cmaj7` comes out at 94% for
C major; `Am F C G` comes out around 32%, because vi–IV–I–V really is
ambiguous. The app says so rather than manufacturing confidence.

## Development

The music theory lives in a pure module — `parseChord`, `scoreKey`, `analyze`,
`diatonicChords`, `modesOfKey` and friends — with no DOM access. The UI layer
only calls `analyze()` and renders the result.

A self-test runs on page load and reports to the console:

```
Fifthtonic self-test — 18/18
```

To run the same tests headlessly:

```sh
node -e "
const fs=require('fs');
const m=fs.readFileSync('index.html','utf8').match(/<script id=\"theory\">([\s\S]*?)<\/script>/);
const mod={exports:{}}; new Function('module',m[1])(mod);
process.exit(mod.exports.selfTest((l,p,t)=>{l.forEach(x=>console.log(x));console.log(p+'/'+t);})?0:1);
"
```

Add cases to `selfTest()` in the theory block as `['C G Am F', 'C major']`, or
`['G C D G', 'Bb major', 3]` to test with a capo.

## License

MIT — see [LICENSE](LICENSE).
