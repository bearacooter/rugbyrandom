# dnd — DM Table Toolkit (charter)

Lightweight, offline, two-window tools for a Dungeon Master who uses a TV as the game
board and runs the session from a business laptop. Designed to sit **beside** the DM's
existing map software, not replace it.

This folder is intended to be split into its own repository. Suggested layout once it is:

```
dm-table-toolkit/
  IDEAS.md            # this planning doc
  shared/bus.js       # BroadcastChannel protocol (idea 7)
  hud/                # TV overlay strips (idea 1)
  tracker/            # initiative + conditions (idea 2)
  calibrate/          # PPI + glare profiles (idea 3)
  reveal/             # handouts / secrets (idea 4)
  sound/              # local soundboard (idea 5)
  session/            # markdown session runner (idea 6)
```

Rules for every tool in here:

- Plain HTML/JS, no build step, no server, no install. Open the file, drag a window to the TV.
- Works offline in Edge on integrated graphics.
- Talks to other tools only through the `bus.js` message contract.

See `IDEAS.md` for the full feature ideas, iterations, and build order.
