# DM Table Toolkit — Ideas for a TV-as-Game-Board Dungeon Master

Target user: a Dungeon Master who lays an old TV flat (or stands it up) as the battle map,
already has his own software driving it, and runs everything from a **business laptop**
(integrated graphics, probably locked-down IT, no admin installs, 8–16 GB RAM, a single
HDMI out).

Design constraints that shape every idea below:

| Constraint | Consequence |
|---|---|
| Business laptop, no admin rights | Everything must run as static HTML/JS in a browser tab or a single portable `.exe`. No Electron, no Foundry server, no Unity (Arkenforge), no Docker. |
| One HDMI out, TV is the second display | Two browser windows: **DM window** on the laptop, **Table window** dragged to the TV. They talk over `BroadcastChannel` (same origin, zero network). |
| He already has map software | Ours must **sit beside or on top of** his, not replace it. Every idea has an "overlay / adapter" iteration. |
| Resource light | No frameworks bigger than ~30 KB, no WebGL unless optional, all assets local, works offline. Target < 100 MB RAM per tab, no GPU compositing spikes. |
| Physical minis on the glass | Grid must be calibrated to real inches, glare/brightness must be controllable, nothing important should be under a mini. |

## What's actually missing (the research summary)

The tools that exist fall into three buckets, and each leaves the TV-table DM with the same gap:

1. **Full VTTs** (Foundry, Roll20, Fantasy Grounds) are built for *remote* play. On a TV
   table they are heavy, need a second client for the player view, and their fog-of-war
   workflows "take significantly longer and disrupt flow" (Ben Straub, *VTTs Are Wrong*).
2. **Light map viewers** (Owlbear Rodeo, VTTRetro, gTove) do maps and tokens well but stop
   there: no initiative, no conditions, no audio, no handouts, no session pacing.
3. **Companion screens** (Tableward, Table Display) show initiative/portraits on a second
   screen but assume *they* are the map layer too, so they don't compose with someone's
   existing map software.

Nobody ships the **glue**: a lightweight, local-only, two-window control surface that
layers the *non-map* parts of running a game (turn order, conditions, secrets, audio, pacing,
handouts, physical-mini calibration) over whatever map software the DM already runs.
That glue is what this repo should build.

Sources used: Tableward (tableward.app), Black Lantern Forge's TV setup guide,
D&D Beyond forum threads on "VTT that allows two outputs", Ben Straub's *VTTs Are Wrong*,
Czepeku's VTT comparison, Owlbear Rodeo / VTTRetro / gTove project pages.

---

## The 7 ideas

Each idea has three iterations: **v0 (weekend build)**, **v1 (useful every session)**,
**v2 (the ambitious version)**. Ideas are ordered by how much daily pain they remove.

### 1. Table HUD — a transparent overlay for the TV, on top of his map software

**The gap:** his map fills the TV, but initiative, round counter, active conditions, and
"whose turn" live on paper or on the laptop where players can't see them.

- **v0 — Corner strip.** A borderless browser window ("always on top" via the OS, or a
  `window.open` with no chrome) sized as a 1080×80 strip along the TV's top edge. Shows
  initiative order as name chips, highlights the active creature, shows round number.
  Controlled from a DM tab via `BroadcastChannel`. No transparency needed: he shrinks his
  map window by 80 px.
- **v1 — Edge rails + condition badges.** Strips on two edges (top: initiative; side:
  conditions per creature with icons and remaining rounds). Click-through mode so he can
  still touch his map software underneath. Colour-blind safe palette, large fonts legible
  from 1 m away across a table.
- **v2 — True overlay.** A tiny portable helper (AutoHotkey script or a 200-line Rust/Go
  `.exe`, no install) that makes the browser window layered/transparent on Windows so the
  HUD floats over the map with alpha. Adds an "attention pulse" (border glow) when it's a
  player's turn, and a "hide everything" panic key for surprise reveals.

### 2. Initiative & Condition Engine with a *player-safe* projection

**The gap:** initiative trackers exist, but they show the DM's data (monster HP, hidden
creatures) or nothing. A TV table needs two renderings of one truth.

