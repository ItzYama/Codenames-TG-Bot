# Deployment Guide

This guide covers deployment of the Codenames Telegram Bot to various platforms.

## Local Development

### Prerequisites
- Python 3.8+
- pip (Python package manager)
- Git

### Setup

1. **Clone the repository**
   ```bash
   git clone <repository-url>
   cd Codenames-Telegram-Bot
   ```

2. **Create virtual environment**
   ```bash
   python -m venv venv
   source venv/bin/activate  # On Windows: venv\Scripts\activate
   ```

3. **Install dependencies**
   ```bash
   pip install -r requirements.txt
   ```

4. **Configure environment**
   ```bash
   cp .env.example .env
   # Edit .env with your values
   nano .env
   ```

5. **Run the bot**
   ```bash
   python main.py
   ```

## Docker Deployment

### Prerequisites
- Docker
- Docker Compose (optional)

### Dockerfile

Create a `Dockerfile`:

```dockerfile
FROM python:3.11-slim

WORKDIR /app

# Install system dependencies
RUN apt-get update && apt-get install -y \
    gcc \
    && rm -rf /var/lib/apt/lists/*

# Copy requirements
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Copy application
COPY . .

# Set environment
ENV PYTHONUNBUFFERED=1

# Download NLTK data during build
RUN python -m nltk.downloader words

# Run
CMD ["python", "main.py"]
```

### Build and Run

```bash
# Build image
docker build -t codenames-bot:latest .

# Run container
docker run -d \
  -e TOKEN="your_token" \
  -e GEMINI_API_KEY="your_key" \
  -e MONGODB_URL="your_url" \
  --name codenames-bot \
  codenames-bot:latest
```

### Docker Compose

Create `docker-compose.yml`:

```yaml
version: '3.8'

services:
  codenames-bot:
    build: .
    container_name: codenames-telegram-bot
    environment:
      TOKEN: ${TOKEN}
      GEMINI_API_KEY: ${GEMINI_API_KEY}
      MONGODB_URL: ${MONGODB_URL}
      LOG_LEVEL: ${LOG_LEVEL:-INFO}
    restart: unless-stopped
    logging:
      driver: "json-file"
      options:
        max-size: "10m"
        max-file: "3"

  mongodb:
    image: mongo:latest
    container_name: codenames-mongodb
    volumes:
      - mongodb-data:/data/db
    environment:
      MONGO_INITDB_ROOT_USERNAME: ${MONGO_USER:-admin}
      MONGO_INITDB_ROOT_PASSWORD: ${MONGO_PASSWORD:-password}
    restart: unless-stopped
    logging:
      driver: "json-file"
      options:
        max-size: "10m"
        max-file: "3"

volumes:
  mongodb-data:
```

Run with:
```bash
docker-compose up -d
```

## VPS Deployment (Linux/Ubuntu)

### Prerequisites
- Ubuntu 20.04 or newer
- SSH access
- sudo privileges

### Setup

1. **Update system**
   ```bash
   sudo apt update && sudo apt upgrade -y
   ```

2. **Install Python and dependencies**
   ```bash
   sudo apt install -y python3.11 python3-pip python3-venv git
   ```

3. **Clone repository**
   ```bash
   cd /opt
   sudo git clone <repository-url>
   sudo chown -R $USER:$USER Codenames-Telegram-Bot
   cd Codenames-Telegram-Bot
   ```

4. **Setup virtual environment**
   ```bash
   python3 -m venv venv
   source venv/bin/activate
   pip install --upgrade pip
   pip install -r requirements.txt
   ```

5. **Configure environment**
   ```bash
   cp .env.example .env
   nano .env  # Fill in your values
   ```

