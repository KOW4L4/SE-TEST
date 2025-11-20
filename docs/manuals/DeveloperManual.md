# Developer Manual – Jungle CLI

This manual describes how to open, run, debug, and extend the Jungle (Dou Shou Qi) command‑line game.

## Target Platform and Tools

- **Platform**: Windows 11 (64‑bit).
- **Python version**: Python **3.12** (CPython). The code should also run on Python 3.11+, but development and testing were done on 3.12.
- **IDE**: Visual Studio Code with the **Python** extension (recommended). Any IDE that understands a `src` layout and virtual environments also works.
- **Runtime dependencies**: Python **standard library only** (no third‑party packages required to run the game).
- **Developer/test tools** (optional but recommended):
  - `coverage` for code‑coverage measurement.

Readers are assumed to already know how to install Python and set up virtual environments on Windows.

## Project Layout

The repository follows a standard `src` layout:

- `src/`
  - `main.py` – entry point; calls `cli.shell.run_shell()`.
  - `cli/`
    - `shell.py` – REPL loop, command parsing/dispatch, high‑level game orchestration.
    - `renderers.py` – text rendering of the board and remaining pieces.
    - `utils.py` – command parsing helpers, random name generation, and file‑name utilities.
  - `model/`
    - `board.py` – board representation, legal‑move checking, and `InvalidMoveError`.
    - `piece.py`, `position.py`, `move.py` – piece types, coordinates, and move records.
    - `enums.py` – enumerations such as `PlayerSide`, `SquareType`, and piece ranks.
    - `game_state.py` – overall `GameState` (players, board, move log, undo credits, winner).
    - `serialization.py` – JSON‑based load/save of games and records (`SerializationError`).
- `tests/`
  - `test_model.py` – unit tests for core model behaviour.
  - `COVERAGE.md` – coverage report and notes.
- `docs/`
  - Design, requirements, and this manual.

Guideline: keep **rules and state** in `src/model`, and keep **I/O and presentation** in `src/cli`.

## Getting the Code and Setting Up

1. **Clone** or download the repository on Windows:
   ```powershell
   git clone https://github.com/KOW4L4/SE-TEST.git
   cd SE-TEST
   ```
2. **Create and activate** a Python 3.12 virtual environment (example using `venv`):
   ```powershell
   python -m venv .venv
   .\.venv\Scripts\activate
   ```
3. **(Optional) Install dev tools** such as coverage:
   ```powershell
   python -m pip install --upgrade pip
   python -m pip install coverage
   ```

## Building and Running the Game

The project is a pure Python application, so there is no separate build step beyond ensuring dependencies are installed.

### Run from the Command Line

From the repository root (with the virtual environment activated):

```powershell
python -m src.main
```

This starts the interactive shell and shows the prompt:

```text
Welcome to Jungle! Type 'help' to see available commands.
jungle>
```

### Run in Debug Mode (VS Code)

1. Open the repository root in VS Code: **File → Open Folder… → `SE-TEST`**.
2. Ensure the interpreter is set to the virtual environment (`.venv`).
3. Create a debug configuration (`.vscode/launch.json`) with a configuration similar to:
   ```json
   {
      "version": "0.2.0",
      "configurations": [
         {
            "name": "Debug Jungle CLI",
            "type": "python",
            "request": "launch",
            "module": "src.main",
            "console": "integratedTerminal",
            "justMyCode": true
         }
      ]
   }
   ```
4. Set breakpoints in the relevant files (for example `src/cli/shell.py` or `src/model/game_state.py`).
5. Press **F5** or use the **Run and Debug** view and select **Debug Jungle CLI**.

The debugger will start the game in the integrated terminal, where you can step through command handling and game‑state updates.

## Running Tests

From the repository root (virtual environment activated):

- **All unit tests**:

  ```powershell
  python -m unittest discover -s tests
  ```
