# Functional Specification Document
## AI-Powered Error Monitor CLI

**Version:** 1.0  
**Date:** January 5, 2026  
**Status:** Draft

---

## 1. Executive Summary

### 1.1 Purpose
This document defines the functional requirements and specifications for an AI-Powered Error Monitor CLI application that automates error detection, notification, and resolution suggestions for PM2 and Laravel applications.

### 1.2 Scope
The application monitors error logs in real-time, sends Telegram notifications when errors occur, and leverages AI (OpenAI GPT-4 or Anthropic Claude) to provide intelligent code fix suggestions through an interactive Telegram interface.

### 1.3 Target Users
- Backend developers running PM2 or Laravel applications
- DevOps engineers monitoring production systems
- Development teams seeking automated error resolution assistance

---

## 2. System Overview

### 2.1 High-Level Architecture
```
┌─────────────────┐
│   Log Sources   │
│  (PM2/Laravel)  │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  Log Monitor    │
│   (Chokidar)    │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│ Error Detector  │
│  & Parser       │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  Telegram Bot   │
│   (Telegraf)    │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│   AI Service    │
│ (OpenAI/Claude) │
└─────────────────┘
```

### 2.2 Core Components
1. **Initialization Wizard** - Interactive setup for configuration
2. **Log Monitor** - Real-time file watching and error detection
3. **Telegram Integration** - Notification and interaction layer
4. **AI Service** - Code analysis and fix generation
5. **Configuration Manager** - Persistent settings storage

---

## 3. Functional Requirements

### 3.1 Initialization and Configuration

#### FR-1.1: Setup Wizard
**Priority:** P0 (Critical)  
**Description:** On first launch, the application must guide users through an interactive configuration process.

**Acceptance Criteria:**
- Display welcome screen with application description
- Prompt user to select monitoring engine (PM2 or Laravel)
- Collect and validate all required configuration parameters
- Save configuration to persistent storage
- Confirm successful setup

#### FR-1.2: Engine Selection
**Priority:** P0 (Critical)  
**Description:** Users must choose between PM2 and Laravel monitoring modes.

**Acceptance Criteria:**
- Present clear options: "PM2" or "Laravel"
- Accept user input via arrow keys or number selection
- Validate selection before proceeding
- Store selected engine in configuration

#### FR-1.3: Log Path Detection - PM2
**Priority:** P0 (Critical)  
**Description:** Automatically detect PM2 error log paths.

**Acceptance Criteria:**
- Execute `pm2 jlist` command programmatically
- Parse JSON output to extract `pm2_err_log_path` for each app
- Display list of detected paths with app names
- Allow user to confirm or manually edit paths
- Handle cases where no PM2 apps are running
- Validate that log files exist and are readable

**Error Handling:**
- If PM2 is not installed, display error message
- If no apps are running, prompt user to start apps first

#### FR-1.4: Log Path Detection - Laravel
**Priority:** P0 (Critical)  
**Description:** Detect Laravel log path from project root.

**Acceptance Criteria:**
- Prompt user for Laravel project root directory
- Validate directory exists
- Check for `storage/logs/laravel.log` file
- Display detected path for confirmation
- Allow manual path override if needed
- Validate log file is readable

**Error Handling:**
- If path doesn't exist, prompt to create or select different path
- If file is not readable, display permission error

#### FR-1.5: Telegram Configuration
**Priority:** P0 (Critical)  
**Description:** Collect Telegram bot credentials.

**Acceptance Criteria:**
- Prompt for Bot Token
- Validate token format (should match pattern: `\d+:[A-Za-z0-9_-]+`)
- Prompt for Chat ID
- Validate Chat ID is numeric
- Test connection by sending a test message
- Store credentials securely

**Error Handling:**
- Invalid token format: display error and re-prompt
- Connection test fails: display error with troubleshooting steps

#### FR-1.6: AI Provider Configuration
**Priority:** P0 (Critical)  
**Description:** Configure AI service provider.

**Acceptance Criteria:**
- Display options: "OpenAI" or "Anthropic Claude"
- Prompt for corresponding API key
- Validate API key format
- Test API connection with a simple request
- Store provider choice and API key securely

**Error Handling:**
- Invalid API key: display error and re-prompt
- API test fails: display quota or connectivity error

