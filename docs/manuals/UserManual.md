# Jungle CLI – User Manual

This manual explains how to start the Jungle (Dou Shou Qi) command‑line game, what commands are available, how to interpret the output, and the rules you must follow to play effectively.

## 1. Starting the Game

1. Open a terminal (e.g., Windows PowerShell).
2. Change into the project folder (the one containing `pyproject.toml` and `src/`).
3. Run the following command:

   ```powershell
   python -m src.main
   ```

You should see:

```text
Welcome to Jungle! Type 'help' to see available commands.
jungle>
```

The `jungle>` prompt indicates that the game is ready to accept commands.

At any time you can type:

```text
help
```

to see the full list of supported commands.

## 2. Board, Coordinates, and Symbols

The Jungle board is a 7×9 grid. Squares are identified using a letter for the **column** and a number for the **row**:

- Columns: `a`–`g` (left to right).
- Rows: `1`–`9` (top to bottom).

Examples:

- `a1` – top‑left square.
- `g9` – bottom‑right square.

### 2.1 Terrain Symbols

Special terrain symbols printed on the board:

- `..` – normal land square.
- `~~` – river.
- `DB` – den of the **blue** player.
- `DR` – den of the **red** player.
- `tb` – trap belonging to blue (near blue’s den).
- `tr` – trap belonging to red.

### 2.2 Piece Symbols (Legend)

Each piece is displayed using a **two‑letter code**. The code is the same for both players, but **blue pieces are printed in UPPERCASE** and **red pieces in lowercase**. For example, `El` is a blue elephant and `el` is a red elephant.

Piece codes are:

| Code   | Piece    | Rank | Notes                           |
| ------ | -------- | ---- | ------------------------------- |
| `Ra` | Rat      | 1    | Can enter rivers; special vs El |
| `Ca` | Cat      | 2    | Land piece                      |
| `Do` | Dog      | 3    | Land piece                      |
| `Wo` | Wolf     | 4    | Land piece                      |
| `Le` | Leopard  | 5    | Land piece                      |
| `Ti` | Tiger    | 6    | Can jump over rivers            |
| `Li` | Lion     | 7    | Can jump over rivers            |
| `El` | Elephant | 8    | Strongest piece, except vs Rat  |

Examples:

- `RA` – blue rat; `ra` – red rat.
- `LI` – blue lion; `li` – red lion.
- `EL` – blue elephant; `el` – red elephant.

If a square shows `..` or one of the terrain codes, it is empty of pieces; otherwise, it shows exactly one piece symbol as above.

## 3. Overview of Commands

All commands are typed at the `jungle>` prompt. Commands are **case‑insensitive**, but file names and player names keep the case you type.

### 3.1 Game Setup and Information

- `help`

  - Shows a summary of all available commands and their usage.
- `new [blue] [red]`

  - Starts a **new game**.
  - Optionally specify the names of the blue and red players.
  - If you do not provide names, the game will generate random names.
  - Examples:
    - `new` – start with two random names.
    - `new Alice Bob` – blue is Alice, red is Bob.
- `players <blue|red> <name>`

  - Changes the name of one player during a game.
  - Example: `players blue Alice`.
- `show`

  - Prints **only** the current board with the coordinate axes.
- `status`

  - Prints:
    - the current board,
    - whose turn it is (player name and side),
    - remaining undo credits for each side,
    - the winner, if the game is already finished,
    - a list of remaining pieces for each side.

### 3.2 Playing Moves and History

- `move <from> <to>`

  - Moves one of your pieces from a starting square to a destination square.
  - Both arguments must be valid coordinates, such as `a3` or `d7`.
  - The move must follow the Jungle rules (see Section 4).
  - Examples:
    - `move b3 b4`
    - `move c2 d2`
- `history [n]`

  - Shows the **most recent moves**.
  - If `n` is provided, shows the last `n` moves; otherwise shows the last 5.
  - Output lines are numbered and include which piece moved and any capture.
  - Example: `history 10`.
- `undo [side]`

  - Undoes the **most recent move**, subject to the available undo credits.
  - Optional `side` is `blue` or `red` to specify which player is requesting the undo.
  - If no side is given, the current player is assumed.
  - After a successful undo, the program prints the updated board, turn information, remaining undo credits, and a message indicating how many credits the chosen side has left.
  - Example: `undo blue`.

### 3.3 Saving, Loading, and Replaying Games

- `save-game <file.jungle>`

  - Saves the **entire current game** (players, board, move log, and undo credits) to a JSON‑based file.
  - If you forget the `.jungle` extension, it is automatically added.
  - Example: `save-game my_match` (file becomes `my_match.jungle`).
- `load-game <file.jungle>`

  - Loads a previously saved `.jungle` file and makes it the current game.
  - If the file does not exist or is not a valid save file, an error message is printed.
  - Example: `load-game my_match.jungle`.
