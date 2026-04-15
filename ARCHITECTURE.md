# Architecture Documentation

## High-Level Overview

The Codenames Telegram Bot is built with a professional, scalable architecture that separates concerns into distinct modules:

```
┌─────────────────────────────────────────┐
│         main.py (Entry Point)           │
│    Initializes App & Registers Handlers │
└────────────────────┬────────────────────┘
                     │
        ┌────────────┴────────────┐
        │                         │
┌───────▼──────────┐    ┌────────▼───────┐
│    handlers.py   │    │   config.py    │
│  (All Commands)  │    │  (Constants &  │
│                  │    │   Config)      │
└───────┬──────────┘    └────────────────┘
        │
        ├──────────┬──────────┬──────────┐
        │          │          │          │
┌───────▼──┐ ┌─────▼──┐ ┌──────▼─┐ ┌────▼────┐
│ game.py  │ │board.py│ │utils.py│ │images.py│
│  (Game   │ │(Board  │ │(Helper │ │ (Media) │
│  Logic)  │ │  Mgmt) │ │Functions)│ │        │
└──────────┘ └────────┘ └────────┘ └────────┘
```

## Module Responsibilities

### 1. main.py (Root Entry Point)
**Location**: `/main.py`

**Responsibilities**:
- Application startup
- Path setup and imports
- Called via `python main.py`

**Key Functions**:
- `main()`: Entry point for the application

---

### 2. src/main.py (Application Builder)
**Location**: `/src/main.py`

**Responsibilities**:
- Telegram bot initialization
- Handler registration
- Application configuration
- Polling setup

**Key Functions**:
- `setup_logging()`: Configure logging
- `build_application()`: Create and configure bot
- `main()`: Run the application

---

### 3. src/config.py (Configuration)
**Location**: `/src/config.py`

**Responsibilities**:
- Environment variable loading
- Constants definition
- Configuration validation
- Logging setup

**Key Components**:
- Environment variables: `TELEGRAM_TOKEN`, `GEMINI_API_KEY`, `MONGODB_URL`
- Game config: `MIN_PLAYERS`, `MAX_PLAYERS`, `GAME_BOARD_SIZE`
- Shop system: `SHOP_ITEMS`, `GAME_DEFAULT_REWARD`
- Resources: `VICTORY_IMAGES`, `LOCAL_IMAGES`, `HINDI_BLACKLIST`

---

### 4. src/game.py (Game State Management)
**Location**: `/src/game.py`

**Responsibilities**:
- Game state management
- Player management
- Team assignment and management
- Board initialization and management
- Turn management
- Win condition checking

**Key Classes**:
- `Player`: Represents a player
- `GameState`: Complete game state
- `Game`: Main game logic class

**Key Methods**:
```python
# Player management
add_player(user) → bool
remove_player(user_id) → bool
get_player_count() → int
is_player_in_game(user_id) → bool

# Team management
assign_teams() → None
change_team(user_id) → bool
set_spymaster(team, user_id) → bool
get_team_for_player(user_id) → Optional[str]

# Board management
initialize_board() → None
reveal_word(word) → bool
get_word_team(word) → Optional[str]
get_remaining_words_for_team(team) → int
check_win_condition() → Optional[str]

# Turn management
set_clue(clue, count) → None
clear_clue() → None
end_turn() → None
```

---

### 5. src/board.py (Board Management)
**Location**: `/src/board.py`

**Responsibilities**:
- Board generation
- Word assignment to teams
- Board rendering (public and spymaster views)
- Board state management

**Key Classes**:
- `BoardWord`: Single word on board
- `Board`: Board management class

**Key Methods**:
```python
generate() → None              # Generate new random board
reveal_word(word) → bool       # Reveal a word
get_word_team(word) → Optional[str]
get_remaining_words(team) → int
render_public() → str          # Public board view
render_spymaster(user_id, game) → str  # Spymaster view
```

---

### 6. src/handlers.py (Telegram Handlers)
**Location**: `/src/handlers.py`

**Responsibilities**:
- Command handling (telegram commands)
- Message filtering
- Game logic execution (via game.py)
- User interaction

**Key Handlers**:

**Game Lifecycle**:
- `startgame()`: Initialize game
- `join()`: Player joins
- `players()`: List players
- `start()`: Start game
- `cancel()`: Cancel game

**Gameplay**:
- `clue()`: Spymaster gives clue
- `guess()`: Player makes guess
- `endturn()`: End turn
- `execute_guess_logic()`: Process guess

**Filters**:
- `spymaster_mute_filter()`: Prevent spymaster chatting
- `handle_stickers()`: Remove stickers during game

**Help**:
- `help_command()`: Display help

---

### 7. src/utils.py (Utilities)
**Location**: `/src/utils.py`

**Responsibilities**:
- Common helper functions
- Word generation
- Input validation
- User formatting
- Game logic helpers
- Logging utilities

**Key Functions**:
```python
# Initialization
initialize_nltk_data() → None

# Name formatting
get_user_name(user) → str
capitalize_words(text) → str

# Word generation
get_easy_words(n) → List[str]
generate_fallback_words(n) → List[str]

# Validation
is_valid_clue(word, blacklist) → bool
is_valid_guess_count(count) → bool

# Message formatting
format_html_message(title, content, footer) → str

# Game helpers
switch_team(current_team) → str
get_safe_user_id(user_or_id) → int
get_team_emoji(team) → str
get_role_emoji(role) → str

# Logging
log_game_event(chat_id, event, details) → None

# Time formatting
format_time(seconds) → str
```

