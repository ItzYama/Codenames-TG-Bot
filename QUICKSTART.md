# Quick Start Guide

## 🚀 Get Started in 5 Minutes

### 1. Prerequisites
- Python 3.8+
- Telegram Bot Token (from [@BotFather](https://t.me/botfather))
- (Optional) MongoDB Atlas account

### 2. Clone & Setup

```bash
cd Codenames-Telegram-Bot
python -m venv venv
source venv/bin/activate  # Windows: venv\Scripts\activate
pip install -r requirements.txt
```

### 3. Configure

```bash
cp .env.example .env
# Edit .env file with your token:
# TOKEN=your_telegram_bot_token_here
```

### 4. Run

```bash
python main.py
```

### 5. Test

Open Telegram and:
```
/startgame
/join
(repeat /join with another account)
/start
/clue word 2
/guess word
```

## 📁 Project Structure

```
src/
├── config.py     - Configuration & constants
├── game.py       - Core game logic
├── board.py      - Board management
├── handlers.py   - Telegram handlers
├── utils.py      - Helper functions
├── images.py     - Media handling
└── main.py       - App initialization
```

## 🎯 Main Concepts

### Game
Central game state management:
```python
game = Game(chat_id, creator)
game.add_player(user)
game.assign_teams()
game.initialize_board()
```

### Handlers
Telegram command handlers:
```python
async def startgame(update, context):
    # Create game
    # Send message
```

### Board
Word management:
```python
board = Board()
board.generate()
board.reveal_word("word")
```

## 💡 Common Tasks

### Add a New Command

1. Add handler in `handlers.py`:
```python
async def mycommand(update: Update, context: ContextTypes.DEFAULT_TYPE) -> None:
    """My new command"""
    chat_id = update.effective_chat.id
    # Your logic here
    await update.message.reply_text("Done!")
```

2. Register in `src/main.py`:
```python
app.add_handler(CommandHandler("mycommand", mycommand))
```

### Add a New Game Feature

1. Add method to `Game` class in `game.py`:
```python
def my_feature(self) -> bool:
    """Description of feature"""
    # Game logic
    return True
```

2. Call from handler:
```python
if game.my_feature():
    await update.message.reply_text("Feature worked!")
```

### Add Configuration

1. Add to `src/config.py`:
```python
MY_SETTING: int = int(os.getenv("MY_SETTING", "42"))
```

2. Set in `.env`:
```
MY_SETTING=100
```

3. Use anywhere:
```python
from config import MY_SETTING
value = MY_SETTING
```

## 🐛 Debugging

### Enable Debug Logging

```bash
export LOG_LEVEL=DEBUG
python main.py
```

### Check Specific Module

```python
import logging
logging.getLogger("handlers").setLevel(logging.DEBUG)
```

### View Game State

```python
# In a handler
print(f"Game state: {game.to_dict()}")
print(f"Current turn: {game.state.turn}")
print(f"Players: {len(game.state.players)}")
```

## 📦 Docker Quick Start

```bash
# Build
docker build -t codenames-bot .

# Run
docker run -e TOKEN="your_token" codenames-bot

# Or use Compose
docker-compose up
```

## 🔗 Key Files

- **main.py** - Entry point
- **src/config.py** - All configuration
- **src/game.py** - Game logic
- **src/handlers.py** - Commands
- **README.md** - Full documentation
- **ARCHITECTURE.md** - Technical details

## ❓ FAQ

**Q: Bot not responding?**
A: Check TOKEN in .env, verify bot is running, check logs

**Q: How to add a new shop item?**
A: Edit SHOP_ITEMS in config.py

**Q: How to change game rules?**
A: Modify Board.generate() in board.py

**Q: How to deploy to production?**
A: See DEPLOYMENT.md for guides

## 📞 Need Help?

- Check README.md for full docs
- See ARCHITECTURE.md for design
- Review PROJECT_SUMMARY.md for changes
- Check logs with `export LOG_LEVEL=DEBUG`

---

**Happy coding! 🎮**
