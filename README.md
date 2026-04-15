# Codenames Telegram Bot

A professional, production-grade Telegram bot implementation of the Codenames word game. Supports multiple concurrent games with team-based gameplay, spymaster mechanics, and advanced game features.

## 📋 Features

- **Team-Based Gameplay**: BJP vs Congress teams with independent games per chat
- **Spymaster Mechanics**: Secret board visibility for spymasters only
- **Vote Systems**: Democratic team decisions on guesses and surrenders
- **Game Economy**: Coin rewards, shop system, and inventory management
- **Safety**: Multi-chat support, proper async handling, error recovery
- **Scalability**: Designed for production deployment with logging and monitoring

## 🏗️ Architecture

The project follows a clean, modular architecture:

```
src/
├── __init__.py          # Package initialization
├── main.py              # Application entry point
├── config.py            # Configuration and constants
├── utils.py             # Shared utility functions
├── game.py              # Core game logic and state
├── board.py             # Board generation and rendering
├── handlers.py          # Telegram command handlers
└── images.py            # Media handling (images, memes)

main.py                 # Root entry point
requirements.txt        # Python dependencies
README.md              # This file
```

### Module Breakdown

- **config.py**: Centralized configuration, environment variables, constants
- **utils.py**: Helper functions (formatting, validation, word generation)
- **game.py**: Game class with full state management
- **board.py**: Board class for word management and rendering
- **handlers.py**: All Telegram handlers and command logic
- **images.py**: Message media (victory images, response pictures)
- **main.py**: Application setup and handler registration

## 🎮 Game Rules

1. **Teams**: Two teams (BJP and Congress) compete to find all their words first
2. **Board**: 25 words on the board
   - 8 words per team (team color)
   - 8 neutral words
   - 1 assassin word (instant loss)
3. **Spymaster**: Gives one-word clues and a number
4. **Guesses**: Team members guess words based on clues
5. **Win Condition**: First team to reveal all their words wins

## 🚀 Installation

### Prerequisites
- Python 3.8+
- Telegram Bot Token (from BotFather)
- MongoDB Atlas account (for stats/leaderboard)
- (Optional) Google Gemini API key

### Setup

1. **Clone or download the repository**

2. **Install dependencies**
   ```bash
   pip install -r requirements.txt
   ```

3. **Set environment variables**
   ```bash
   export TOKEN="your_telegram_bot_token"
   export GEMINI_API_KEY="your_gemini_api_key"
   export MONGODB_URL="your_mongodb_connection_string"
   export LOG_LEVEL="INFO"
   ```

4. **Run the bot**
   ```bash
   python main.py
   ```

## 📝 Usage

### Commands

#### Game Setup
- `/startgame` - Initialize a new game lobby
- `/join` - Join the current game
- `/players` - View current player roster
- `/start` - Begin the game (creator only)
- `/team` - View team assignments

#### Gameplay
- `/clue <word> <number>` - Give a clue (Spymaster only, DM)
- `/guess <word>` - Make a guess
- `/endturn` - End your team's turn
- `/hint` - Show current game status

#### Admin
- `/cancel` - Cancel the game (creator only)
- `/shuffle` - Reshuffle teams (creator only)
- `/changespy <team> <user>` - Change spymaster
- `/changeteam` - Switch to other team

#### Economy
- `/profile` - View personal statistics
- `/shop` - View available items
- `/buy <item_id>` - Purchase item
- `/inventory` - View owned items
- `/use <item_id>` - Use item during game
- `/leaderboard` - View top players

#### Help
- `/help` - Show commands help
- `/guide` - Game rules and instructions

## 🔧 Configuration

Edit `src/config.py` to customize:

- Game limits (`MIN_PLAYERS`, `MAX_PLAYERS`)
- Board size (`GAME_BOARD_SIZE`)
- Timer defaults (`DEFAULT_SPYMASTER_TIMER`, `DEFAULT_PLAYER_TIMER`)
- Shop items and prices
- Victory image URLs
- Word blacklist

## 📊 Data Structures

### Game State
```python
GameState:
  - chat_id: Chat where game is happening
  - creator_id: User who started the game
  - players: Dict of all players
  - teams: Team assignments
  - spymasters: Spymaster assignments
  - board: Current game board
  - turn: Current team's turn
  - clue: Active clue (if any)
  - active: Is game running
  - started: Has game started
```

### Board Word
```python
BoardWord:
  - word: The word text
  - team: Team assignment (BJP, Congress, Neutral, Assassin)
  - revealed: Is word revealed
```

## 🔐 Security Features

- **Multi-chat Safety**: Each game is isolated per chat
- **Spymaster Protection**: DM-only clue transmission
- **Permission Checks**: Creator-only commands protected
- **Input Validation**: Clue and guess validation
- **Error Recovery**: Graceful error handling throughout

## 📈 Scalability

- **Async/Await**: Full async implementation for performance
- **Timer Management**: Non-blocking timer logic
- **Message Handling**: Efficient message processing with filters
- **Database Optimization**: MongoDB for persistent data (optional)
- **Logging**: Comprehensive logging for debugging and monitoring

## 🐛 Error Handling

The bot implements robust error handling:

- Try-except blocks on all handler functions
- Graceful fallbacks for failed operations
- Logging of all errors for debugging
- User-friendly error messages

## 📝 Logging

Logging is configured through `src/config.py`:

- Set `LOG_LEVEL` environment variable (DEBUG, INFO, WARNING, ERROR)
- Logs include timestamps and module names
- All game events are logged

## 🔄 Game Flow

```
1. /startgame → Create lobby
2. /join [players] → Players join
3. /start → Assign teams, generate board
4. /clue → Spymaster gives clue
5. /guess → Players make guesses
6. /endturn → Move to next team
7. Repeat 4-6 until win condition
8. /cancel → End game if needed
```

## 🎯 Type Hints

The codebase uses full type hints for IDE support and type checking:

```bash
mypy src/
```

## 🧪 Testing

Run tests with:

```bash
pytest
```

## 📦 Deployment

For production deployment:

1. Use a process manager (supervisor, systemd)
2. Set environment variables securely
3. Enable MongoDB for persistent data
4. Configure logging appropriately
5. Use a reverse proxy if needed

Example systemd service:
```ini
[Unit]
Description=Codenames Bot
After=network.target

[Service]
Type=simple
User=bot
WorkingDirectory=/path/to/bot
Environment="TOKEN=your_token"
Environment="MONGODB_URL=your_mongodb"
ExecStart=/usr/bin/python3 main.py
Restart=always

[Install]
WantedBy=multi-user.target
```

## 🤝 Contributing

Improvements welcome! Areas for enhancement:

- Difficulty levels
- Custom word sets
- Game statistics
- UI improvements
- AI spymaster option

## 📄 License

[Specify your license here]

## 👨‍💻 Author

Bhawani Singh Shekhawat

## 🙏 Credits

- Telegram Bot API (telegram-bot-api)
- python-telegram-bot library
- Original Codenames game (Czech Games Edition)

## 📞 Support

For issues or questions:
1. Check the `/help` and `/guide` commands in-bot
2. Review the logs at `DEBUG` level
3. Verify environment variables are set
4. Check MongoDB connection if using stats

---

**Version**: 2.0.0 | **Last Updated**: 2024
