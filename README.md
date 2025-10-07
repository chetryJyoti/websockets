# Connect Four WebSocket Game

A real-time multiplayer Connect Four game built with Python WebSockets and vanilla JavaScript.

## 🎮 About the Game

Connect Four is a two-player strategy game where players take turns dropping colored discs into a 7×6 vertical grid. The objective is to connect four of your pieces horizontally, vertically, or diagonally before your opponent does.

- **Players**: Red vs Yellow
- **Board**: 7 columns × 6 rows
- **Win Condition**: 4 pieces in a row (any direction)
- **Gameplay**: Pieces fall to the lowest available position in the selected column

## 🏗️ Architecture

This project uses WebSockets for real-time bidirectional communication between the browser client and Python server.

```
Browser (JavaScript) ←─── WebSocket ───→ Python Server
        ↓                                      ↓
    UI Rendering                         Game Logic
```

## 📁 Project Structure

```
websockets/
├── src/
│   ├── app.py           # WebSocket server
│   ├── connect4.py      # Game logic (Python)
│   ├── index.html       # Entry point
│   ├── main.js          # WebSocket client controller
│   ├── connect4.js      # UI rendering
│   └── connect4.css     # Board styling
├── requirements.txt     # Python dependencies
└── README.md
```

## 📄 File Descriptions

### Backend (Python)

#### `src/app.py`
**WebSocket server that coordinates the game**

- Listens on `ws://localhost:8001`
- Creates a new game instance for each connection
- Receives player moves from the client
- Validates moves using game logic
- Broadcasts results back to the client
- Manages turn alternation using `itertools.cycle`
- Includes debug logging for all messages

**Key Functions:**
- `handler(websocket)` - Handles WebSocket connections and game flow
- `main()` - Starts the WebSocket server

**Message Types:**
- **Receives**: `{"type": "play", "column": 0-6}`
- **Sends**:
  - Play event: `{"type": "play", "player": "red/yellow", "column": N, "row": N}`
  - Error event: `{"type": "error", "message": "..."}`
  - Win event: `{"type": "win", "player": "red/yellow"}`

#### `src/connect4.py`
**Core game logic and rules**

- Defines player constants: `PLAYER1 = "red"`, `PLAYER2 = "yellow"`
- Manages game state (board, moves, winner)
- Validates moves (turn order, column availability)
- Detects winning conditions using bitwise operations

**Connect4 Class:**
- `__init__()` - Initializes empty board and game state
- `play(player, column)` - Executes a move, returns row position
- `last_player` - Property that tracks who played last
- `last_player_won` - Property that detects 4-in-a-row using bit manipulation

**Game State:**
- `moves` - List of all moves as `(player, column, row)` tuples
- `top` - Array tracking next available row for each column
- `winner` - Stores the winning player or `None`

### Frontend (JavaScript)

#### `src/index.html`
**Minimal HTML entry point**

- Contains a `<div class="board">` container
- Loads `main.js` as an ES6 module
- JavaScript dynamically builds the game grid

#### `src/main.js`
**WebSocket client controller**

- Establishes WebSocket connection to server
- Coordinates between UI events and server communication
- Handles incoming messages and updates the UI accordingly

**Key Functions:**
- `sendMoves(board, websocket)` - Listens for clicks, sends play events to server
- `receiveMoves(board, websocket)` - Processes server messages (play, win, error)
- `showMessage(message)` - Displays alerts for game events

**Event Flow:**
1. User clicks a column
2. Sends column number to server
3. Receives validated move with row position
4. Updates UI with the piece placement
5. Shows winner alert if game ends

#### `src/connect4.js`
**UI rendering and board management**

- Dynamically creates the 7×6 game grid
- Handles visual representation of moves
- Validates move rendering

**Key Functions:**
- `createBoard(board)` - Generates HTML grid structure and injects CSS
- `playMove(board, player, column, row)` - Updates cell class to show colored piece

**Visual Updates:**
- Cells start with class `"empty"`
- Playing a move replaces class with `"red"` or `"yellow"`
- CSS handles the visual appearance

## 🚀 How to Run

### Prerequisites
- Python 3.7+
- Modern web browser with WebSocket support

### Installation

1. **Clone/navigate to project directory**
   ```bash
   cd websockets
   ```

2. **Create virtual environment** (optional but recommended)
   ```bash
   python -m venv venv
   source venv/bin/activate  # On Windows: venv\Scripts\activate
   ```

3. **Install dependencies**
   ```bash
   pip install -r requirements.txt
   ```

### Running the Game

1. **Start the HTTP server** (serves frontend files)
   ```bash
   python -m http.server
   ```
   This starts on `http://localhost:8000`

2. **Start the WebSocket server** (in a new terminal)
   ```bash
   python src/app.py
   ```
   This starts on `ws://localhost:8001`

3. **Open the game in your browser**
   ```
   http://localhost:8000/src/index.html
   ```

### Alternative: CLI Connection
You can also connect via command line:
```bash
websockets ws://localhost:8001/
```

## 🎯 How It Works

### Complete Game Flow

1. **Browser loads** → `index.html` loads `main.js`
2. **UI initialization** → `createBoard()` builds the grid
3. **WebSocket connection** → Connects to `ws://localhost:8001`
4. **User clicks column** → JavaScript sends play event
5. **Server validates** → Checks turn order and column availability
6. **Server responds** → Sends back row position or error
7. **UI updates** → Piece appears in the correct cell
8. **Turn switches** → Next player's turn begins
9. **Game ends** → Server detects winner and sends win event

### Key Technologies

- **WebSockets** - Bidirectional real-time communication
- **Python asyncio** - Asynchronous server handling
- **ES6 Modules** - Modern JavaScript structure
- **Bitwise Operations** - Efficient win detection algorithm

### Turn Management

Players alternate automatically:
- Red (PLAYER1) always goes first
- Server uses `itertools.cycle([red, yellow])` for infinite turn rotation
- Invalid moves don't advance the turn

### Win Detection

Uses an optimized bitwise algorithm:
- Each board position maps to a bit in an integer
- Checks 4 directions: horizontal, vertical, and both diagonals
- Runs in constant time regardless of board size

## 🔍 Current Features

✅ Real-time gameplay with WebSocket communication
✅ Turn-based validation
✅ Automatic win detection
✅ Error handling for invalid moves
✅ Debug logging for development
✅ Same-browser multiplayer

## 📝 Limitations

- Both players share the same browser (no separate clients)
- No game reset button (must refresh page)
- No draw/tie detection
- One game per WebSocket connection
- No player selection (red always starts)

## 🛠️ Development Notes

### Important Reminders

- **Server restart required**: Changes to Python files require restarting `python src/app.py`
- **Frontend updates**: JavaScript/HTML changes are visible after browser refresh (no server restart needed)
- **HTTP server**: Required to serve static files; browser can't load local files directly due to CORS

### Debug Logging

The server logs all WebSocket messages:
```
2025-10-07 23:10:15 Received: {'type': 'play', 'column': 3}
2025-10-07 23:10:15 Sent: {'type': 'play', 'player': 'red', 'column': 3, 'row': 0}
```

## 📚 Resources

- [WebSockets Documentation](https://websockets.readthedocs.io/en/stable/)
- Python `websockets` library: v15.0.1

## 🎮 Gameplay Tips

1. Red always moves first
2. Click any column to drop a piece
3. Pieces fall to the lowest available spot
4. Try to block your opponent while building your own line
5. Watch for diagonal opportunities!

---

**Enjoy the game!** 🎉
