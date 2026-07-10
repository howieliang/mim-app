# Mind In Image App

Current Version: v26.1 | Last Update: Jul 9, 2026

Interactive prototype of the CuCo MIM cognitive assessment game, co-developed bu Eindhoven University of Technology (TU/e) and Wageningen University & Research (WUR). This is a single-file HTML implementation that mirrors the full game flow and visual design of the reference implementation (`DemoGame_MIM_v1`).

Contact: Dr. Rong-Hao Liang (r.liang@tue.nl)

## Overview

The game assesses creative cognition through three tasks:

- **Convergence** — 5 rounds. Select the 3 images that have the most in common.
- **Divergence** — 5 rounds. Select the 3 images that stand out as the most different.
- **Memory** — 3 rounds. Identify 2 images previously chosen during Convergence and Divergence. Memory becomes available only after both Convergence and Divergence are completed.

Each task presents 10 image cards per round. Card selections are recorded and used to seed the Memory task with the participant's own previously chosen images.

## Running

No build step or dependencies. Serve the folder over HTTP — `file://` will not work because audio loading requires HTTP.

```bash
cd TUe_MIM_v1
python3 -m http.server 8080
# Open http://localhost:8080
```

## File Structure

```
TUe_MIM_v1/
├── index.html          # Entire application (single file)
└── Assets/
    ├── Cartas/         # I01.png – I97.png  (card images)
    ├── Audio/
    │   ├── Palabras/   # 01.mp3 – 97.mp3   (card word audio)
    │   ├── Instrucciones/ # 00.mp3 – 09.mp3 (screen instruction audio)
    │   ├── Efectos/    # S00–S04.mp3        (UI sound effects)
    │   └── Musica/     # musicaMenu.mp3     (background music)
    ├── Generales/      # GN00–GN27.png      (UI buttons, logo, overlays)
    ├── FondoEstatico/  # 00–06.png          (static background images)
    ├── Fondo/          # F01–F40.png        (animated menu background)
    ├── Transision/     # Estrellas/Nubes/Ciudad/… (page transition frames)
    ├── ExpConvergencia/ # 00–30.png         (convergence example animation)
    ├── ExpDivergencia/  # 00–33.png         (divergence example animation)
    ├── ExpMemoria/      # 00–16.png         (memory example animation)
    └── GIFS/EMOJI1/    # 00–19.png          (task-complete emoji animation)
```

## Game Flow

```
Loading → Start Screen → Task Menu
                             ├── Convergence → Instructions → Cards (5 rounds) → Complete
                             ├── Divergence  → Instructions → Cards (5 rounds) → Complete
                             └── Memory*     → Instructions → Cards (3 rounds) → Complete
```

\* Memory unlocks after both Convergence and Divergence are completed.

## Implementation Notes

**Single file.** All logic, styles, and HTML live in `index.html`. No frameworks, bundlers, or external dependencies.

**State machine.** A `taskType` variable (`conv` / `div` / `memo`) and a `round` counter drive the game loop, mirroring the `estado` string in DemoGame.

**Card maps.** `mapaConvergencia` and `mapaDivergencia` are exact copies of the round-image index arrays from DemoGame. Indices are 0-based; filenames are 1-based (`I01.png` = index 0).

**Memory seeding.** The memory map is generated dynamically from the user's actual Convergence and Divergence picks (`convUnicas` / `divUnicas` Sets), supplemented by random distractors drawn from the remaining card pool.

**User ID.** On first launch the manager is prompted to set a User ID, which is persisted in `localStorage` under `mim_manager_id` and reused on every subsequent visit. If the prompt is skipped, the ID is set to `guest`. The ID can be changed at any time from the start screen via a password-protected reset flow.

**Data logging.** Every time the player presses Continue on a card screen, a log entry is recorded containing: user ID, game type (1 = Convergence, 2 = Divergence, 3 = Memory), round number (0 = instruction screen continue, 1–5 = card rounds), the image IDs of the selected answers, and a timestamp (`yyyy-mm-dd-hh-mm-ss`). Entries are logged to the browser console and accumulated in a session-level `sessionLog` array accessible from DevTools.

**Remote logging.** Log entries are sent in real time to a Data Foundry database via HTTP POST. Each entry is transmitted as a JSON object with fields `user_id`, `game_type`, `round_number`, `answer_1`, `answer_2`, `answer_3`, and `timestamp`.

**Offline resilience.** If a send fails due to a network error or server unavailability, the entry is saved locally in `localStorage` under `mim_pending_log`. Failed entries are retried automatically on the next successful send and on every page load. The start screen displays a warning indicator showing the number of unsent entries; tapping it opens a view of the queued data with a manual retry option.

**Audio.** Two independent audio systems: (1) instruction/card-word audio — one track at a time, toggle on button press; (2) sound effects (`sfx`) — play independently on top. Background music loops on menu screens and stops during card play.

**Transitions.** Page changes use frame-sequence animations (20ms/frame) from `Assets/Transision/`. Convergence uses Estrellas, Divergence uses Nubes, Memory uses Ciudad — matching DemoGame exactly.

**Layout.** A centered 9:16 portrait column (optimised for 1080×1920 mobile screens) scales to any window size via a JS `resize()` listener.