#### FR-1.7: Configuration Persistence
**Priority:** P0 (Critical)  
**Description:** Save configuration for future use.

**Acceptance Criteria:**
- Create `.env` file or `config.json` in project root
- Store all configuration parameters
- Exclude sensitive data from version control (add to `.gitignore`)
- Load existing configuration on subsequent launches
- Support configuration updates via CLI flag (e.g., `--reconfigure`)

---

### 3.2 Error Monitoring

#### FR-2.1: Real-time Log Watching
**Priority:** P0 (Critical)  
**Description:** Monitor error log files for new entries in real-time.

**Acceptance Criteria:**
- Use file watcher (chokidar) to detect file changes
- Only process new lines (tail behavior)
- Handle multiple log files simultaneously (for PM2 multi-app setups)
- Minimal CPU and memory usage
- Non-blocking operation

**Performance Requirements:**
- Log check latency: < 1 second
- Memory footprint: < 50MB for monitoring 10 log files

#### FR-2.2: Error Detection Logic
**Priority:** P0 (Critical)  
**Description:** Identify error entries from log content.

**Acceptance Criteria:**
- Detect lines containing keywords: `Error`, `Exception`, `Fatal`, `stack trace`
- Case-insensitive matching
- Support regex patterns for custom error detection
- Extract full error context (multi-line stack traces)
- Deduplicate identical errors within 5-minute window

**Exclusions:**
- Info, warning, or debug level logs
- Expected errors (configurable whitelist)

#### FR-2.3: Stack Trace Parsing
**Priority:** P0 (Critical)  
**Description:** Extract file path and line number from error messages.

**Acceptance Criteria:**
- Parse PM2 error format
- Parse Laravel error format with stack traces
- Extract file path (absolute or relative)
- Extract line number
- Handle various error formats (Node.js, PHP)

**Examples to Support:**
- Node.js: `at Object.<anonymous> (/path/to/file.js:123:45)`
- Laravel: `#0 /path/to/file.php(123): ClassName->methodName()`

#### FR-2.4: Log Rotation Handling
**Priority:** P1 (High)  
**Description:** Handle log file rotation gracefully.

**Acceptance Criteria:**
- Detect when log file is rotated, cleared, or renamed
- Re-attach watcher to new log file
- No errors lost during rotation
- Display notification when rotation is detected

---

### 3.3 Telegram Integration

#### FR-3.1: Error Notification
**Priority:** P0 (Critical)  
**Description:** Send formatted error notifications to Telegram.

**Acceptance Criteria:**
- Send message immediately when error is detected (< 2 seconds)
- Include error timestamp
- Include error type and message
- Include file path and line number
- Include first 10 lines of stack trace
- Format message for readability (use code blocks)
- Include inline keyboard with "🛠️ Fix this Error" button

**Message Format:**
```
⚠️ Error Detected

Time: 2026-01-05 17:00:00
Type: TypeError
File: /app/src/controllers/user.js:45

Message: Cannot read property 'name' of undefined

Stack Trace:
```
[First 10 lines of stack trace]
```

[🛠️ Fix this Error]
```

#### FR-3.2: Interactive Buttons
**Priority:** P0 (Critical)  
**Description:** Provide inline buttons for user interaction.

**Acceptance Criteria:**
- Button labeled "🛠️ Fix this Error"
- Callback data includes error ID for reference
- Visual feedback when button is clicked
- Disable button after first click to prevent duplicate requests
- Display "Analyzing..." message during processing

#### FR-3.3: AI Fix Response
**Priority:** P0 (Critical)  
**Description:** Send AI-generated fix suggestions to Telegram.

**Acceptance Criteria:**
- Send response within 10 seconds of button click
- Include error explanation
- Include suggested code fix with syntax highlighting
- Provide step-by-step resolution instructions
- Include disclaimer that AI suggestions should be reviewed

**Response Format:**
```
🤖 AI Analysis Complete

📋 Error Explanation:
[AI explanation of the error]

🔧 Suggested Fix:
```javascript
[Corrected code block]
```

📝 Steps to Fix:
1. [Step 1]
2. [Step 2]
...

⚠️ Note: Please review this suggestion before applying to production.
```

#### FR-3.4: Error Handling in Telegram
**Priority:** P1 (High)  
**Description:** Handle errors in Telegram communication.

