# ArkComposer

Windows score editor and AI-assisted composition workstation.

> Write, edit, arrange, and refine music in a full score editor, then use built-in AI tools or MCP-connected clients when you want generation help.

![ArkComposer Screenshot](docs/screenshot.png)

---

## 👨‍💻 About the Author

**Park Byoung Gu (박병구)**

Director & SW Group Manager at **[AXT (Ajinextek)](https://www.ajinextek.com)** — a leading company in industrial motion controller markets — where he leads software R&D.

ArkComposer is a personal hobby project, built on weekends and spare moments, driven by a deep love of music.

---

## What Makes ArkComposer Different

- Full score editing and composition tools live in one workflow instead of separate notation and generator apps
- `Edit Measure (MIDI)` gives you a modeless piano-roll editor for precise measure-level repair, timing edits, velocity shaping, and AI-assisted rewriting
- Built-in composition helpers cover rhythm, melody variation, chord suggestion, voice leading, accompaniment, and AI generation
- An MCP server is built into the app, so external AI clients can control the same score editor when needed
- Visual analysis overlays and score-aware editing tools stay tied to the musical structure instead of raw MIDI only

---

## Features

### Top-Bar Workflow
- Undo / Redo stays at the far left for quick recovery
- `Edit Measure` opens `Edit Measure (MIDI)` for the current measure selection
- `Measure Select`, `Note Select`, and `Score Edit` split selection from direct score editing
- Pinned note and eraser tools can switch the app into score editing immediately
- Rhythm, Melody, Helper, and AI Helper menus are grouped on the right side of the top bar
- A status bar below the top menu shows the current mode, tool, track, selected measures, rhythm, melody, and AI state

### Score Editing
- Multi-track score editor with treble and bass clef support
- Note input: whole, half, quarter, 8th, 16th, 32nd (+ dotted variants)
- Accidentals, dynamics (p, mf, f, cresc, dim), tempo marks (accel, rit)
- Key signature: major/minor, all 15 keys, 3 apply modes (pitch-fixed / position-fixed / transpose)
- Time signatures: 4/4, 3/4, 2/4, 6/8, 12/8
- Chord symbol display above measures
- Lyrics editor with per-measure editing
- Multi-document tabs with independent undo/redo per tab

### Edit Measure (MIDI)
- Piano-roll style editor for note timing, duration, pitch, and velocity inside selected measures
- Edit / Insert / Delete / Tie modes for fast repair without leaving the score workflow
- Track-wide measure navigation slider for moving across the active track while staying in the same editor
- Built-in playback controls for checking only the current track or all tracks
- Velocity popup editor for shaping dynamics inside the current measure range
- Designed for repeated measure cleanup and detailed note-level correction after generation or manual entry

### Track Management
- Add, delete, merge, split (by pitch range), reorder tracks
- Hide/show individual tracks
- Per-track instrument settings: SoundFont, Program (0–127), MIDI Channel, Bank MSB/LSB, Volume, Pan, Percussion

### Compose Assist
- `Rhythm` reassigns rhythm patterns to selected measures
- `Melody` applies transforms such as inversion and other note-sequence variations
- `Helper` groups chord suggestion, voice leading, accompaniment, and composition guidance
- `AI Helper` groups AI-backed generation tools for voicing and accompaniment
- Voice Leading provides tunable parameters for harmonic line shaping and chord motion
- Accompaniment supports Arpeggio, Chord Beat, Drum Groove, and AI-driven styles

### AI Voicing and AI Accompaniment
- `AI Voicing` is available from the AI Helper menu and directly inside `Edit Measure (MIDI)`
- Use `AI Voicing` when the current track already has harmonic intent but needs cleaner voicing, redistribution, or regeneration
- `AI Accomp` inside `Edit Measure (MIDI)` generates accompaniment that follows the current melody and harmony while staying in the measure-edit workflow
- AI accompaniment styles include `AI Chord Beat`, `AI Arpeggio`, `AI Free`, `AI Free (Melody Variation)`, and `AI Drum Beat`
- Built-in ONNX and TinyGRU models can be selected from the app for local generation

### AI Integration via MCP
- Full MCP server built into the app (stdio and TCP modes)
- Compatible with **Claude Code** and other MCP-capable clients
- Compose through natural language, then continue refining the result in the score editor or `Edit Measure (MIDI)`
- Scope control includes targeted edits such as selected range and whole-song operations
- MCP exposes self-documentation so connected AI clients can inspect available tools

### Audio & Export
- Playback with SoundFont synthesis (.sf2)
- Export audio: **WAV** (lossless) / **MP3** (compressed, requires `lame.exe` in `Tools/`)
- Export score: **PDF** / **PNG** / **JPG**
- Import / Export **MIDI**
- Native save format: `.arkscore` / `.json` (human-readable)

### Plugin Filter Structure
- Built-in audio effect filters (Distortion, Echo, and more)
- Extensible plugin architecture for custom effects

---

## AI Composition via MCP

### Mode 1 — stdio (Claude Code) ✅ Tested

Launch ArkComposer in MCP stdio mode:

```bash
ArkComposer.exe --mcp-mode
```

Add to your `claude_desktop_config.json`:

```json
{
  "mcpServers": {
    "arkcomposer": {
      "command": "C:/path/to/ArkComposer.exe",
      "args": ["--mcp-mode"]
    }
  }
}
```

Then talk to Claude Code:
```
"Create an 8-bar melody in C major, 4/4 time"
"Add an arpeggio accompaniment track"
"Apply jazz voice leading to measures 1–4"
"Continue the melody for 4 more measures"
```

### Mode 2 — TCP (Codex CLI and others) ✅ Tested

1. Launch ArkComposer normally
2. In the left palette, click **MCP Server**
3. Enter a TCP port (e.g. `7890`) → Click **Start**
4. Connect your MCP-compatible client to `localhost:7890`

> **Note on ChatGPT:** ChatGPT does not natively support MCP.  
> To use ArkComposer with ChatGPT, you need to implement a separate OpenAPI server that bridges ChatGPT function calls to ArkComposer's MCP tools.

---

## MCP Tool Surface

ArkComposer exposes a full score editing API over MCP:

| Category | Tools |
|----------|-------|
| Score reading | `get_score`, `get_score_range` |
| Note editing | `add_note`, `add_notes_batch`, `change_note`, `delete_note` |
| Track management | `add_track`, `delete_track`, `set_track_props` |
| Measure management | `add_measure`, `delete_measure`, `delete_measures_range`, `clear_measure`, `set_measure_props` |
| Document | `new_song`, `set_title`, `undo` |

> Full AI usage guide: [`docs/HowToUse_ArkComposerForLLM.md`](docs/HowToUse_ArkComposerForLLM.md)  
> Also available at runtime via MCP resource: `arkcomposer://docs/how-to-use-llm`

---

## Installation

### Requirements
- Windows 10 / 11 (64-bit)
- `lame.exe` in `Tools/` folder — required for MP3 export ([download](https://lame.sourceforge.io/))

### SoundFonts
Place `.sf2` files in the `SoundFonts/` folder.  
ArkComposer ships with:
- `MuseScore_General.sf2` — MIT License (S. Christian Collins)
- `FluidR3_GM2.sf2` — MIT License (Frank Wen)

### Download
👉 **[Latest Release](https://github.com/arkark2010arkark/ArkComposer/releases/latest)**

---

## Roadmap

- [ ] macOS / Linux support (C++ / JUCE core is cross-platform ready)
- [ ] Cloud sync
- [ ] Plugin marketplace
- [ ] More AI model integrations

---

## Credits

| Component | Author | License |
|-----------|--------|---------|
| JUCE Framework | Raw Material Software | JUCE Starter |
| MuseScore_General.sf2 | S. Christian Collins (based on FluidR3 by Frank Wen) | MIT |
| FluidR3_GM2.sf2 | Frank Wen | MIT |
| FluidSynth | Peter Hanappe and contributors | LGPL 2.1 |
| LAME MP3 Encoder | Mark Taylor and contributors | LGPL 2 |
| SDL3 | Sam Lantinga and contributors | zlib |
| libsndfile | Erik de Castro Lopo and contributors | LGPL 2.1 |

See [`CREDITS.txt`](CREDITS.txt) for full details.

---

## License

ArkComposer is free to download and use.  
Music created with ArkComposer may be used for personal or commercial purposes without restriction.  
Redistribution of ArkComposer itself is not permitted.  
Source code is proprietary and not included in this distribution.

---

*Built with C++ and [JUCE](https://juce.com/)*  
*Website: [arkcomposer.com](https://arkcomposer.com)*