- **v0 — Two-view tracker.** DM tab has full rows (HP, AC, notes, hidden flag). Table view
  shows only name, portrait, turn marker, public conditions. State is one JSON object in
  `localStorage`, mirrored to the TV window. Keyboard driven: `n` next turn, `d` damage,
  `c` condition.
- **v1 — Rules-aware timers.** Conditions carry durations ("until end of X's next turn",
  "1 minute", "concentration"); the engine auto-expires them and prompts saves at the
  right moment. Concentration checks fire on damage. Legendary/lair actions get slots in
  the order. Import a party from a CSV he types once.
- **v2 — Encounter deck.** Pre-build encounters in a folder of `.json` files; drop one in
  to load monsters with stat blocks from the SRD (bundled, offline). "Group" monsters
  roll once. Log every round to a text file so he can recap the fight next week.

### 3. Physical-mini calibration & glare kit

**The gap:** everyone who puts a TV flat hits the same problems: the grid isn't 1 inch,
the map is too bright under minis, and the screen edge shows the OS taskbar.

- **v0 — Calibration page.** A full-screen page that draws a grid; two keys nudge the
  scale until a physical 1" base fits a square. Saves the TV's pixels-per-inch to
  `localStorage` and exposes it (`?ppi=42.3`) so his map software or any other tool can
  read it. Also a "black bars" frame to hide taskbar/overscan.
- **v1 — Brightness & wash profiles.** One-key toggles: *Dim* (a 40% black overlay so
  minis stand out and glare drops), *Night*, *Underwater*, *Fog* tints. Runs as the same
  overlay window as idea 1, costs nothing.
- **v2 — Reference overlays.** Blast templates (cones, spheres, lines) at true inch scale
  that he can drag onto the glass under minis, a range ruler, and a "you can see this far"
  circle around a token. Measured in feet, drawn in calibrated pixels.

### 4. Secrets, reveals & handouts channel

**The gap:** the TV is player-facing, so *everything* on it is public. DMs need a way to
push a single thing to the table (a portrait, a letter, a map fragment, a countdown) and
pull it back without touching the map.

- **v0 — Show/hide one image.** A DM tab lists a local folder of images (via the browser
  file picker, cached in IndexedDB). Click one → it fades in over the TV. Esc → gone.
  Villain portraits, handouts, tavern menus.
- **v1 — Reveal effects.** Curtain wipe, burn-in, "torn page", slow zoom. Text cards with
  large type ("The door slams shut."). Queue several reveals in order for a scripted scene.
  A "player-eyes-only" mode that blacks the TV instantly (someone's phone rang, or an
  ambush).
- **v2 — Per-player secrets.** QR code on the TV → each player opens a tiny page on their
  phone (still no server: use a WebRTC data channel initiated from the DM tab, or fall
  back to a shared LAN folder). DM sends a note to one player's phone only ("you notice
  the cleric is lying"). Nothing to install, works on any phone.

### 5. Soundscape that follows the encounter

**The gap:** ambient audio tools (Syrinscape, Kenku) are separate apps, subscription-bound,
and not tied to what's on the table. On a business laptop the DM wants one tab, local
files, no cloud.

- **v0 — Local soundboard.** Grid of buttons bound to local audio files (Web Audio API,
  files picked once and cached). Loop toggle, fade, master volume, keyboard hotkeys.
- **v1 — Scenes.** A "scene" = a set of ambient loops + a one-shot palette. "Tavern",
  "Cave", "Combat". Switching scenes cross-fades. Scenes are named the same as encounter
  files from idea 2 so loading an encounter can auto-start its audio.
- **v2 — Reactive cues.** The initiative engine emits events (turn start, crit, death,
  bloodied); the soundboard subscribes and plays stingers. Optional TV vignette flash on
  crits (idea 1 overlay).

### 6. Session pacing & prep binder

**The gap:** the on-table software handles the fight; nothing handles the *rest* of the
session: which scene are we in, what's the clock, which NPCs are here, what did I promise
last week.

- **v0 — Markdown session runner.** Load a `.md` file for tonight; it renders as a
  sidebar with headings collapsed. Checkboxes for beats. A big "session timer" and a
  "time since last break" nag.