---

### 8. src/images.py (Media Handling)
**Location**: `/src/images.py`

**Responsibilities**:
- Victory image sending
- Meme/response image sending
- Error handling for image operations

**Key Functions**:
```python
send_winner_image(context, chat_id, winner_team) → bool
send_fak_image(context, chat_id, reply_to_id) → bool
send_lmao_image(context, chat_id, reply_to_id) → bool
send_suicide_image(context, chat_id, reply_to_id) → bool
```

---

## Data Flow

### Game Initialization Flow
```
1. User calls /startgame
2. Handler creates Game instance
3. Game instance stored in games dict
4. Channel opened for players to join
```

### Player Join Flow
```
1. User calls /join
2. Handler verifies game exists
3. Handler checks DM connectivity
4. Game.add_player() called
5. Player added to game.state.players
```

### Game Start Flow
```
1. Creator calls /start
2. Permission check
3. Game.start()
   - assign_teams()
   - initialize_board()
   - set active=True
4. Teams announced
5. Timer started (if enabled)
```

### Guess Flow
```
1. Player calls /guess <word>
2. Handler validates
3. execute_guess_logic()
4. game.reveal_word()
5. Check win condition
6. Update turn or end game
```

## Global State Management

### games Dictionary
```python
games: Dict[int, Game] = {}

# Key: chat_id
# Value: Game instance
```

### Game Instance Structure
```python
Game:
  .chat_id: int
  .state: GameState
    .chat_id: int
    .creator_id: int
    .players: Dict[int, User]
    .teams: Dict[str, List[User]]
    .spymasters: Dict[str, User]
    .board: Dict[str, Dict]
    .turn: str
    .clue: Optional[str]
    .left: int
    .active: bool
    .started: bool
    # ... other fields
```

## Import Structure

### Root main.py
```python
import sys
from src.main import main

if __name__ == "__main__":
    main()
```

### src/main.py
```python
from config import TELEGRAM_TOKEN, logger
from handlers import startgame, join, ...
```

### src/handlers.py
```python
from config import MIN_PLAYERS, MAX_PLAYERS, ...
from game import Game
from utils import get_user_name, ...
from images import send_winner_image, ...
```

### src/game.py
```python
from board import Board, generate_board
from utils import switch_team, get_safe_user_id
```

## Type Hints

All functions and classes use full type hints:

```python
async def startgame(update: Update, context: ContextTypes.DEFAULT_TYPE) -> None:
    """Docstring with summary and parameter descriptions"""
    pass

def is_valid_clue(word: str, blacklist: List[str]) -> bool:
    """Function with type hints"""
    pass

class Game:
    """Class with typed methods"""
    def set_clue(self, clue: str, count: int) -> None:
        pass
```

## Error Handling Strategy

1. **Try-Except Blocks**: All user-facing operations wrapped
2. **User Messages**: User-friendly error messages
3. **Logging**: All errors logged for debugging
4. **Fallbacks**: Graceful degradation

Example:
```python
try:
    # Operation
except FileNotFoundError:
    logger.warning("File not found")
    # Send fallback message
except Exception as e:
    logger.error(f"Unexpected error: {e}")
    # Send generic error message
```

## Concurrency & Async

- **Full async/await**: All handlers are async
- **Timer Management**: Non-blocking timer via `asyncio.create_task()`
- **No Global Async State**: Each game manages its own timers
- **Safe Multi-game**: Each chat_id has its own Game instance

## Security Considerations

1. **Input Validation**: All clues and guesses validated
2. **Permission Checks**: Creator-only and spymaster-only commands protected
3. **DM Verification**: Players must have DM enabled
4. **No Global Variables**: Game state encapsulated
5. **Clean Message Handling**: Deleted when appropriate

## Scalability Features

1. **Per-Chat Isolation**: Each game is independent
2. **Modular Design**: Each module has single responsibility
3. **Type Safety**: Full type hints enable IDE optimization
4. **Logging**: Extensive logging for monitoring
5. **Async I/O**: Non-blocking operations
6. **Optional Persistence**: MongoDB support (when configured)

## Testing Strategy

### Unit Testing
- Individual functions with mocked external dependencies
- Class methods with test fixtures

### Integration Testing
- Handler functions with mock Update/Context objects
- Game flow simulations

### Load Testing
- Multiple concurrent games
- High message volume handling

## Deployment Considerations

1. **Docker**: Self-contained deployment
2. **Environment Variables**: Configuration via .env
3. **Logging**: Configurable log level
4. **Health Checks**: Optional health endpoints
5. **Auto-restart**: Systemd or Docker restart policies

## Future Extensions

1. **Persistent Leaderboards**: MongoDB integration
2. **Difficulty Levels**: Easy/Normal/Hard modes
3. **Custom Word Sets**: User-defined word lists
4. **Team Voice Chat**: Integration with VoIP
5. **Web Dashboard**: Stats and game monitoring
6. **AI Spymaster**: Machine learning clue generation
7. **Game Replays**: Save and replay past games
8. **Multiplayer Features**: Cross-chat tournaments