**Acceptance Criteria:**
- Retry failed message sends (max 3 attempts)
- Display error if all retries fail
- Log telegram errors to console
- Continue monitoring even if Telegram fails

---

### 3.4 AI Integration

#### FR-4.1: Code Context Extraction
**Priority:** P0 (Critical)  
**Description:** Read source code around the error location.

**Acceptance Criteria:**
- Given file path and line number, read file contents
- Extract 20 lines: 10 before and 10 after error line
- Handle file not found errors gracefully
- Handle permission denied errors
- Support both absolute and relative paths
- Resolve relative paths based on project root

#### FR-4.2: AI Prompt Construction
**Priority:** P0 (Critical)  
**Description:** Create effective prompts for AI analysis.

**Acceptance Criteria:**
- Include error type and message
- Include stack trace
- Include code context (20 lines)
- Include file path and language
- Request explanation and fix
- Optimize prompt length for API limits

**Prompt Template:**
```
You are a senior developer. Analyze this error and provide a fix.

Error Type: [ERROR_TYPE]
Error Message: [ERROR_MESSAGE]
File: [FILE_PATH] (Line [LINE_NUMBER])
Language: [LANGUAGE]

Stack Trace:
[STACK_TRACE]

Code Context:
```[LANGUAGE]
[CODE_CONTEXT]
```

Please:
1. Explain why this error occurred
2. Provide the corrected code
3. Suggest steps to prevent this in the future
```

#### FR-4.3: OpenAI Integration
**Priority:** P0 (Critical)  
**Description:** Integrate with OpenAI API.

**Acceptance Criteria:**
- Use GPT-4 or GPT-3.5-turbo model
- Set appropriate temperature (0.3 for code)
- Handle API rate limits
- Handle API errors gracefully
- Parse and format response
- Log API usage for monitoring

#### FR-4.4: Anthropic Claude Integration
**Priority:** P0 (Critical)  
**Description:** Integrate with Anthropic Claude API.

**Acceptance Criteria:**
- Use Claude 3 (Opus, Sonnet, or Haiku)
- Set appropriate parameters
- Handle API rate limits
- Handle API errors gracefully
- Parse and format response
- Log API usage for monitoring

#### FR-4.5: AI Response Parsing
**Priority:** P1 (High)  
**Description:** Extract and format AI responses.

**Acceptance Criteria:**
- Extract explanation section
- Extract code blocks with proper syntax
- Extract step-by-step instructions
- Handle malformed responses
- Validate response completeness

---

### 3.5 Configuration Management

#### FR-5.1: Environment Variables
**Priority:** P0 (Critical)  
**Description:** Manage configuration via .env file.

**Acceptance Criteria:**
- Support `.env` file in project root
- Load environment variables on startup
- Support following variables:
  - `ENGINE` (PM2 or Laravel)
  - `LOG_PATHS` (comma-separated for PM2)
  - `TELEGRAM_BOT_TOKEN`
  - `TELEGRAM_CHAT_ID`
  - `AI_PROVIDER` (openai or anthropic)
  - `OPENAI_API_KEY`
  - `ANTHROPIC_API_KEY`

#### FR-5.2: Configuration Validation
**Priority:** P0 (Critical)  
**Description:** Validate configuration on startup.

**Acceptance Criteria:**
- Check required fields are present
- Validate field formats
- Test file permissions for log paths
- Test API connectivity
- Display clear error messages for invalid config
- Exit gracefully if validation fails

#### FR-5.3: Reconfiguration Support
**Priority:** P1 (High)  
**Description:** Allow users to update configuration.

**Acceptance Criteria:**
- Support `--reconfigure` CLI flag
- Re-run setup wizard
- Preserve existing values as defaults
- Update configuration file
- Restart monitoring with new config

---

## 4. User Stories and Use Cases

### 4.1 User Story 1: First-Time Setup
**As a** developer  
**I want to** easily configure the error monitor  
**So that** I can start monitoring my application without reading extensive documentation

**Acceptance Criteria:**
- Setup completes in under 2 minutes
- All steps are clearly explained
- Errors are caught and explained
- Confirmation message shown on success

### 4.2 User Story 2: Receive Error Notification
**As a** developer  
**I want to** receive instant notifications when errors occur  
**So that** I can respond to production issues quickly