- **v1 — Scene cards.** Each `##` heading becomes a card with linked encounters (idea 2),
  reveals (idea 4), and audio scenes (idea 5). Pressing "Go" on a card loads all three.
  This is the "combine it all" piece the DM asked for.
- **v2 — Campaign memory.** Auto-generate a recap from the round logs, revealed handouts,
  and ticked beats. NPC cards with "last seen" and "knows about". Player-facing recap
  slide on the TV at session start.

### 7. Adapter layer for *his* existing software

**The gap:** none of the above matters if it fights the software he already has. This
idea is the integration contract, and it should be built first.

- **v0 — Two-window protocol.** Publish a tiny spec: a `BroadcastChannel("table")` message
  format (`{type, payload, ts}`) that any tool can join. Provide a 40-line JS snippet he
  can paste into his software so it emits `map.loaded`, `token.moved`, `fog.revealed`.
  Everything else in this repo only needs to *listen*.
- **v1 — Screen-region layout manager.** A layout page that reserves regions of the TV
  (map area, HUD strips, reveal layer) and tells his software the map viewport size via
  URL params or a `postMessage` if his tool is a web page in an `<iframe>`. If his tool is
  a native app, the layout manager just positions our transparent windows around it.
- **v2 — Hotkey bridge.** A portable, no-install `.exe` (or AutoHotkey) that maps a
  cheap USB macro pad / numpad to both his software and ours: next turn, dim, reveal,
  panic-blank. The DM never alt-tabs mid-fight.

---

## Other things that seem missing (bonus ideas, one iteration each)

- **Laptop-death insurance.** A single `table.html` that mirrors current state to a USB
  stick every 30 s; plug it into any other machine and the session resumes. Business
  laptops get IT reboots at the worst time.
- **"What can they see" spot check.** Given calibrated PPI and a token position, draw
  exact light radii (bright/dim) for torches and darkvision, so fog decisions are
  consistent without dynamic lighting engines.
- **Rules card flash.** Hotkey + condition name → a large, player-readable rules card on
  the TV for 15 s ("Grappled: speed 0…"). Ends the "what does frightened do again"
  interruption.
- **Player initiative input on phones.** Players type their own roll on the QR page
  (idea 4 v2), the tracker sorts itself. Removes the slowest minute of every fight.
- **Loot & shop table.** Player-facing price list rendered from a CSV; DM marks items
  sold. Trivial, but nobody does it on the table screen.
- **Physical dice cam (optional, later).** The laptop webcam pointed at a dice tray;
  ML is too heavy for this hardware, so instead just a picture-in-picture on the TV so
  everyone sees the roll. Zero compute.
- **Table etiquette timer.** A quiet per-turn shot clock on the HUD, DM-configurable,
  off by default. Solves the analysis-paralysis problem without the DM nagging.

## Recommended build order

1. Idea 7 v0 (protocol) + Idea 1 v0 (HUD strip) — proves the two-window model in one evening.
2. Idea 2 v0/v1 (tracker) — the thing he'll use every single session.
3. Idea 3 v0 (calibration) — small, and every other overlay depends on the PPI value.
4. Idea 4 v0, Idea 5 v0 — reveals and audio, both are ~300 lines each.
5. Idea 6 v1 — the scene card that ties all of it together.
6. Only then consider any `.exe` helper (1 v2, 7 v2).

## Tech stance

- Plain HTML + vanilla JS modules, one `index.html` per tool, a shared `bus.js`.
  No build step, so it runs from a folder on the desktop or a USB stick.
- State: `localStorage` for settings, IndexedDB for cached media, JSON files for prep.
- Cross-window: `BroadcastChannel` first, `postMessage` for iframe embedding, WebRTC
  only for the optional phone features.
- Rendering: DOM + CSS for HUD/text, `<canvas>` 2D for grids/templates. No WebGL.
- Audio: Web Audio API with pre-decoded buffers for zero-latency one-shots.
- Verified budget target: idle CPU < 2%, each tab < 100 MB, works in Edge (what a business
  laptop will have) with no extensions.