- `export-record <file.record>`

  - Exports a **record of all moves** played in the current game so far.
  - This is useful for archiving or replaying games.
  - If no moves have been played, the command fails with a helpful error message.
  - Example: `export-record demo_game` (saved as `demo_game.record`).
- `replay-record <file.record>`

  - Replays a recorded game from a `.record` file.
  - The program prints:
    - when the record was created,
    - the players’ names,
    - each move number, side, and source/target,
    - the board after each move.
  - If the record contains an illegal move (for example, from a corrupted file), replay stops and an error is shown for that move.

### 3.4 Quitting

- `quit` or `exit`
  - Cleanly ends the program and returns you to the terminal.

## 4. Game Rules

This section summarizes the Jungle rules as implemented by the program.

### 4.1 Objective

You win the game by **either**:

1. Moving one of your pieces into the **opponent’s den**, or
2. Capturing **all** of the opponent’s pieces so that none remain on the board.

### 4.2 Piece Ranks

Each piece has a rank (strength). Higher‑ranked pieces normally capture lower‑ranked pieces, with special exceptions involving rats and elephants.

Typical ranks (from weakest to strongest) are:

1. Rat
2. Cat
3. Dog
4. Wolf
5. Leopard
6. Tiger
7. Lion
8. Elephant

When two pieces of different ranks fight on the same square, the **higher rank** wins, except for special rat rules described below.

### 4.3 Movement

- Most pieces move **one square at a time**, horizontally or vertically (no diagonals).
- Pieces cannot move through other pieces.
- Pieces cannot move onto squares occupied by their own pieces.

#### Rivers

- Only **rats** may enter river squares (`~~`).
- All other pieces treat river squares as blocked.

#### Lion and Tiger Jumps

- **Lions** and **tigers** can jump over the river horizontally or vertically.
- The path **across the river must be clear of rats** in all intermediate river squares.
- They land on the first land square across the river; that landing square may be empty or hold an enemy piece (which can then be captured if allowed by the ranks and trap rules).

### 4.4 Capturing

- You capture an opposing piece by moving one of your pieces onto the square occupied by the enemy piece, subject to rank and terrain rules.
- Normally, higher‑ranked pieces capture lower‑ranked pieces.

#### Rats and Elephants

- A **rat** is allowed to capture an **elephant**.
- Extra restrictions exist regarding land vs. water:
  - A rat in the **water** cannot be captured by an elephant on **land**.
  - Land rats cannot attack rats that are in the water, and vice versa, where applicable by the implemented rules.

#### Traps

- Each side has traps surrounding their den (`tb` for blue, `tr` for red).
- An enemy piece that moves into **your trap** becomes temporarily weakened to **rank 0**.
- While in a trap, that piece can be captured by **any** of your pieces, regardless of normal rank.

### 4.5 Dens

- `DB` is the den belonging to the blue player; `DR` is the red den.
- You **cannot move into your own den**.
- If you move one of your pieces into the **opponent’s den**, you immediately **win** the game.

### 4.6 Undo Credits

- Each side starts with a limited number of **undo credits** (tracked internally).
- When you use `undo` on behalf of a side:
  - The most recent move is reverted.
  - One undo credit is consumed for that side.
- When a side has no credits remaining, further undo attempts for that side will fail.

## 5. Invalid Commands and Errors

The program is designed to be robust against mistakes:

- If you type an **unknown command** (e.g., `mvoe`), you see an error such as:

  ```text
  Unknown command 'mvoe'. Type 'help' for a list of commands.
  ```
- If a command has the wrong **number of arguments**, you see a usage message, for example:

  ```text
  Error: Usage: move <from> <to>
  ```
- If a coordinate is invalid (e.g., `z10`), the input parser raises an error and the game prints a message like:

  ```text
  Invalid input: ...
  ```
- If a **move is illegal** under the rules (for example, a non‑rat piece entering the river), the program prints a detailed message such as:

  ```text
  Move rejected: Only rats may enter the river.
  ```

  and the board state does **not** change.
- If a file cannot be loaded or parsed when using `load-game` or `replay-record`, you see a message like:

  ```text
  File error: <details>
  ```

In all of these cases, the game continues to run, and you can correct your command.

## 6. Example Play Session

Below is a short sample session showing common commands and the expected flow.

```text
python -m src.main
Welcome to Jungle! Type 'help' to see available commands.
jungle> new Alice Bob
New game created.
... (board and status shown) ...
jungle> move a3 a4
Moved RAT from a3 to a4
... (updated status) ...
jungle> move g7 g6
Moved LION from g7 to g6
jungle> history 2
#1: BLUE moved RAT a3->a4
#2: RED  moved LION g7->g6
jungle> undo blue
Last move undone. Undo credits left for BLUE: 2
... (updated status) ...
jungle> save-game demo
Game saved to demo.jungle.
jungle> export-record demo
Record exported to demo.record.
jungle> quit
Goodbye!
```

With these commands and rules, you have everything you need to play Jungle effectively using the command‑line interface.