**Acceptance Criteria:**
- Notification received within 2 seconds of error
- Error details are clearly formatted
- Stack trace is included
- Fix button is available

### 4.3 User Story 3: Get AI Fix Suggestion
**As a** developer  
**I want to** get AI-powered fix suggestions  
**So that** I can resolve errors faster

**Acceptance Criteria:**
- Click "Fix this Error" button
- Receive explanation within 10 seconds
- See suggested code fix
- Understand steps to implement fix

### 4.4 User Story 4: Monitor Multiple Apps
**As a** developer running multiple PM2 apps  
**I want to** monitor all apps simultaneously  
**So that** I don't miss errors from any service

**Acceptance Criteria:**
- All PM2 apps detected during setup
- Each app monitored independently
- Notifications identify which app errored
- No performance degradation with multiple apps

---

## 5. Data Models

### 5.1 Configuration Schema

```json
{
  "engine": "pm2 | laravel",
  "logPaths": ["string"],
  "telegram": {
    "botToken": "string",
    "chatId": "string"
  },
  "ai": {
    "provider": "openai | anthropic",
    "apiKey": "string",
    "model": "string"
  },
  "options": {
    "errorDeduplicationWindow": 300,
    "maxStackTraceLines": 10,
    "codeContextLines": 20
  }
}
```

### 5.2 Error Event Schema

```javascript
{
  id: "uuid",
  timestamp: "ISO8601 datetime",
  source: {
    engine: "pm2 | laravel",
    appName: "string",
    logPath: "string"
  },
  error: {
    type: "string",
    message: "string",
    stackTrace: "string[]",
    filePath: "string",
    lineNumber: number
  },
  status: "pending | analyzing | resolved | ignored",
  aiResponse: {
    explanation: "string",
    suggestedFix: "string",
    steps: "string[]",
    timestamp: "ISO8601 datetime"
  }
}
```

---

## 6. API Specifications

### 6.1 Internal CLI Commands

#### Command: `start`
```bash
node index.js start
```
**Description:** Start monitoring with existing configuration  
**Exit Codes:**
- 0: Success
- 1: Configuration error
- 2: Permission error

#### Command: `setup`
```bash
node index.js setup
```
**Description:** Run configuration wizard  
**Exit Codes:**
- 0: Setup complete
- 1: Setup cancelled

#### Command: `test`
```bash
node index.js test
```
**Description:** Test all integrations (Telegram, AI, Log access)  
**Exit Codes:**
- 0: All tests passed
- 1: One or more tests failed

#### Command: `status`
```bash
node index.js status
```
**Description:** Display current monitoring status  
**Output:** JSON object with monitoring status

### 6.2 External API Integrations

#### Telegram Bot API
- **Endpoint:** `https://api.telegram.org/bot{token}/sendMessage`
- **Method:** POST
- **Rate Limit:** 30 messages per second

#### OpenAI API
- **Endpoint:** `https://api.openai.com/v1/chat/completions`
- **Method:** POST
- **Rate Limit:** Varies by tier
- **Timeout:** 30 seconds

#### Anthropic API
- **Endpoint:** `https://api.anthropic.com/v1/messages`
- **Method:** POST
- **Rate Limit:** Varies by tier
- **Timeout:** 30 seconds

---

## 7. Error Handling and Edge Cases

### 7.1 Error Scenarios

| Scenario | Handling |
|----------|----------|
| Log file deleted | Re-attach watcher when file recreated |
| Log file rotated | Detect and attach to new file |
| Permission denied | Display error, skip that file |
| API rate limit | Queue requests, retry with backoff |
| Network timeout | Retry 3 times, then skip |
| Invalid config | Exit with clear error message |
| Telegram blocked | Log error, continue monitoring |
| AI API down | Fall back to basic error notification |
| Out of memory | Restart monitoring, log error |
| Circular error (monitor crashes) | Implement circuit breaker |

### 7.2 Edge Cases

1. **Empty log file:** Skip until content is added
2. **Binary log file:** Display error, require text logs
3. **Very large stack trace (>1000 lines):** Truncate to first 100 lines
4. **Same error repeated 100+ times:** Group and send summary
5. **Multiple errors in same second:** Queue and send sequentially
6. **Log file on network drive:** Warn about latency
7. **Non-standard error format:** Use fallback regex patterns

---

## 8. Non-Functional Requirements

