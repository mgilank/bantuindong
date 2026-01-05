# Technical Requirements Document: AI-Powered Error Monitor CLI

## 1. Project Overview
Create a CLI application that monitors error logs from **PM2** or **Laravel**, sends notifications to **Telegram**, and uses **AI (Claude/OpenAI)** to suggest code fixes directly via Telegram interaction.

## 2. Technical Stack
- **Language:** Node.js (v18+)
- **CLI Framework:** `commander` or `inquirer` for the setup wizard.
- **Log Monitoring:** `tail` or `chokidar` for real-time file watching.
- **Telegram Bot:** `telegraf` library.
- **AI Integration:** OpenAI API (Codex/GPT-4) or Anthropic API (Claude 3).

---

## 3. CLI Workflow & Requirements

### A. Initialization Wizard
When the app starts (e.g., `node index.js`), it must prompt the user:
1.  **Select Engine:** `PM2` or `Laravel`.
2.  **Path Detection:**
    - **If PM2:** Run `pm2 jlist` to fetch the list of running apps and their `pm2_err_log_path`. Display paths to user for confirmation.
    - **If Laravel:** Ask for the project root and look for `storage/logs/laravel.log`.
3.  **Telegram Config:** Ask for `BOT_TOKEN` and `CHAT_ID`.
4.  **AI Config:** Ask for `AI_PROVIDER` (OpenAI/Claude) and the corresponding `API_KEY`.
5.  **Persistence:** Save these configs in a `.env` or `config.json` file.

### B. Error Monitoring Logic
- Monitor only the **Error Log** file.
- Use a "tail" mechanism to read only new lines added to the file.
- **Trigger:** Send a message to Telegram only if the line contains keywords like `Error`, `Exception`, `stack trace`, or `Fatal`.

### C. Telegram Interaction & AI Fix
1.  **Notification:** When an error occurs, send the log snippet to Telegram with an **Inline Button**: `[ 🛠️ Fix this Error ]`.
2.  **Execution:** When the user clicks the button:
    - The CLI must parse the stack trace to find the **File Path** and **Line Number**.
    - Read the file content (approx. 20 lines around the error).
    - Send the context (Error + Code Snippet) to the chosen AI Engine.
3.  **Prompt to AI:** *"You are a senior developer. Here is an error log: [ERROR]. Here is the source code: [CODE]. Please explain why this happened and provide the corrected code block."*
4.  **Response:** Send the AI's explanation and fix back to the user's Telegram.

---

## 4. Engineering Prompt for AI Agent

**Copy and paste the prompt below into your AI Agent (Claude/GPT):**

> "Act as a Senior Node.js Developer. Build a CLI tool based on the following specs:
> 
> 1. **Setup Wizard:** Use `inquirer` to let users choose between 'PM2' and 'Laravel' monitoring. For PM2, auto-detect log paths using `pm2 jlist`. For Laravel, target `storage/logs/laravel.log`.
> 2. **Log Watcher:** Implement a non-blocking log watcher that monitors new entries. Filter for 'Error' or 'Exception' strings.
> 3. **Telegram Integration:** Use `telegraf`. When an error is detected, send the log to a specific Chat ID with an inline button 'Fix with AI'.
> 4. **AI Logic:** >    - On button click, the app should parse the file path from the error string.
>    - Read the local file content around the error line.
>    - Send the error and code context to OpenAI/Claude API.
>    - Return the suggested fix to the Telegram chat.
> 5. **Code Quality:** Use modular structure (logger.js, telegram.js, ai-service.js). Provide the `package.json` and a clear `README.md` for installation.
> 
> Start by outlining the file structure, then provide the full implementation."

---

## 5. Security Considerations
- **Environment Variables:** Ensure sensitive data like API Keys and Bot Tokens are stored in `.env` and not hardcoded.
- **File Permissions:** The CLI must have read access to the log files and source code directories.
- **Log Rotation:** The monitor should handle log rotation (when the log file is cleared or archived).