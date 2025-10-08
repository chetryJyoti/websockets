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
**WebSocket server that coordinates multiplayer games**

- Listens on `ws://localhost:8001`
- Supports multiple concurrent games with unique tokens
- Routes connections to start/join/watch handlers
- Broadcasts moves to all connected clients in a game
- Manages player connections and cleanup

**Key Functions:**
- `handler(websocket)` - Main connection handler, routes based on init event
- `start(websocket)` - Creates new game for Player 1, generates join/watch tokens
- `join(websocket, join_key)` - Adds Player 2 to existing game
- `watch(websocket, watch_key)` - Adds spectator to game
- `play(websocket, game, player, connected)` - Handles move events and broadcasts
- `replay(websocket, game)` - Sends previous moves to new connections
- `error(websocket, message)` - Sends error messages

**Global State:**
- `JOIN` - Dictionary mapping join tokens to games
- `WATCH` - Dictionary mapping watch tokens to games

**Message Types:**
- **Receives**:
  - Init: `{"type": "init"}` or `{"type": "init", "join": "token"}` or `{"type": "init", "watch": "token"}`
  - Play: `{"type": "play", "column": 0-6}`
- **Sends**:
  - Init: `{"type": "init", "join": "token", "watch": "token"}`
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

- Contains action buttons: New, Join, Watch
- Contains a `<div class="board">` container
- Loads `main.js` as an ES6 module
- JavaScript dynamically builds the game grid and updates link hrefs

#### `src/main.js`
**WebSocket client controller**

- Establishes WebSocket connection to server
- Coordinates between UI events and server communication
- Handles incoming messages and updates the UI accordingly
- Manages game modes (start/join/watch)

**Key Functions:**
- `initGame(websocket)` - Sends init event with URL parameters (join/watch tokens)
- `sendMoves(board, websocket)` - Listens for clicks, sends play events (disabled for spectators)
- `receiveMoves(board, websocket)` - Processes server messages and updates join/watch links
- `showMessage(message)` - Displays alerts for game events

**Event Flow:**
1. WebSocket opens → Send init event (with join/watch token if present)
2. Receive init response → Update join/watch link hrefs
3. User clicks a column (if not spectating)
4. Send column number to server
5. Receive validated move with row position
6. Update UI with the piece placement
7. Show winner alert if game ends

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

1. **Start the WebSocket server**
   ```bash
   cd src
   python app.py
   ```
   This starts on `ws://localhost:8001`

2. **Open the game in your browser**
   - Open `src/index.html` directly in your browser
   - Or use a local HTTP server:
     ```bash
     python -m http.server
     # Then visit http://localhost:8000/src/index.html
     ```

3. **Start a new game**
   - Click the **New** button to start as Player 1 (red)
   - The Join and Watch links will appear

4. **Add a second player**
   - Copy the **Join** link URL
   - Open it in another browser window/tab or share with another player
   - Player 2 (yellow) can now play

5. **Add spectators** (optional)
   - Copy the **Watch** link URL
   - Spectators can see all moves but cannot play

## 🎯 How It Works

### Complete Game Flow

**Player 1 (Starting a game):**
1. **Browser loads** → `index.html` loads `main.js`
2. **UI initialization** → `createBoard()` builds the grid
3. **WebSocket connection** → Connects to `ws://localhost:8001`
4. **Send init event** → `{"type": "init"}` (no join/watch token)
5. **Server creates game** → Generates unique join and watch tokens
6. **Receive init response** → Join and Watch links become active
7. **Make moves** → Click columns to play as red

**Player 2 (Joining a game):**
1. **Click Join link** → Opens page with `?join=token` in URL
2. **WebSocket connects** → Sends `{"type": "init", "join": "token"}`
3. **Server finds game** → Adds Player 2 to existing game
4. **Replay moves** → Receives all previous moves to sync board state
5. **Make moves** → Click columns to play as yellow

**Spectator (Watching a game):**
1. **Click Watch link** → Opens page with `?watch=token` in URL
2. **WebSocket connects** → Sends `{"type": "init", "watch": "token"}`
3. **Server finds game** → Adds spectator to broadcast list
4. **Replay moves** → Receives all previous moves
5. **View only** → Cannot make moves, only watch

**During Gameplay:**
1. **User clicks column** → JavaScript sends play event
2. **Server validates** → Checks turn order and column availability
3. **Server broadcasts** → Sends move to all connected clients (both players + spectators)
4. **All UIs update** → Piece appears in the correct cell on all screens
5. **Turn switches** → Next player's turn begins
6. **Game ends** → Server detects winner and broadcasts win event to all

### Key Technologies

- **WebSockets** - Bidirectional real-time communication
- **Python asyncio** - Asynchronous server handling
- **Broadcasting** - `websockets.asyncio.server.broadcast()` for multiplayer sync
- **URL Parameters** - Query strings for join/watch links (`?join=token`)
- **Secret Tokens** - `secrets.token_urlsafe()` for secure game access
- **ES6 Modules** - Modern JavaScript structure
- **Bitwise Operations** - Efficient win detection algorithm

### Turn Management

Players alternate automatically:
- Red (PLAYER1) always goes first
- Game logic tracks `last_player` and enforces turn order
- Invalid moves don't advance the turn
- Error sent if player tries to move out of turn

### Win Detection

Uses an optimized bitwise algorithm:
- Each board position maps to a bit in an integer
- Checks 4 directions: horizontal, vertical, and both diagonals
- Runs in constant time regardless of board size

## 🔍 Current Features

- ✅ Real-time multiplayer gameplay with WebSocket communication
- ✅ **Multi-browser support** - Players can join from different browsers/devices
- ✅ **Spectator mode** - Watch ongoing games without participating
- ✅ **Game broadcasting** - All connected clients see moves instantly
- ✅ **Secure game sessions** - Unique join and watch tokens for each game
- ✅ **Move replay** - Players joining mid-game see all previous moves
- ✅ Turn-based validation
- ✅ Automatic win detection
- ✅ Error handling for invalid moves
- ✅ Connection management with automatic cleanup

## 📝 Limitations

- No game reset button - click "New" to start fresh
- No draw/tie detection
- Join/watch links expire when host disconnects
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
