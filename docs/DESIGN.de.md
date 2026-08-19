# Paula

Ein Music Tracker nach dem Vorbild von OctaMED Professional V5/V6, in Lyx auf
der Vega-VCL. Der Name kommt vom Soundchip des Amiga.

Ein **Music Tracker in Lyx**, der unter **Vega** (LyxOS) läuft und sich visuell wie
funktional an **OctaMED Professional V5/V6** auf dem Commodore Amiga orientiert.

Das Ziel ist nicht „ein moderner Tracker mit Retro-Skin", sondern der Eindruck einer
**verschollenen OctaMED-Version für einen Amiga-Nachfolger**. Bedienung, Tracker-Raster,
Block-/Song-Struktur, Instrumentverwaltung, Effect Commands, MIDI-Konzept und der
Achtkanal-Modus sollen sich möglichst authentisch anfühlen — inklusive der
Unbequemlichkeiten, die zum Instrument gehören.

---

## 1. Referenzpunkt: OctaMED Professional V5/V6

**Designentscheidung (fix): V5/V6, nicht SoundStudio.**

| | OctaMED Pro V5/V6 | OctaMED SoundStudio |
|---|---|---|
| Charakter | klassischer Amiga-Tracker | schon deutlich DAW-artig |
| Kanäle | 4 Paula / 8 Channel Mode | Mixing-Modus mit vielen Kanälen |
| Format | MMD1 / MMD2 | MMD3 |
| Komfort | genug (MIDI, Sections, Playing Sequence) | zu viel für das Projektziel |

V5/V6 wirkt noch wie ein Tracker, hat aber schon MIDI, komplexere Songstrukturen und
den 8-Kanal-Modus. SoundStudio-Features (Mix-Mode-Commands `20`–`2F`, Echo,
Stereo-Separation) gelten als **optionale Ausbaustufe**, nicht als Basis.

### Terminologie — nicht modernisieren

Verbindlich im Code, in der UI und in der Doku:

```
Song        Block       Track       Line
Instrument  Sample      Command     Sequence
```

**Verboten:** Clip, Scene, Lane, Device, Pattern (statt Block), Step (außer als
Cursor-Advance-Wert), Channel Strip, Piano Roll, Automation, Plugin.
Das ist keine Kosmetik — die Begriffe sind Bestandteil des Charakters.

---

## 2. Track-Format

Kompakt wie im Original, **keine** fünf modernen Spalten:

```
NOTE INST COMMAND
C-3   01    000
```

Erweiterter Modus (4-stelliger Command-Teil, MMD1/MMD2):

```
C-3 01 0C40      <- Command 0C (Set Volume), Level 40
```

Feld-Semantik:

| Feld | Breite | Wertebereich | Bedeutung |
|---|---|---|---|
| Note | 3 | `---`, `C-1` … `B-6` | Notenname + Oktave |
| Instrument | 2 | `00`–`3F` (hex) | `00` = kein Instrument-Wechsel |
| Command | 2 | `00`–`2F` (hex) | Effect Command |
| Level | 2 | `00`–`FF` (hex) | Command-Datenbyte |

Anzeige der Notenspalte immer 3 Zeichen (`C-3`, `C#3`), leere Line `--- 00 000`.
Volumes hexadezimal oder dezimal, umschaltbar wie im Original (`FLAG_VOLHEX`).

---

## 3. Hauptbildschirm

Vier Bereiche, dicht gepackt, keine 12-px-Padding-Philosophie:

```
┌─────────────────────────────────────────────┐
│                  MENUBAR                    │
├─────────────────────────────────────────────┤
│ SONG / BLOCK / TEMPO / INSTRUMENT CONTROL   │
├─────────────────────────────────────────────┤
│                                             │
│                BLOCK EDITOR                 │
│                                             │
├───────────────────────┬─────────────────────┤
│ INSTRUMENT / SAMPLE   │ SONG / BLOCK LIST   │
└───────────────────────┴─────────────────────┘
```

Konkret:

```
┌──────────────────────────────────────────────────────────────────┐
│ Project  Block  Edit  Instrument  MIDI  Settings                 │
├──────────────────────────────────────────────────────────────────┤
│ SONG: Demo Song         BLOCK: 00       TEMPO: 125   LPB: 4      │
├──────────────────────────────────────────────────────────────────┤
│ 00 │ C-3 01 000 │ --- 00 000 │ G-2 03 000 │ --- 00 000          │
│ 01 │ --- 00 000 │ --- 00 000 │ --- 00 000 │ --- 00 000          │
│ 02 │ --- 00 000 │ C-4 02 000 │ --- 00 000 │ --- 00 000          │
│ 03 │ --- 00 000 │ --- 00 000 │ --- 00 000 │ --- 00 000          │
│ 04 │ E-3 01 037 │ --- 00 000 │ D-3 03 000 │ --- 00 000          │
│ 05 │ --- 00 000 │ --- 00 000 │ --- 00 000 │ --- 00 000          │
├──────────────────────────────────────────────────────────────────┤
│ Instr: 01 Bass        Octave: 3      Step: 1                     │
├──────────────────────────────────────────────────────────────────┤
│ [Play Song] [Play Block] [Stop] [Rec]                            │
└──────────────────────────────────────────────────────────────────┘
```

Line-Nummern links (hex oder dezimal), Beat-Lines (jede LPB-te Line) leicht abgesetzt,
Play-Position invers hervorgehoben.

---

## 4. GUI-Doktrin: Amiga, nicht Ableton

**Ausdrücklich nicht erwünscht:** dunkle DAW-Panels, Flat-Design-Toolbars, riesige
Waveform-Flächen, abgerundete Ecken, Schatten, Hover-Animationen, proportionale Fonts,
Icon-Only-Buttons.

**Erwünscht:** Amiga-Workbench-/GadTools-Anmutung — harte 1–2-Pixel-Kanten, plastische
Buttons (hell oben/links, dunkel unten/rechts), keine Rundungen, dichte Packung.

### Theme `TPaulaTheme`

```
Background       = AmigaGrey
Panel            = LightGrey
BorderLight      = White
BorderDark       = DarkGrey
Text             = Black
SelectedRow      = Blue
SelectedText     = White
ActiveCell       = Black/White inverted
```

