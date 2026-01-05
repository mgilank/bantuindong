# AI-Powered Error Monitor CLI

An intelligent CLI application that monitors error logs from **PM2** or **Laravel** applications, sends real-time notifications to **Telegram**, and leverages **AI** (Claude/OpenAI) to suggest code fixes directly through Telegram interaction.

## ✨ Features

- 🔍 **Real-time Error Monitoring** - Automatically monitors PM2 or Laravel error logs
- 📱 **Telegram Notifications** - Instant error alerts sent directly to your Telegram
- 🤖 **AI-Powered Fix Suggestions** - Get intelligent code fix recommendations from Claude or GPT-4
- 🎯 **Interactive Workflow** - One-click error analysis and fix suggestions via Telegram inline buttons
- ⚙️ **Easy Setup** - Interactive wizard for quick configuration
- 🔒 **Secure** - Environment variables for sensitive data

## 🛠️ Technical Stack

- **Language:** Node.js (v18+)
- **CLI Framework:** `commander` or `inquirer` for interactive setup
- **Log Monitoring:** `tail` or `chokidar` for real-time file watching
- **Telegram Bot:** `telegraf` library
- **AI Integration:** OpenAI API (GPT-4) or Anthropic API (Claude 3)

## 📋 Prerequisites

- Node.js v18 or higher
- A Telegram Bot Token (get from [@BotFather](https://t.me/botfather))
- Your Telegram Chat ID
- API Key for OpenAI or Anthropic Claude
- PM2 or Laravel application running

## 🚀 Installation

```bash
# Clone the repository
git clone <repository-url>
cd 77.bantuindong

# Install dependencies
npm install

# Start the CLI
node index.js
```

## ⚙️ Configuration

On first run, the interactive wizard will guide you through the setup:

### 1. Select Monitoring Engine
Choose between:
- **PM2** - Monitors PM2 application error logs
- **Laravel** - Monitors Laravel application logs

### 2. Log Path Detection
- **PM2:** Automatically detects log paths using `pm2 jlist`
- **Laravel:** Prompts for project root and monitors `storage/logs/laravel.log`

### 3. Telegram Configuration
Provide:
- Bot Token (from [@BotFather](https://t.me/botfather))
- Chat ID (your Telegram chat ID)

### 4. AI Configuration
Choose your AI provider:
- **OpenAI** (GPT-4/Codex) - Provide OpenAI API Key
- **Anthropic Claude** (Claude 3) - Provide Anthropic API Key

All configurations are saved in `.env` or `config.json` for future use.

## 📖 How It Works

### Error Detection
1. The CLI monitors your error log file in real-time
2. Filters for keywords: `Error`, `Exception`, `stack trace`, `Fatal`
3. Sends a notification to Telegram when an error is detected

### AI-Powered Fixes
1. **Notification:** Error snippet sent to Telegram with a `[ 🛠️ Fix this Error ]` button
2. **Analysis:** When clicked, the CLI:
   - Parses the stack trace for file path and line number
   - Reads ~20 lines of code around the error
   - Sends context to AI engine
3. **Solution:** AI explains the issue and provides corrected code
4. **Delivery:** Fix suggestion sent back to your Telegram chat

## 🏗️ Project Structure

```
.
├── index.js              # Main entry point
├── logger.js             # Log monitoring logic
├── telegram.js           # Telegram bot integration
├── ai-service.js         # AI API integration
├── config.json           # Configuration storage
├── .env                  # Environment variables
└── package.json          # Dependencies
```

## 🔐 Security Considerations

- **Environment Variables:** All sensitive data (API keys, tokens) stored in `.env`
- **File Permissions:** Ensure the CLI has read access to log files and source directories
- **Log Rotation:** Handles log file rotation and archiving automatically

## 🎯 AI Prompt Template

When an error is detected, the following prompt is sent to the AI:

```
You are a senior developer. Here is an error log: [ERROR]. 
Here is the source code: [CODE]. 
Please explain why this happened and provide the corrected code block.
```

## 📝 Example Workflow

```bash
# Start monitoring
$ node index.js

✓ Monitoring PM2 error logs...
✓ Telegram bot connected
✓ AI service ready (OpenAI GPT-4)

# When an error occurs:
# 1. Telegram notification received with error details
# 2. Click "🛠️ Fix this Error" button
# 3. AI analyzes the error and code
# 4. Receive fix suggestion in Telegram
```

## 🤝 Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

## 📄 License

This project is licensed under the MIT License.

## 🔗 Resources

- [PM2 Documentation](https://pm2.keymetrics.io/)
- [Laravel Logging](https://laravel.com/docs/logging)
- [Telegraf Documentation](https://telegraf.js.org/)
- [OpenAI API](https://platform.openai.com/docs)
- [Anthropic Claude API](https://docs.anthropic.com/)

---

**Note:** For detailed technical specifications, see [architechture.md](./architechture.md)