6. **Install systemd service**
   
   Create `/etc/systemd/system/codenames-bot.service`:
   ```ini
   [Unit]
   Description=Codenames Telegram Bot
   After=network.target

   [Service]
   Type=simple
   User=<your_user>
   WorkingDirectory=/opt/Codenames-Telegram-Bot
   Environment="PATH=/opt/Codenames-Telegram-Bot/venv/bin"
   EnvironmentFile=/opt/Codenames-Telegram-Bot/.env
   ExecStart=/opt/Codenames-Telegram-Bot/venv/bin/python main.py
   Restart=always
   RestartSec=10
   StandardOutput=journal
   StandardError=journal

   [Install]
   WantedBy=multi-user.target
   ```

7. **Enable and start service**
   ```bash
   sudo systemctl daemon-reload
   sudo systemctl enable codenames-bot
   sudo systemctl start codenames-bot
   sudo systemctl status codenames-bot
   ```

8. **Monitor logs**
   ```bash
   sudo journalctl -u codenames-bot -f
   ```

## Heroku Deployment

### Prerequisites
- Heroku account
- Heroku CLI installed

### Setup

1. **Create Procfile**
   ```
   worker: python main.py
   ```

2. **Create runtime.txt**
   ```
   python-3.11.0
   ```

3. **Login to Heroku**
   ```bash
   heroku login
   ```

4. **Create app**
   ```bash
   heroku create codenames-bot-app
   ```

5. **Set environment variables**
   ```bash
   heroku config:set TOKEN="your_token"
   heroku config:set GEMINI_API_KEY="your_key"
   heroku config:set MONGODB_URL="your_url"
   ```

6. **Deploy**
   ```bash
   git push heroku main
   ```

7. **View logs**
   ```bash
   heroku logs -t
   ```

## AWS Deployment

### Using Lambda + API Gateway

1. **Create Lambda function**
   - Runtime: Python 3.11
   - Handler: main.lambda_handler

2. **Configure environment variables** in Lambda

3. **Set up API Gateway** for webhook

4. **Deploy and configure webhook** with Telegram

### Using EC2

Follow VPS deployment steps above.

## Monitoring & Maintenance

### Logs

Location depends on deployment:
- **Local**: Console output
- **Docker**: `docker logs -f codenames-bot`
- **Systemd**: `journalctl -u codenames-bot -f`
- **Heroku**: `heroku logs -t`

### Database Backups

For MongoDB:
```bash
# Backup
mongodump --uri="mongodb+srv://..." --out=./backup

# Restore
mongorestore --uri="mongodb+srv://..." ./backup
```

### Health Checks

Monitor bot responsiveness:
```bash
# Check if bot is running (bot should respond to /help in a test chat)
```

## Troubleshooting

### Bot Not Responding
1. Check TOKEN is correct
2. Verify internet connectivity
3. Check logs for errors
4. Ensure bot is running

### Database Connection Issues
1. Verify MONGODB_URL is correct
2. Check firewall allows connection
3. Verify IP whitelist in MongoDB Atlas
4. Check credentials

### Memory Issues
- Monitor with `top` or `docker stats`
- Increase swap if needed
- Consider limiting concurrent games

## Performance Optimization

1. **Connection Pooling**
   - Already handled by python-telegram-bot

2. **Message Caching**
   - Edit message text instead of deleting/resending

3. **Database Indexing**
   - Add indexes on frequently queried fields

4. **Rate Limiting**
   - Implement request throttling if needed

## Security Considerations

1. **Environment Variables**
   - Never commit .env files
   - Use secret management in production

2. **Database Security**
   - Use MongoDB Atlas IP whitelist
   - Enable SSL/TLS
   - Use strong passwords

3. **API Keys**
   - Rotate keys periodically
   - Use separate keys for dev/prod
   - Monitor usage

## Backup Strategy

1. **Code**: Use Git for version control
2. **Data**: Regular MongoDB backups
3. **Configuration**: Store .env securely
4. **Frequency**: Daily backups recommended

## Update & Rollback

### Update Code
```bash
git pull origin main
pip install -r requirements.txt --upgrade
systemctl restart codenames-bot
```

### Rollback
```bash
git revert <commit>
git push
# Redeploy
```

## Support

For deployment issues:
1. Check logs first
2. Verify environment setup
3. Test locally before production
4. Review platform-specific documentation