Als Vega-Theme-Part-Baum (Belastungstest für den neuen Theme-Apparat):

```
window.amiga
window.amiga.titlebar
window.amiga.closebutton
window.amiga.depthbutton
window.amiga.resizehandle
```

Nichts hartcodiert — jedes Widget liest seine Farben aus dem Theme
(Vertrag aus WP33.10 „StyleBook-Modell").

### Font

Ein moderner proportionaler Font würde alles ruinieren. Zwingend Bitmap/Monospace:

```
TTrackerGrid.Font      = fixed-width (VGA 8x16 aus vgfx)
TTrackerGrid.RowHeight = font height + 1
```

Der in Vega bereits vorhandene VGA-8×16-Bitmapfont ist genau richtig: feste
Zellbreite 8 px → Spaltenraster ohne Textmessung, `RowHeight = 17`.

### Fensterliste

Eigene, klassisch dekorierte Amiga-Fenster (Titelbalken, Close-Gadget links,
Depth-/Zoom-Gadget rechts):

```
Main Editor           Instrument Properties     Sample Editor
Song Sequence         MIDI Setup                Preferences
Block Properties      About
```

Keine „Inspector Panels" — echte kleine Dialogfenster.

---

## 5. Instrumentverwaltung

Schlichte Liste, Nummern zweistellig hex:

```
01 Bass
02 Snare
03 Closed Hat
04 Open Hat
05 Pad
06 Lead
```

Instrumentfenster (Felder 1:1 aus der MMD-Instrumentstruktur):

```
Instrument 01

Name: Bass

Volume:      64          <- svol, 0..64
Transpose:   0           <- strans
Fine Tune:   0           <- -8..+7

Repeat Start: 0000       <- rep   (Wortoffset, >>1 wie Protracker)
Repeat Len:   1840       <- replen

MIDI Channel: --         <- midich, 0 = kein MIDI
MIDI Preset:  --         <- midipreset
```

Zusätzlich (V5-Instrumentparameter): Hold, Decay, Loop on/off, 8-Bit/IFF-Typ.

---

## 6. Song-/Block-Struktur

- **Song** = Instrumentsatz (max. 63) + Blockliste + **Playing Sequence** + Tempo/Flags.
- **Block** = eigenständiges Raster mit *eigener* Track-Zahl und *eigener* Line-Zahl
  (nicht global wie bei ProTracker). Default 64 Lines, 4 Tracks.
- **Sequence** = Playing Sequence, geordnete Liste von Block-Nummern (max. 256 Einträge
  in MMD0/MMD1). Ein Block darf mehrfach vorkommen.
- **Sections** (V5): mehrere Playing Sequences, Sektionsliste darüber.
- Tempo: `deftempo` (Slider bzw. BPM bei `FLAG2_BPM`) + `tempo2` = **TPL**
  (Ticks Per Line). LPB = Beat-Länge in Lines (`FLAG2_BMASK`, 1..32).

---

## 7. Effect Commands

Basis-Set (V5, „Normal Commands"), Level = Datenbyte:

| Cmd | Bedeutung |
|---|---|
| `00` | Arpeggio (Level 1/2 = Halbtöne über Grundton) |
| `01` / `02` | Slide Pitch Up / Down |
| `03` | Portamento (Zielnote wird nicht neu angeschlagen) |
| `04` | Vibrato (Level 1 = Speed, Level 2 = Depth) |
| `05` | Slide Pitch and Fade (PT-kompatibel) |
| `06` | Vibrato and Fade (PT-kompatibel) |
| `07` | Tremolo |
| `08` | Hold and Decay (Level 1 = Decay, Level 2 = Hold) |
| `09` | TPL Slider setzen (`01`–`20`) |
| `0A` | Volume Slide (nur Tracker-Kompatibilität, `0D` benutzen) |
| `0B` | Playing Sequence Position Jump |
| `0C` | Set Volume (0–64; `80`–`C0` setzt Default-Instrumentvolume) |
| `0D` | Volume Slide (Level 1 = up, Level 2 = down) |
| `0E` | Synth Jump (nur Synth-/Hybrid-Instrumente) |
| `0F` | Primary Tempo / Miscellaneous — `00` = nächster Sequence-Eintrag, `01`–`F0` = Tempo, `FF1`–`FFF` = Spezialfunktionen (`FF1` doppelt anschlagen, `FF2` halbe Line Delay, `FF3` dreifach, `FF4`/`FF5` Triolen-Delay) |
| `11` / `12` | Slide Pitch Up / Down Once (PT `E1`/`E2`) |
| `14` | ProTracker-Style Vibrato |
| `15` | Set Finetune |
| `16` | Repeat Lines / Loop (PT `E6`) |
| `17` | Change Volume Controller |
| `18` | Cut Note (PT `E8`) |
| `19` | Sample Start Offset (PT `9`) |
| `1A` / `1B` | Slide Volume Up / Down Once (PT `EA`/`EB`) |
| `1C` | Change MIDI Preset |
| `1D` | Jump to Next Playing Sequence Entry (PT `D`) |
| `1E` | Replay Line (PT `EE`) |
| `1F` | Note Delay and Retrigger (PT `EC`/`ED`) |

MIDI-Tracks interpretieren einen Teil der Command-Nummern anders (`01`/`02`
Pitchbender, `03`/`13` Set Pitchbender, `04` Modulation Wheel, `05`+`00`
Controller Number/Value, `08` Set Hold Only, `0A` Polyphonic Aftertouch,
`0D` Channel Aftertouch, `0E` Pan, `10` Send MIDI Message, `31`–`3F` Set MIDI
Controller). → Dispatch abhängig vom Track-Typ.

Mix-Mode-Commands `20` (Reverse Sample / Relative Offset), `21`/`22` (Fixed-Rate
Slide), `2E` (Set Track Panning), `2F` (Echo Depth / Stereo Separation) gehören zum
Software-Mixer-Backend und sind **Ausbaustufe**, kein V5-Kern.

---

## 8. Playback-Modi — historisches Verhalten *nicht* wegabstrahieren

Der charakteristische Klang gehört zur Nostalgie. Deshalb echte Betriebsmodi:

```
Playback Mode:

[ ] 4 Channel Paula
[ ] 8 Channel OctaMED
[ ] Modern Mixer
```

**4 Channel Paula (Original Mode)**
- 4 Kanäle, 8-Bit-Samples
- Paula-artiges Panning (Kanal 0/3 links, 1/2 rechts)
- historische Frequenzgrenzen (Period-Bereich, ~28 kHz Obergrenze)

**8 Channel OctaMED Mode**
- 8 virtuelle Kanäle
- 2 Voices pro Paula-Kanal gemischt (Software-Mix in einen Hardware-Kanal)
- reduzierter effektiver Dynamikbereich (der typische leisere, rauere Klang)
- historisches Stereo-Layout
- entspricht `FLAG_8CHANNEL` (0x40) im Song-Header

**Modern Mixer (optional, später)**
- 16 / 32 / 64 Software-Kanäle, Panning, Echo, 16-Bit-Ausgabe

Dieselbe `.med`-Datei muss in allen Modi hörbar unterschiedlich klingen.

### Architektur

Sauber getrennt, damit das historische Verhalten Implementierungsdetail *eines*
Backends bleibt:

```
TPlaybackBackend
    |
    +-- TPaula4ChannelBackend
    |
    +-- TOctaMed8ChannelBackend
    |
    +-- TSoftwareMixerBackend
```

`TPlaybackBackend` ist abstrakt: `Init(rate)`, `NoteOn(ch, inst, period, vol)`,
`SetVolume`, `SetPeriod`, `NoteOff`, `RenderTick(buf, frames)`.
Darüber sitzt `TReplayEngine` (Tick-/Line-/Block-/Sequence-Logik + Command-Dispatch),
die *kein* Backend-Wissen hat. Ausgabe über den vorhandenen **AC97-Treiber**
(`ac97.lyx`, Codec STAC9700) — Signed-16-Bit-Stereo-Mix, intern 8-Bit-Sampledaten.

---

## 9. Workflow: bewusst unbequem

Keine Piano Roll. Keine Drag-and-Drop-Clip-Timeline. Der Benutzer tippt Musik in
Tabellenzeilen:

```
00 C-3 01 000
01 --- 00 000
02 --- 00 000
03 --- 00 000
04 C-3 01 000
05 --- 00 000
06 --- 00 000
07 --- 00 000
08 G-3 01 000
```

Das ist kein UI-Detail. Das ist das Instrument.

### Tastatureingabe (klassisch)

- **Notenzeilen**: zwei Reihen wie beim Piano-Layout — untere Reihe
  `Z S X D C V G B H N J M` = C bis B der Basisoktave, obere Reihe
  `Q 2 W 3 E R 5 T 6 Y 7 U` = eine Oktave höher.
- **Oktave**: Zifferntasten / `-` `+` setzen die Eingabeoktave (1–6).
- **Cursor**: Pfeiltasten, `Tab` = nächster Track, `Shift-Tab` = voriger.
- **Step**: Cursor-Advance nach Eingabe (0 = stehen bleiben).
- **Hex-Eingabe** direkt in Instrument-/Command-/Level-Spalten (`0`–`9`, `A`–`F`).
- **Space** = Edit-Modus an/aus, `Del`/`Backspace` löscht Line-Inhalt,
  `Return` = Line einfügen.
- **Block-Operationen** über Menü *Block*: Copy/Paste/Cut Range, Split, Join,
  Change Tracks/Lines, Transpose.

---

## 10. Zielplattform & Constraints

- **Sprache**: Lyx, Compiler `lyxc` (`/usr/local/bin/lyxc`),
  Includes aus `/home/andreas/PhpstormProjects/aurum` (`ebnf.md` = maßgebliche Grammatik).
- **OS**: LyxOS (`/home/andreas/PhpstormProjects/lyx-os`), Fenster über
  Vega-Compositor (Window-Syscalls 110–152, PID-besessene Top-Level-Fenster).
- **Toolkit**: `vgfx` (TCanvas: FillRect/DrawRect/DrawLine/Text/Blit/Clip) +
  `vui` (TControl-Basis mit echter Vererbung, TForm-Message-Loop, Hit-Test,
  Fokus-Kette, Popup-Layer, Align/Anchors).

Bekannte Toolkit-/Compiler-Grenzen, die das Design mitbestimmen:

| Constraint | Konsequenz für lyx-octaMED |
|---|---|
| Ein flacher Framebuffer pro Fenster (1280×720, fixer Stride) | Alle Widgets malen in eine Fläche; Painter's Algorithm + Clipping |
| Render-Fläche = Compositor `INIT_DISP_W/H` (600×520) | Layout auf 600×520 auslegen oder `INIT_DISP_*` erhöhen |
| Methoden max. 5 Argumente + `self` | Setter statt breiter Konstruktoren (`SetupGrid`, `SetColors`) |
| Geerbte nicht-virtuelle Methoden nur über base-typisierte Referenz | `var base: TControl := self; base.AddChild(..)` |
| Keine Arrays von Klassentypen | Zellen/Instrumente als mmap-Blöcke + `peek64`/`poke64` |
| Reservierte Namen: `form`, `Set` | `frm`, `SetText`/`SetColors` |
| `TStringGrid`/`TDrawGrid` (WP33.8) noch offen | `TTrackerGrid` wird eigenes `TControl`-Derivat, kein Grid-Reuse |
| AC97 nur per Ohr testbar, VM braucht `--audio-controller ac97` | Playback-Verifikation manuell im GUI-Test |

---

## 11. Klassenskizze

```
TOctaMEDForm        (TForm)      Hauptfenster, Menubar, Message-Loop
  TSongInfoBar      (TControl)   SONG / BLOCK / TEMPO / LPB / Instr / Octave / Step
  TTrackerGrid      (TControl)   Block Editor: Lines x Tracks, Cursor, Selection
  TInstrumentList   (TControl)   01..3F mit Namen
  TSequenceList     (TControl)   Playing Sequence / Blockliste
  TTransportBar     (TControl)   Play Song / Play Block / Stop / Rec

TSong               numblocks, songlen, sequence[256], deftempo, tempo2, flags, flags2,
                    trkvol[16], mastervol, instruments[63]
TBlock              numtracks, lines, notes (mmap: line*tracks*4 Byte)
TInstrument         name, svol, strans, finetune, rep, replen, hold, decay,
                    midich, midipreset, sampledata, samplelen
TReplayEngine       Tick-Counter, Line-Advance, Command-Dispatch, Sequence-Position
TPlaybackBackend    abstrakt -> Paula4 / OctaMed8 / SoftwareMixer
TMMDReader          MMD0/MMD1/MMD2 Import
```

Datenmodell nach MMD-Vorbild (MMD1: Note = 4 Byte
`nnnnnnn / iiiiii / cccccccc / dddddddd`; MMD0: 3 Byte mit 6-Bit-Note und
Instrument-Highbits im Notenbyte).

---

## 12. Entwicklungsziele (Authentizität zuerst)

| Phase | Inhalt | Deliverable |
|---|---|---|
| 1 | OctaMED-artiges Hauptfenster | Menubar + Infozeile + 4 Bereiche + `TPaulaTheme`, Bitmapfont, harte Kanten |
| 2 | 4-Track Block Editor | `TTrackerGrid` mit 64 Lines × 4 Tracks, Cursor, Scroll, Beat-Lines |
| 3 | Klassische Tracker-Tastatureingabe | Notenreihen, Oktave, Step, Hex-Felder, Edit-Modus |
| 4 | Instrument + 8-Bit Sample Playback | IFF-8SVX/RAW laden, `TInstrument`, Einzelton über AC97 |
| 5 | Paula-artiger 4-Channel Mixer | `TPaula4ChannelBackend`, Period-Tabelle, Panning, Frequenzgrenzen |
| 6 | OctaMED 8-Channel Mode | `TOctaMed8ChannelBackend`, 2 Voices je Hardware-Kanal, reduzierte Dynamik |
| 7 | Klassische Effect Commands | `TReplayEngine`-Dispatch `00`–`1F` inkl. `0F`-Spezialfälle |
| 8 | Song Sequence | Playing Sequence, Sections, Block-Properties-Dialog, Play Song |
| 9 | MIDI | MIDI-Tracks, Channel/Preset je Instrument, MIDI-Command-Dispatch, MIDI Setup |
| 10 | MMD Import | `TMMDReader` für MMD0/MMD1/MMD2, Flags, Sample-Array, expdata |

Zwischen den Phasen jeweils QEMU-Screendump-Verifikation (headless), Playback nur im
echten GUI-/Audio-Test prüfbar.

---

## 13. Nicht-Ziele

- Kein Modern-DAW-Look, keine Piano Roll, keine Clip-Timeline.
- Keine modernisierten Begriffe.
- Keine VST-/Plugin-Architektur, kein Audio-Effekt-Rack.
- Keine Abstraktion, die den 4-Kanal-Paula-Klang „aufhübscht".
- Kein Export in moderne Formate vor Phase 10.

---

## 13a. Bedienung Blockeditor (Phase 2/3)

| Taste | Wirkung |
|---|---|
| Pfeil hoch/runter | Cursor eine Line, laeuft am Blockende um |
| Pfeil links/rechts | Cursorfeld innerhalb der Zelle, ueber die Trackgrenze hinweg |
| Strg + links/rechts | einen Track weiter (in OctaMED lag das auf Tab, siehe unten) |
| Bild hoch/runter | LPB * 4 Lines |
| Pos1 / Ende | erste bzw. letzte Line, Strg+Pos1 zusaetzlich Track 0, Feld 0 |
| Leertaste | Editiermodus umschalten (Cursor orange statt dunkelblau) |
| Mausrad | Ansicht scrollen, ohne den Cursor mitzunehmen |
| Klick | Cursor auf das getroffene Feld |

Cursorfelder je Zelle: Note, Instrument-High, Instrument-Low, Command
(bei `ExtCmd` zweistellig), Level-High, Level-Low.

### Eingabe (Phase 3)

Geschrieben wird nur im Editiermodus (Leertaste).

| Taste | Wirkung |
|---|---|
| `Z S X D C V G B H N J M` | Note C..B der Eingabeoktave (nur im Notenfeld) |
| `Q 2 W 3 E R 5 T 6 Y 7 U` | dieselben Noten eine Oktave hoeher |
| `0`–`9`, `A`–`F` | Hexziffer in Instrument-, Command- oder Levelfeld |
| `-` / `+` | Eingabeoktave 1..6 (wirkt auch ausserhalb des Editiermodus) |
| `Entf` | Zelle loeschen, danach Step-Vorschub |
| `Rueckschritt` | Zelle loeschen, eine Line zurueck |
| `Return` | Line einfuegen, alles darunter rutscht nach unten |
| `Shift+Return` | Line loeschen, alles darunter rueckt hoch |

Nach jeder Eingabe rueckt der Cursor um `Step` Lines vor; `Step: 0` laesst ihn
stehen, damit Akkorde uebereinander getippt werden koennen. Eine neu gesetzte
Note bekommt das Instrument aus der EditBar, sofern dieses nicht `00` ist —
`00` heisst im Zellformat "kein Instrumentwechsel".

**Abweichung von Abschnitt 9**: die Zifferntasten liegen auf den Kreuznoten der
oberen Reihe (`2 3 5 6 7`) und nicht auf der Oktavwahl — sonst fehlte der
oberen Notenreihe die Haelfte ihrer Halbtoene. Die Oktave liegt deshalb auf
`-` und `+`. Ein Tastenkuerzel fuer `Step` gibt es noch nicht.

**Tab ist nicht verfuegbar**: `TDispatcher.HandleKeyDown` faengt `KEY_TAB` fuer
die Fokusverwaltung ab, bevor ein Control es sieht. Deshalb Strg+Pfeil fuer den
Trackwechsel.

---

## 13b. Instrumente & Vorhoeren (Phase 4)

- `OMED/Sample.lyx` laedt **IFF-8SVX** (`FORM`/`VHDR`/`BODY`, big endian,
  Fibonacci-Delta wird abgelehnt) und **RAW** (kopflos, jedes Byte ein Sample).
  Das Format wird am Inhalt erkannt, nicht an der Endung. `rep`/`replen`
  landen in Worten im `TInstrument`, so wie in MMD und in den Paula-Registern.
  `MakeSquareSample` erzeugt einen Ersatzklang, damit der Editor auch ohne
  geladene Datei hoerbar ist.
- `OMED/Playback.lyx` rechnet in **Amiga-Perioden** (PAL-Takt 3546895,
  Periodentabelle Oktave 1, Halbierung je Oktave, Grenzen 113..3576) und
  enthaelt `TVoice` (16.16-Fixpunkt), das abstrakte `TPlaybackBackend` und
  `TSoftwareMixerBackend` (8 Stimmen, signed 16 Bit stereo).
- `OMED/Audio.lyx` kapselt die Ausgabe: auf dem Entwicklungsrechner ALSA ueber
  `std.audio.playback`, unter LyxOS spaeter `ac97.lyx` (STAC9700).
- Menue *Instrument -> Load Sample...* laedt in das in der Instrumentliste
  gewaehlte Instrument. Eine getippte Note wird vorgehoert — im Editiermodus
  zusaetzlich zum Schreiben, ausserhalb nur gespielt.

Bekannte Grenze: die Vorhoerhilfe schreibt den fertigen Puffer blockierend
(rund 0,35 s Stillstand der Oberflaeche). Ab Phase 5 loest die Tickschleife
das ab.

`sampletest.lyx` prueft Loader, Periodentabelle und Mixer ohne Oberflaeche:

```sh
lyxc sampletest.lyx -I /home/andreas/PhpstormProjects/aurum -o sampletest && ./sampletest
```

---

## 13c. Paula-Modus (Phase 5)

`TPaula4ChannelBackend` in `OMED/Playback.lyx` bildet die Hardware nach, statt
sie zu gluetten:

- **Vier Stimmen**, nicht mehr. `NoteOn` auf Kanal 4 landet wieder auf Kanal 0
  (`ch % 4`) — eine fuenfte Note nimmt einer bestehenden ihren Kanal.
- **Hartes Panning**: Kanal 0 und 3 links, 1 und 2 rechts, nichts dazwischen.
  Paula hat keinen Panoramaregler.
- **Lautstaerke vor der Aufloesung**: 8-Bit-Sample mal 6-Bit-Volume, Ergebnis
  wieder 8 Bit. Daher das Rauschen leiser Stimmen — im Software-Mixer bleibt
  es aus, weil dort in 16 Bit gerechnet wird.
- **Periodengrenze 124** (rund 28,8 kHz): darunter kommt der Chipbus dem DMA
  nicht mehr nach, hohe Noten werden nicht mehr hoeher.

Umschaltbar im Menue *Settings*: „Playback: 4 Channel Paula" (Vorgabe) und
„Playback: Modern Mixer". Der Vorhoerkanal folgt dem Track, auf dem getippt
wird — im Paula-Modus hoert man dadurch schon beim Eingeben die Seite.

Messwerte aus `sampletest.lyx`: Kanal 0 nur links (Spitze 16128 / 0), Kanal 1
nur rechts (0 / 16128), Periode 60 wird auf 124 gefangen, bei `Volume 2` bleibt
im Paula-Modus eine Spitze von 384 gegenueber 504 im Software-Mixer — die
sichtbare 8-Bit-Quantisierung.

---

## 13d. 8-Kanal-Modus (Phase 6)

`TOctaMed8ChannelBackend` mischt je zwei Stimmen in Software auf einen
Hardwarekanal — genau das, was OctaMED dem Amiga abgerungen hat:

- **8 Stimmen auf 4 Hardwarekanaelen**: Stimme n liegt auf Kanal `n % 4`, also
  0 und 4 links, 1 und 5 rechts, 2 und 6 rechts, 3 und 7 links.
- **Halbe Dynamik je Stimme**: vor dem Addieren wird halbiert, sonst laeuft die
  Summe aus den acht Bit heraus. Daher klingt der Modus leiser und rauer — das
  ist der Klang, nicht ein Fehler.
- Die Lautstaerke wirkt weiter vor der Aufloesung, es sind also zwei Verluste
  hintereinander: erst das 6-Bit-Volume, dann die Halbierung.
- Der Modus setzt `Song.EightChannel` (`FLAG_8CHANNEL`, 0x40 im Songheader).

Messwerte aus `sampletest.lyx`: eine Stimme allein erreicht 8064 statt 16128
wie im Paula-Modus; zwei Stimmen auf demselben Hardwarekanal kommen zusammen
wieder auf 16128, der andere Kanal bleibt stumm.

Menue *Settings*: „Playback: 4 Channel Paula" (Vorgabe), „Playback: 8 Channel
OctaMED", „Playback: Modern Mixer".

**Beim Umbau gefunden**: der Platzhalter unter der Menueleiste (Phase 1) lag
als gewoehnliches `TControl` genau auf der Leiste und hat als zuletzt
getroffenes Kind jeden Klick abgefangen — die Menues liessen sich nie
aufklappen. Er ist jetzt `TOmedMenuSpacer` mit einem `HitTest`, das `null`
liefert.

---

## 13e. Replay-Engine (Phase 7)

`OMED/Replay.lyx` spielt, `OMED/Playback.lyx` klingt — die Engine kennt kein
Backend-Detail, sie sagt nur „Kanal 2, Periode 214, Volume 48".

Zeitmodell wie im Original: **Tick** (aus dem Tempo gerechnet, Tempo 125 = 50
Ticks je Sekunde wie der PAL-Vertical-Blank), **Line** (TPL Ticks, Noten auf
Tick 0), **Block**, **Playing Sequence**. `TTrackState` haelt je Track das
Gedaechtnis, das die Commands brauchen: Perioden, Volume, Vibrato-Phase,
Portamento-Ziel, Cut- und Delay-Zaehler.

Umgesetzte Commands: `00` Arpeggio, `01`/`02` Slide, `03` Portamento (schlaegt
nicht an), `04` Vibrato, `05`/`06` Slide bzw. Vibrato mit Fade, `07` Tremolo,
`09` TPL, `0A`/`0D` Volume Slide, `0B` Position Jump, `0C` Set Volume (inkl.
`80`–`C0` = Instrumentvolume), `0F` Tempo und Spezialfaelle (`00` naechster
Sequence-Eintrag, `01`–`F0` Tempo, `FF2`/`FF4`/`FF5` Line Delay), `11`/`12`
Slide Once, `14` PT-Vibrato, `15` Set Finetune, `16` Repeat Lines, `18` Cut
Note, `19` Sample Offset, `1A`/`1B` Volume Slide Once, `1D` naechster
Sequence-Eintrag, `1E` Replay Line, `1F` Note Delay und Retrigger.

Noch nicht umgesetzt: `08` (Hold and Decay), `0E` (Synth Jump — es gibt noch
keine Synth-Instrumente), `17` und `1C` sowie die Mix-Mode-Commands `20`–`2F`
(Ausbaustufe laut Abschnitt 7).

**Perioden ohne Hardwaregrenze**: `NotePeriod` schneidet nur noch bei 27 ab.
Die Paula-Grenze (124) sitzt in den beiden Hardware-Backends — eine Grenze in
der Notentabelle waere eine Grenze fuer alle Backends, und im Mixer-Modus sind
die hohen Oktaven hoerbar.

Abgespielt wird ueber einen `TTimer` (20 ms): je Tick ruft er die Engine,
mischt einen Tick Audio und schreibt ihn in den laufenden ALSA-Strom. Der
Schreibvorgang blockiert, bis das Geraet Platz hat — das haelt den Takt, ohne
dass irgendwo eine Wartezeit gezaehlt wird. `TOmedAudio` setzt Format und
Vorbereitung deshalb nur noch EINMAL (`Prepare`) statt bei jedem Puffer.
Der Cursor folgt der gespielten Line.

`replaytest.lyx` prueft die Engine ohne Oberflaeche und ohne Audiogeraet:

```sh
lyxc replaytest.lyx -I /home/andreas/PhpstormProjects/aurum -o replaytest && ./replaytest
```

Geprueft werden Ticklaenge (882 Bilder bei Tempo 125, 1102 bei 100), Anschlag,
Set Volume, Arpeggio (+4/+7 Halbtoene), Portamentoziel und -weg, Slide, Cut
Note, Repeat Lines und der Blockumlauf.

---

## 13f. Song Sequence und Block Properties (Phase 8)

**Datenmodell**: die Playing Sequence ist jetzt eine eigene Klasse `TPlaySeq`
(Add / Insert / Remove / Swap), und ein Song haelt mehrere davon — die
**Sections** aus V5. `TSong.Seq()` liefert die gerade gueltige; alles, was nur
eine Sequence kennt (Replay, Anzeige), fragt dort und muss von Sections nichts
wissen. `TBlock.Resize` aendert Track- und Linezahl und behaelt den Inhalt,
soweit er in die neuen Masse passt.

**Zwei Fenster** in `OMED/Dialogs.lyx`, beide als echte kleine Fenster und
nicht als eingeblendete Bereiche:

- *Block Properties* (Menue **Block**): Name, Tracks, Lines. In OctaMED
  gehoeren diese Masse dem Block und nicht dem Song — anders als bei
  ProTracker, wo alle Patterns gleich gross sind.
- *Song Sequence* (Menue **Block**): die Eintragsliste mit Add, Insert,
  Delete, Up, Down und der Sektionsleiste darueber (`<`, `>`, `New`).

**Listen sind jetzt Bedienelemente**: `TOmedList` meldet die Auswahl ueber
`OnSelect`. Ein Klick in die Blockliste schaltet den Blockeditor um (ohne das
waere ein zweiter Block gar nicht erreichbar), ein Klick in die
Instrumentliste setzt das Eingabeinstrument. *New Block* uebernimmt die Masse
des aktuellen Blocks und haengt ihn an die Sequence.

---

## 13g. MIDI (Phase 9)

`OMED/Midi.lyx` schreibt rohe MIDI-Bytes in eine Geraetedatei (`/dev/midi1`
und Verwandte): kein Sequencer, keine Zeitstempel — die Replay-Engine hat
ihren Takt schon, ein zweiter waere ein zweiter Fehler. Ein Pfad, der sich
nicht oeffnen laesst, schaltet die Ausgabe still ab. Zeigt der Pfad auf eine
gewoehnliche Datei, entsteht dort eine Mitschrift; das ist die einzige Art,
die Ausgabe ohne MIDI-Hardware nachzupruefen.

**Ein Track wird zum MIDI-Track durch das Instrument**: sobald ein Instrument
mit gesetztem `midich` darauf gespielt wird, geht die Note ans Geraet und
NICHT ans Sample-Backend. Derselbe Block treibt so Sample- und MIDI-Spuren
nebeneinander, genau wie in OctaMED. Notennummern: `C-3` (25) wird MIDI 60,
Volume 0..64 wird Velocity 0..127.

Command-Dispatch auf MIDI-Tracks (Abschnitt 7): `01`/`02` Pitchbender je Tick,
`03`/`13` Set Pitchbender, `04` Modulation Wheel (CC 1), `05` + `00`
Controllernummer und -wert, `08` Set Hold Only, `0A` Polyphonic Aftertouch,
`0C` Kanallautstaerke (CC 7), `0D` Channel Aftertouch, `0E` Pan (CC 10),
`18` Cut, `31`–`3F` Set MIDI Controller (die Commandnummer IST die
Controllernummer). `0F` bleibt global (Tempo und Sequenzsprung).

Beim Stoppen gehen alle klingenden Noten aus, danach `All Notes Off` auf allen
16 Kanaelen — ein MIDI-Geraet haelt eine Note, bis jemand sie abstellt.

Zwei neue Fenster: *MIDI Setup* (Geraetepfad, Open/Close, Status) und
*Instrument Properties* (Name, Volume, Transpose, Fine Tune, MIDI Channel,
MIDI Preset — `MIDI Channel: 0` heisst "kein MIDI").

`miditest.lyx` schreibt den Bytestrom in eine Datei und prueft ihn:

```sh
lyxc miditest.lyx -I /home/andreas/PhpstormProjects/aurum -o miditest && ./miditest
```

Ergebnis der Mitschrift: `Program Change 4`, `Note On 60/127`, `CC 10 = 64`,
`CC 7 = 63`, `Note Off 60`, `Note On 72`, beim Stoppen `Note Off` und `CC 123`
auf allen Kanaelen — waehrend die Sample-Spur unberuehrt weiterklingt.

---

## 13h. MMD-Import (Phase 10)

`OMED/MMD.lyx` liest echte OctaMED-Module. MMD ist keine Datei-, sondern eine
Formatfamilie, und der Reader kennt alle drei Stufen:

| | Noten | Blockkopf | Sequence |
|---|---|---|---|
| **MMD0** | 3 Byte, 6-Bit-Note, Instrument-Highbits im Notenbyte | 8-Bit-Masse | 256 Bytes im Songkopf |
| **MMD1** | 4 Byte: Note / Instrument / Command / Datenbyte | 16-Bit-Masse plus Infozeiger | 256 Bytes im Songkopf |
| **MMD2** | wie MMD1 | wie MMD1 | mehrere Playing Sequences (Sections) hinter Zeigern |

Alle Zahlen big endian — die Dateien kommen vom 68000, jedes Feld wird
byteweise zusammengesetzt. Gelesen werden Songflags (`FLAG_8CHANNEL`,
`FLAG_VOLHEX`, LPB und BPM-Modus aus `flags2`), Tempo und TPL, die
Sample-Kopfdaten (rep, replen, midich, midipreset, svol, strans), die
8-Bit-Sampledaten hinter dem Samplefeld sowie Instrument- und Songnamen aus
`expdata`. Jeder Zeiger aus der Datei wird gegen die Dateigroesse geprueft,
bevor er benutzt wird.

Bewusst uebersprungen statt geraten: Synth- und Hybridinstrumente, 16-Bit- und
Stereosamples, zusaetzliche Commandseiten und die Mix-Mode-Erweiterungen.

*Project → Load Song...* laedt ein Modul; schlaegt das fehl, bleibt der
bisherige Song unangetastet. `FLAG_8CHANNEL` schaltet dabei gleich das
passende Backend ein.

`mmdtest.lyx` prueft alle drei Formate gegen erzeugte Testdateien (Blocks,
Zellen inklusive der MMD0-Instrument-Highbits, Flags, Sequence, Sections,
Namen, Samplelaengen) und dass eine fremde Datei abgewiesen wird.

**Beim Testen gefunden — Vega-Fehler**: `TFileDialog` stuerzte beim Bestaetigen
einer Datei ab. `_active` und `_activeApp` standen als Modulvariablen HINTER
der Klasse, die sie benutzt; lyxc legt eine erst nach ihrer Verwendung
deklarierte Modulvariable zweimal an (Befund G in
`lyx-gpi/ISSUE-lyxc-und-vega.md`). `Execute` schrieb also in die eine und die
Handler lasen die andere. Deklaration nach oben gezogen — damit funktionieren
*Load Song* und *Load Sample*.

---

## 13i. Theme laedt wieder (Nacharbeit)

Seit Phase 1 meldete der Start „paula.theme nicht gefunden" — die Meldung war
irrefuehrend: die Datei war da, der Parser brach sie ab. Verschachtelte Bloecke
muessen in Vegas Theme-Grammatik `part NAME { ... }`, `role NAME { ... }`,
`class NAME { ... }` oder ein Zustandsname (`hover`, `pressed`, `focused`, ...)
sein. `paula.theme` schrieb `item { ... }`, `separator`, `header` und `thumb`
ohne das `part` davor; der erste dieser Bloecke (Zeile 71) liess das Laden
scheitern, und ohne Stylesheet stand alles im Vega-Standardtheme.

Der Fehler war nur sichtbar, wenn man `TThemeLoadResult.Messages` ausliest —
`UseOctaMedTheme` wirft das Ergebnis weg und meldet pauschal „nicht gefunden".

Dazu zwei fehlende Typen ergaenzt: `dialog` (Vegas eigene Meldungs-, Rueckfrage-
und Dateidialoge) und `listView` (die Dateiliste). Ohne sie waren genau diese
Fenster die einzigen, die weiss blieben. Eingabefelder zeigen den Fokus jetzt
mit schwarzem Rahmen statt der blauen Leuchtkante des Standardthemes.

Geprueft an den Pixelfarben: Menueleiste, aufgeklapptes Menue, Dialoge und die
Dateiliste stehen auf `#AAAAAA`, das Notenraster auf `#888888`.

---

## 13j. Speichern (MMD1-Export)

Bis hierher konnte das Programm Module lesen, aber nichts behalten.
`TMMDWriter` in `OMED/MMD.lyx` schreibt **MMD1** — und zwar immer, auch wenn
das geladene Modul MMD0 oder MMD2 war:

- **MMD0** quetscht Note und Instrument in sechs Bit und kann ein Command ueber
  `0F` gar nicht darstellen. Ein hier getippter Song passt also nicht
  zuverlaessig hinein.
- **MMD2** brauchte man nur fuer Sections; solange gespeichert wird, was gerade
  gilt, waere das Format ohne Gewinn.

Die Datei entsteht in EINEM Puffer, weil jeder Zeiger darin ein Offset vom
Dateianfang ist: erst wird der Inhalt abgelegt, dann werden Kopf und
Songstruktur mit den nun bekannten Offsets nachgetragen. Strukturen liegen auf
geraden Adressen — auf einem 68000 waere ein Zeiger auf eine ungerade Adresse
ein Address Error.

Geschrieben werden Blocks samt Namen (ueber `MMDBlockInfo`), die
Sample-Kopfdaten, die 8-Bit-Sampledaten, die aktuelle Playing Sequence, die
Songflags sowie Instrument- und Songnamen in `expdata`.

*Project → New Song* legt einen leeren Song mit einem Block und einem
Sequenzeintrag an, *Project → Save Song...* sichert als `.med`.

Geprueft auf drei Wegen: der eigene Leser liest das Geschriebene wieder ein
(`mmdtest.lyx`, Rundlauf inklusive Blockname und Command `1F`), ein
unabhaengiger Python-Parser bestaetigt Kopf, Zeiger, Wortausrichtung und
Notenbytes, und im Editor selbst kommen getippte Noten nach Speichern und
Laden unveraendert zurueck.

---

## 13k. Markierung und Bereichsbefehle

Der Blockeditor kann jetzt einen Bereich markieren:

| Bedienung | Wirkung |
|---|---|
| Shift + Pfeiltasten | Markierung aufziehen (Anker bleibt, wo der Cursor beim ersten Shift stand) |
| Pfeiltasten ohne Shift, `Esc` | Markierung aufheben |
| Klicken und Ziehen | Bereich ueber Lines und Tracks aufziehen |

Menue *Edit*: **Cut Range**, **Copy Range**, **Paste Range**, **Clear Range**.
Ohne Markierung wirken sie auf die Zelle unter dem Cursor — eine einzelne
Zelle zu kopieren soll nicht erst eine Markierung verlangen.

Die Zwischenablage ist selbst ein `TBlock`: ein Ausschnitt IST ein kleiner
Block, und dann setzt ihn Paste ohne Umrechnung wieder ein. Eingesetzt wird ab
dem Cursor; was ueber den Blockrand hinausragt, faellt weg — ein Paste, das den
Block heimlich vergroessert, waere schlimmer als eines, das sichtbar
abschneidet.

**Stolperstein**: `TOmedGrid` ueberschreibt `DoMouseDown` und ruft die Vorgabe
nicht auf, also blieb `Pressed` false und `DoMouseMove` hielt jedes Ziehen fuer
eine blosse Zeigerbewegung. Der Druckzustand wird jetzt von Hand gesetzt.

---

## 13l. Blockoperationen: Transpose, Split, Join

Menue *Block*, unterhalb der Fensterbefehle:

- **Split at Cursor** — teilt den Block an der Cursorline. Der untere Teil wird
  ein eigener Block direkt hinter dem aktuellen.
- **Join with Next** — verschmilzt den Block mit seinem Nachfolger. Die
  Trackzahl richtet sich nach dem breiteren der beiden.
- **Transpose ±1 / ±12** — verschiebt die Noten im markierten Bereich, sonst
  die Zelle unter dem Cursor. Instrument, Command und Datenbyte bleiben
  unangetastet: transponiert wird die Tonhoehe, nicht die Spielanweisung. Was
  aus dem Bereich `C-1`..`B-6` fiele, bleibt stehen — eine Note, die beim
  Transponieren heimlich auf `C-1` zusammensackt, waere schlimmer als eine,
  die sich nicht bewegt.

**Die Playing Sequence zieht mit.** Sie zeigt auf Blocknummern, und die
verschieben sich beim Teilen und Verschmelzen: nach einem Split ruecken alle
Eintraege hinter der Einfuegestelle um eins nach hinten, nach einem Join zeigt
alles, was auf den verschluckten Block zeigte, auf den verschmolzenen. Ohne
diese Nachfuehrung spielte der Song nach jeder Blockoperation etwas anderes,
als er zeigt.

Geprueft in `mmdtest.lyx`: Zellenverteilung nach Split und Join, Linezahlen,
und dass der Sequenzeintrag `1` nach dem Split auf `2` und nach dem Join
wieder auf `1` zeigt.

---

## 13m. Name

Das Programm heisst **Paula** — nach dem Soundchip, der den Klang dieser
Maschinen gemacht hat. Der Name OctaMED gehoert dem Original und bleibt dort;
er steht in diesem Text nur noch da, wo es um das Vorbild geht.

Umbenannt wurden Programmdatei (`paula.lyx`), Binary (`paula`), Fenstertitel,
Theme (`paula.theme`, Themename `paula`) und die Meldungen. Die Units unter
`OMED/` behalten ihr Kuerzel, ebenso das Projektverzeichnis.

**Achtung, doppelter Name**: `Paula` ist jetzt sowohl das Programm als auch der
Chip, auf den sich der Wiedergabemodus *Playback: 4 Channel Paula* bezieht. Im
Menue ist der Chip gemeint.

---

## 14. Bauen

```sh
lyxc paula.lyx -I /home/andreas/PhpstormProjects/aurum -o paula
```

Integration in LyxOS analog `vega/uidemo.lyx`: als ELF bauen, in `build.sh` staggen,
im Compositor per `SysSpawnAsync("/paula.elf"c)` starten. Audio benötigt eine VM mit
AC97:

```sh
VBoxManage modifyvm LyxOS --audio-driver pulse --audio-controller ac97 \
  --audio-enabled on --audio-out on
```

---

## 15. Quellen

- [OctaMED SoundStudio 1.03 — Appendix A: Player commands (libxmp)](https://github.com/libxmp/libxmp/blob/master/docs/formats/octamedss1.03-effects.txt)
- [MMD File Format Documentation (UADE / OctaMED Programmers)](https://github.com/dv1/ion_player/blob/master/extern/uade-2.13/amigasrc/players/med/MMD_FileFormat.doc)
- [MED-Format.txt (MMD0-tools)](https://raw.githubusercontent.com/cpressey/MMD0-tools/master/doc/MED-Format.txt)
- [OctaMED module (MED) — Just Solve the File Format Problem](http://justsolve.archiveteam.org/wiki/OctaMED_module_(MED))
- [OpenMPT `Load_med.cpp`](https://github.com/OpenMPT/openmpt/blob/master/soundlib/Load_med.cpp)
- [OctaMED Professional v3.00 Manual (archive.org)](https://archive.org/stream/OctaMED_Professional_v3.00_1992_RBF_Software_CU_Amiga/OctaMED_Professional_v3.00_1992_RBF_Software_CU_Amiga_djvu.txt)
- [OctaMED — Wikipedia](https://en.wikipedia.org/wiki/OctaMED)
- Intern: Trilium `Vega-UI-Toolkit-Konzept`, `Vega-Controls-Roadmap`, `Vega-Controls-vs-Windows`