- **Coverage for the model** (requires `coverage`):

  ```powershell
  python -m coverage run -m unittest discover -s tests
  python -m coverage report --include "src/model/*"
  ```

Update `tests/COVERAGE.md` when you make changes that significantly affect coverage.

## Launching with Different Entry Points

- **Standard interactive shell**: `python -m src.main` (recommended).
- **Direct module invocation for debugging**: you can also configure your IDE to run `cli.shell` directly by calling `run_shell()`, but this is equivalent to going through `main.py`.

Always invoke the game as a **module** (using `-m`); this ensures that relative imports such as `from ..model.game_state import GameState` work correctly.

## Internal Architecture Overview

### CLI Layer (`src/cli`)

- `JungleShell` in `shell.py` is a simple command loop around `input()` with a dictionary of command handlers.
- Commands are parsed by `parse_command()` in `utils.py` and dispatched to methods such as `_cmd_move`, `_cmd_undo`, `_cmd_save`, etc.
- The shell keeps a single `GameState` instance as `self.state` and delegates all rule checking and state changes to it.
- `render_board()` and `render_status()` in `renderers.py` are responsible for converting a `Board` into ASCII art and summary text.

### Model Layer (`src/model`)

- `GameState` tracks:
  - player names and current player (`PlayerSide`),
  - a `Board` instance with piece placement and square types,
  - move history and undo credits per player,
  - winner (if the game has finished).
- `Board` implements the Jungle rules, including
  - legal move generation and validation,
  - capture rules (rank comparison, trap effects, river rules),
  - win conditions (entering opponent den or eliminating all enemy pieces).
- `serialization.py` provides `save_game`, `load_game`, `export_record`, and `load_record` using JSON files; it raises `SerializationError` if a file is missing or malformed.

## Extending the Game

When adding new features, follow these guidelines:

- **Keep rules in the model**: if a feature changes how moves are validated, or introduces new win conditions, implement it in `board.py` / `game_state.py` and add tests in `tests/test_model.py`.
- **Keep UI in the CLI**: new commands or output formats belong in `src/cli`.
- **Update help text**: if you add a new command, update `_cmd_help` in `JungleShell` and the User Manual.
- **Persist new state**: if `GameState` gains new attributes that must be saved, update `serialization.py` so they round‑trip through save/load and records.

### Example: Adding a "hint" Command

1. Implement a helper in the model (e.g., `GameState.legal_moves_for_current_player()`).
2. Expose it via a new command handler `_cmd_hint` in `JungleShell` and register it in `self._commands`.
3. Print one or more suggested moves based on the returned list.
4. Add/extend tests to cover the helper and any edge cases.

## Troubleshooting

- **`ModuleNotFoundError` for `model` or `cli`**:
  - Ensure you are in the repository root and use `python -m src.main` instead of `python src/main.py`.
- **Game exits immediately in VS Code**:
  - Confirm the debug configuration uses `"module": "src.main"` and `"console": "integratedTerminal"`.
- **Undo fails**:
  - Each side has a limited number of undo credits (tracked in `GameState.undo_remaining`).
  - Use `undo blue` or `undo red` if you need to specify which player is requesting the undo.
- **Save / load errors**:
  - Check the file path and extension; `ensure_extension()` in `utils.py` automatically appends `.jungle` or `.record` if omitted.
  - Corrupted or hand‑edited JSON will raise `SerializationError` with a descriptive message.

## Contribution Workflow

1. Create a feature or bug‑fix branch.
2. Modify or add tests in `tests/` to describe the desired behaviour.
3. Implement changes in `src/model` and/or `src/cli` following the separation‑of‑concerns guidelines.
4. Run `python -m unittest discover -s tests` and, if available, coverage.
5. Update relevant documentation (`docs/DesignDocument,` `docs/RequirementsCoverage `and the manuals) if behaviour or commands changed.
6. Submit the branch for review or integration.

This completes the developer‑oriented overview needed to build, run, debug, and extend the Jungle CLI on Windows with Python 3.12.