### 8.1 Performance
- **Startup time:** < 3 seconds
- **Error detection latency:** < 1 second from log write
- **Telegram notification latency:** < 2 seconds
- **AI response time:** < 15 seconds (90th percentile)
- **Memory usage:** < 100MB with 10 monitored files
- **CPU usage:** < 5% average

### 8.2 Reliability
- **Uptime:** 99.9% (excluding planned maintenance)
- **Error recovery:** Automatic restart on crash
- **Data loss:** Zero errors missed under normal operation

### 8.3 Security
- **Secrets storage:** Environment variables only
- **File permissions:** Read-only access to logs
- **API keys:** Never logged or displayed
- **Transport:** HTTPS for all external API calls

### 8.4 Scalability
- **Max log files:** 50 simultaneous
- **Max error rate:** 100 errors/minute sustained
- **Max log file size:** No limit (tail only)

### 8.5 Usability
- **Setup time:** < 5 minutes for first-time users
- **Learning curve:** No prior CLI experience required
- **Documentation:** Complete README and inline help

---

## 9. Testing Requirements

### 9.1 Unit Tests
- Configuration parser
- Log file watcher
- Error pattern matching
- Stack trace parser
- AI prompt builder

### 9.2 Integration Tests
- PM2 log path detection
- Laravel log path detection
- Telegram message sending
- OpenAI API integration
- Anthropic API integration

### 9.3 End-to-End Tests
1. **Full PM2 workflow:**
   - Setup with PM2
   - Trigger error in PM2 app
   - Receive Telegram notification
   - Click fix button
   - Receive AI suggestion

2. **Full Laravel workflow:**
   - Setup with Laravel
   - Trigger error in Laravel app
   - Receive Telegram notification
   - Click fix button
   - Receive AI suggestion

### 9.4 Error Scenario Tests
- Log file rotation
- Permission denied
- API rate limiting
- Network timeout
- Invalid configuration
- Malformed error logs

---

## 10. Acceptance Criteria

### 10.1 MVP Release Criteria
- ✅ Supports PM2 and Laravel
- ✅ Interactive setup wizard completes successfully
- ✅ Detects errors in real-time
- ✅ Sends Telegram notifications
- ✅ AI fix suggestions working for both OpenAI and Claude
- ✅ Configuration persists across restarts
- ✅ Handles log rotation
- ✅ Basic error handling implemented
- ✅ README documentation complete

### 10.2 Production Readiness
- ✅ All unit tests passing (>80% coverage)
- ✅ All integration tests passing
- ✅ End-to-end tests passing
- ✅ Performance benchmarks met
- ✅ Security audit completed
- ✅ Documentation reviewed
- ✅ Beta testing with 3+ users
- ✅ Known issues documented

---

## 11. Future Enhancements (Out of Scope for v1.0)

### 11.1 Planned Features
- **Multi-user support:** Different Telegram users/groups
- **Web dashboard:** View error history and statistics
- **Custom error rules:** User-defined patterns and actions
- **Auto-fix application:** Automatically apply AI suggestions with approval
- **Plugin system:** Support for other log sources (Docker, Nginx, etc.)
- **Error trending:** ML-based anomaly detection
- **Team collaboration:** Share errors and fixes with team
- **CI/CD integration:** Report errors to GitHub/GitLab issues

### 11.2 Potential Integrations
- Slack, Discord notifications
- PagerDuty, OpsGenie alerts
- GitHub Issues auto-creation
- Sentry, DataDog integration
- Jira ticket creation

---

## 12. Appendix

### 12.1 Glossary
- **PM2:** Process manager for Node.js applications
- **Laravel:** PHP web application framework
- **Telegraf:** Telegram Bot API framework
- **Chokidar:** File watcher library
- **Stack Trace:** Detailed error location trace

### 12.2 References
- [PM2 Documentation](https://pm2.keymetrics.io/)
- [Laravel Logging](https://laravel.com/docs/logging)
- [Telegram Bot API](https://core.telegram.org/bots/api)
- [OpenAI API Docs](https://platform.openai.com/docs)
- [Anthropic API Docs](https://docs.anthropic.com/)

### 12.3 Document History
| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | 2026-01-05 | Generated | Initial FSD draft |

---

**Document Status:** Ready for Review  
**Next Steps:** Technical review and stakeholder approval
