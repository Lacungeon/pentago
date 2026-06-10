# Pentago — AI Agent Arena

A Python implementation of the board game **Pentago**, built as a playground for game-playing AI agents. Agents with different strategies compete head-to-head or battle through a single-elimination tournament, and every game can be replayed visually with a `pygame` viewer.

## What is Pentago?

Pentago is a two-player abstract strategy game played on a 6×6 board made of four 3×3 quadrants. On each turn a player does two things:

1. **Place** one marble on any empty cell.
2. **Rotate** one of the four quadrants 90° clockwise or counterclockwise.

The first player to get **five of their marbles in a row** — horizontally, vertically, or diagonally — wins. If the board fills with no winner, the game is a draw.

## Features

- Full Pentago game engine (placement, quadrant rotation, win/draw detection).
- Four pluggable AI agents with increasingly sophisticated strategies.
- A 1-vs-1 game runner (`main.py`).
- A single-elimination **tournament** system (`multiplayer.py`) that ranks all agents.
- A `pygame` replay viewer that animates every move of a recorded game.
- Game history persisted to disk so any match can be re-watched.

## The Agents

| Agent | File | Strategy |
|-------|------|----------|
| **RandomAgent** | [random_agent.py](random_agent.py) | Picks a random empty cell, quadrant, and rotation direction. |
| **AdjacentAgent** | [adjacent_agent.py](adjacent_agent.py) | Prefers placing marbles next to its own existing marbles. |
| **BuildAgent** | [build_agent.py](build_agent.py) | Simulates every possible move and chooses the one that builds its own longest sequence. |
| **BuildAndDenyAgent** | [build_and_deny_agent.py](build_and_deny_agent.py) | Scores each move by *both* extending its own sequence and shortening the opponent's, then picks the highest net gain. Also takes an immediate winning move when one exists. |

All agents expose the same interface — a `play()` method that returns a move as `(x, y, quarter, direction)` — so they are interchangeable.

### Move encoding

- **`x`, `y`** — row and column (0–5) where the marble is placed.
- **`quarter`** — which 3×3 quadrant to rotate:
  - `1` → top-left `[:3, :3]`
  - `2` → top-right `[:3, 3:]`
  - `3` → bottom-left `[3:, :3]`
  - `4` → bottom-right `[3:, 3:]`
- **`direction`** — `+1` for clockwise, `-1` for counterclockwise.

## Project Structure

| File | Purpose |
|------|---------|
| [pentago.py](pentago.py) | Core game engine: the `Pentago` class, the game loop, and win/draw detection. |
| [apply_move.py](apply_move.py) | Applies a move to the board (place + rotate) and appends the new state to the history file. |
| [validator.py](validator.py) | Validates a move (in-bounds, valid quadrant/direction, target cell empty). |
| [latest_board_state.py](latest_board_state.py) | Reads the most recent board state back from the history file. |
| [highest_sequence.py](highest_sequence.py) | Computes the longest marble run for a player — used by the smarter agents. |
| [new_game_env.py](new_game_env.py) | `pygame` viewer that animates a recorded game. |
| [main.py](main.py) | Entry point for a single 1-vs-1 match. |
| [multiplayer.py](multiplayer.py) | Single-elimination tournament runner with ranking and logging. |
| [mod_multiplayer.py](mod_multiplayer.py) | Extended tournament runner supporting *best-of-N* matches and auto-padding the bracket to a power of two. |
| [agents.txt](agents.txt) | Tournament roster (one agent per line: `AgentType name id`). |
| [original.hbd](original.hbd) | A blank starting board, copied at the start of each game. |
| [record.hbd](record.hbd) | History of board states for the current/last game (used for replay). |
| [results.txt](results.txt) | Output log of the most recent tournament. |

> **Note:** `.hbd` files are plain CSV-style snapshots of the board — each line is one flattened 6×6 grid, where `0` = empty, `1` = player 1, `2` = player 2.

## Requirements

- Python 3.8+
- [NumPy](https://numpy.org/)
- [pandas](https://pandas.pydata.org/)
- [pygame](https://www.pygame.org/) — for the replay viewer
- [names](https://pypi.org/project/names/) — only required by `mod_multiplayer.py` (random bracket fillers)

Install everything with:

```bash
pip install numpy pandas pygame names
```

## Usage

### Play a single match

Edit the two agents at the top of [main.py](main.py), then run:

```bash
python main.py
```

This plays one game between the two configured agents and then opens the `pygame` window to replay it move by move.

### Run a tournament

1. List your competitors in [agents.txt](agents.txt), one per line in the form:

   ```
   AgentType name id
   ```

   For example:

   ```
   RandomAgent ali 1
   BuildAgent naghi 2
   BuildAndDenyAgent kuber 1
   AdjacentAgent nars 2
   ```

2. Run the tournament:

   ```bash
   python multiplayer.py
   ```

   The bracket plays out, the final match is shown in the viewer, and full results (round-by-round and final ranking) are written to [results.txt](results.txt).

   > `multiplayer.py` expects the number of agents to be a power of two (2, 4, 8, …).

### Best-of-N tournament

[mod_multiplayer.py](mod_multiplayer.py) supports best-of-N matches and automatically pads the roster up to the next power of two with random agents:

```bash
python mod_multiplayer.py 3   # best-of-3 per match
```

The number of matches per pairing must be odd.

## How it works

1. At the start of every game, [original.hbd](original.hbd) (a blank board) is copied to [record.hbd](record.hbd).
2. The two agents alternate turns. Each `play()` returns a move, which [apply_move.py](apply_move.py) applies to the board and appends to [record.hbd](record.hbd).
3. After each move, the engine checks all rows, columns, and diagonals for a five-in-a-row win (or a full-board draw).
4. When the game ends, [new_game_env.py](new_game_env.py) reads [record.hbd](record.hbd) and replays the whole game in a `pygame` window, with the latest board on the right and the previous state on the left.

## Ideas for extension

- Add new agents (e.g. minimax or Monte Carlo Tree Search) by implementing a class with a `play()` method.
- Re-enable the move `validator` in the game loop to enforce legal-move checking.
- Swap the time-based replay for the click-to-advance mode already stubbed in [new_game_env.py](new_game_env.py).
