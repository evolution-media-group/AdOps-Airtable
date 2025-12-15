# Logger Implementation Pattern
## Complete JavaScript Logger Class for Airtable Automations

**Version:** v1.0.0
**Created:** 2025-12-15
**Purpose:** Comprehensive logging with API request/response tracking, console output buffering, and permanent audit trail

**Source:** Production-proven pattern from AdOps-TTD/CreativeAutomation project
**Related:** [Logging Principles & Standards](file:///Users/jlindsay/work/EMG_Common/LOGGING_STANDARDS/LOGGING_PRINCIPLES_AND_STANDARDS.md)

---

## Logging Principles

### Core Philosophy

**1. Console.log and Stdout-Based Logging**
- All logging uses JavaScript `console.log()` for Airtable automation compatibility
- Logs write to stdout and are captured by Airtable's automation execution log viewer
- No external logging libraries required - pure JavaScript implementation
- Log persistence handled by Airtable platform (not by automation script)
- Logs accessible via Airtable automation run history interface

**Why Console.log:**
- **Platform Native:** Airtable automations capture stdout automatically
- **Zero Dependencies:** No external logging libraries needed
- **Immediate Visibility:** Logs appear in real-time during automation runs
- **No Configuration:** Works out-of-the-box without setup
- **Historical Access:** Airtable preserves logs for past automation runs

**2. Console Output Buffering for Permanent Audit Trail**
- **CRITICAL PATTERN:** All console.log output is buffered in memory
- Buffered output exported to database log table (e.g., API_Log)
- Creates permanent audit trail beyond Airtable's 30-day log retention
- Complete console output stored with API transaction records
- Enables historical debugging and compliance requirements

**3. Observable System Behavior**
- Every external API call MUST be logged (request + response)
- State transitions MUST be logged (start, success, error, status changes)
- Execution context MUST be preserved through request correlation
- All logging dual-output: console.log (real-time) + buffer (permanent)

**4. Security-First Design**
- Sensitive data automatically sanitized before logging
- Log structure preserved for debugging while protecting credentials
- Never log raw authentication tokens or passwords
- Console.log outputs are visible to anyone with Airtable base access

---

## Complete Logger Class Implementation (Production)

```javascript
/**
 * Enhanced Logger Class - Production Implementation
 *
 * Features:
 * - Console.log based stdout logging (Airtable native)
 * - Console output buffering for permanent audit trail
 * - Request ID correlation across log entries
 * - API request/response logging with timing
 * - Automatic header sanitization
 * - Version and status tracking
 * - Multiple log levels (start, success, error, info, warning)
 * - Visual indicators (emojis) for quick log scanning
 * - Execution time tracking
 *
 * Standards Compliance:
 * - Follows EMG Common Logging Principles & Standards
 * - Production-proven in AdOps-TTD/CreativeAutomation project
 * - Airtable automation compatible (pure console.log)
 * - Provides complete API audit trail with dual output
 *
 * Platform: Airtable Automations (JavaScript)
 * Output: console.log → stdout → Airtable log viewer (real-time)
 *         consoleBuffer → database log table (permanent)
 */
class Logger {
    constructor(scriptName, version, releaseDate = null, status = null) {
        this.scriptName = scriptName;
        this.version = version;
        this.releaseDate = releaseDate;
        this.status = status;
        this.startTime = new Date();
        // Generate unique request ID for correlation
        this.requestId = Math.random().toString(36).substr(2, 9);
        // CRITICAL: Buffer all console output for permanent storage
        this.consoleBuffer = [];
    }

    /**
     * Base logging method - all other log methods use this
     * Outputs to BOTH console (real-time) AND buffer (permanent)
     */
    log(emoji, message) {
        const line = `${emoji} ${message} [${this.requestId}]`;
        console.log(line);              // Real-time Airtable viewer
        this.consoleBuffer.push(line);  // Permanent database storage
    }

    /**
     * Log script initialization
     * Optional custom message parameter for specialized initialization
     */
    logStart(message = null) {
        const versionInfo = this.releaseDate ? `v${this.version} (${this.releaseDate})` : `v${this.version}`;
        const statusInfo = this.status ? ` [${this.status}]` : '';
        const msg = message || `${this.scriptName} ${versionInfo}${statusInfo} starting...`;
        this.log('🚀', msg);
    }

    /**
     * Log informational messages
     */
    logInfo(message) {
        this.log('ℹ️', message);
    }

    /**
     * Log successful operations
     */
    logSuccess(message) {
        this.log('✅', message);
    }

    /**
     * Log warning conditions
     */
    logWarning(message) {
        this.log('⚠️', message);
    }

    /**
     * Log error conditions
     */
    logError(message) {
        this.log('❌', message);
    }

    /**
     * Log API request with full details
     * REQUIRED: Call BEFORE sending API request
     */
    logApiRequest(apiName, method, url, headers, body = null) {
        const timestamp = new Date().toISOString();
        this.log('🌐', `API REQUEST ${timestamp}`);

        // Log individual fields (both console and buffer)
        console.log(`   API: ${apiName}`);
        this.consoleBuffer.push(`   API: ${apiName}`);

        console.log(`   Method: ${method}`);
        this.consoleBuffer.push(`   Method: ${method}`);

        console.log(`   URL: ${url}`);
        this.consoleBuffer.push(`   URL: ${url}`);

        const headerStr = `   Headers: ${JSON.stringify(this.sanitizeHeaders(headers), null, 2)}`;
        console.log(headerStr);
        this.consoleBuffer.push(headerStr);

        if (body) {
            const bodyStr = `   Body: ${JSON.stringify(body, null, 2)}`;
            console.log(bodyStr);
            this.consoleBuffer.push(bodyStr);
        }
    }

    /**
     * Log API response with timing
     * REQUIRED: Call IMMEDIATELY after receiving API response
     */
    logApiResponse(apiName, statusCode, responseData, duration) {
        const timestamp = new Date().toISOString();
        this.log('🌐', `API RESPONSE ${timestamp}`);

        console.log(`   API: ${apiName}`);
        this.consoleBuffer.push(`   API: ${apiName}`);

        console.log(`   Status: ${statusCode}`);
        this.consoleBuffer.push(`   Status: ${statusCode}`);

        console.log(`   Duration: ${duration}ms`);
        this.consoleBuffer.push(`   Duration: ${duration}ms`);

        if (responseData) {
            const responseStr = JSON.stringify(responseData, null, 2);
            // Production uses 2000 character limit
            if (responseStr.length > 2000) {
                const truncated = `   Response: ${responseStr.substring(0, 2000)}... [TRUNCATED]`;
                console.log(truncated);
                this.consoleBuffer.push(truncated);
            } else {
                const full = `   Response: ${responseStr}`;
                console.log(full);
                this.consoleBuffer.push(full);
            }
        }
    }

    /**
     * Log API error with context
     * REQUIRED: Call in catch block for API failures
     */
    logApiError(apiName, error, context = null) {
        const timestamp = new Date().toISOString();
        this.log('❌', `API ERROR ${timestamp}`);

        console.log(`   API: ${apiName}`);
        this.consoleBuffer.push(`   API: ${apiName}`);

        console.log(`   Error: ${error.message}`);
        this.consoleBuffer.push(`   Error: ${error.message}`);

        if (context) {
            const contextStr = `   Context: ${JSON.stringify(context, null, 2)}`;
            console.log(contextStr);
            this.consoleBuffer.push(contextStr);
        }
    }

    /**
     * Sanitize authentication headers
     * Production: Simple header-only sanitization
     */
    sanitizeHeaders(headers) {
        const sanitized = { ...headers };
        if (sanitized['Authorization']) {
            sanitized['Authorization'] = sanitized['Authorization'].substring(0, 8) + '...[REDACTED]';
        }
        if (sanitized['Api-Key']) {
            sanitized['Api-Key'] = '[REDACTED]';
        }
        return sanitized;
    }

    /**
     * Export complete buffered console output
     * CRITICAL: Use this to store logs in database table
     */
    getConsoleOutput() {
        return this.consoleBuffer.join('\n');
    }

    /**
     * Get total execution time in milliseconds
     * Use for performance tracking and database logging
     */
    getExecutionTime() {
        return Date.now() - this.startTime.getTime();
    }
}
```

---

## Usage Examples

### Basic Initialization

```javascript
// Initialize logger with full version information
const logger = new Logger(
    "DataSyncScript",           // Script name
    "1.0.0",                    // Version
    "2025-12-15",               // Release date
    "Production Ready"          // Status
);

logger.logStart();
// Console output: 🚀 DataSyncScript v1.0.0 (2025-12-15) [Production Ready] starting... [abc123xyz]
```

### Complete Script Lifecycle with Database Logging

```javascript
const SCRIPT_VERSION = "1.0.0";
const SCRIPT_NAME = "MyAutomationScript";
const RELEASE_DATE = "2025-12-15";
const SCRIPT_STATUS = "Production Ready";

async function main() {
    const logger = new Logger(SCRIPT_NAME, SCRIPT_VERSION, RELEASE_DATE, SCRIPT_STATUS);

    try {
        // 1. Log script start
        logger.logStart();

        // 2. Log configuration/initialization info
        logger.logInfo(`Processing ${recordCount} records`);

        // 3. Main operations with detailed logging
        const result = await processRecords(logger);

        // 4. Log warnings if applicable
        if (result.skippedCount > 0) {
            logger.logWarning(`Skipped ${result.skippedCount} records`);
        }

        // 5. Log successful completion
        logger.logSuccess(`Processed ${result.successCount} records`);

        // 6. CRITICAL: Write buffered console output to database
        await writeToApiLog({
            scriptName: SCRIPT_NAME,
            scriptVersion: SCRIPT_VERSION,
            runTime: logger.getExecutionTime(),
            consoleOutput: logger.getConsoleOutput(),
            status: 'Success'
        });

    } catch (error) {
        logger.logError(`Script execution error: ${error.message}`);

        await writeToApiLog({
            scriptName: SCRIPT_NAME,
            scriptVersion: SCRIPT_VERSION,
            runTime: logger.getExecutionTime(),
            consoleOutput: logger.getConsoleOutput(),
            status: 'Failed',
            errorMessage: error.message
        });

        throw error;
    }
}

main();
```

### API Request Logging

```javascript
async function fetchData(url, logger) {
    const startTime = Date.now();
    const headers = {
        'Authorization': 'Bearer your_token_here',
        'Content-Type': 'application/json'
    };

    // Log request BEFORE sending
    logger.logApiRequest('Airtable', 'GET', url, headers);

    try {
        const response = await fetch(url, { method: 'GET', headers });
        const data = await response.json();
        const duration = Date.now() - startTime;

        // Log response IMMEDIATELY after receiving
        logger.logApiResponse('Airtable', response.status, data, duration);

        return data;

    } catch (error) {
        logger.logApiError('Airtable', error, { url });
        throw error;
    }
}
```

---

## Standards Compliance Checklist

**Every Airtable Automation Script MUST:**
- [ ] Initialize Logger with script name, version, release date, status
- [ ] Call `logStart()` at script entry point
- [ ] Log ALL external API calls (request + response + errors)
- [ ] Include request correlation ID in all log entries
- [ ] Sanitize authentication headers
- [ ] Call `logSuccess()` or `logError()` at script completion
- [ ] Call `getConsoleOutput()` to export buffered logs
- [ ] Write buffered console output to database log table

---

**Related Documentation:**
- [Logging Principles & Standards](file:///Users/jlindsay/work/EMG_Common/LOGGING_STANDARDS/LOGGING_PRINCIPLES_AND_STANDARDS.md)
- [AdOps-TTD/CLAUDE.md](file:///Users/jlindsay/CurrentWork/AdOps-TTD/CLAUDE.md) - Development conventions
- [Integration Principles & Standards](file:///Users/jlindsay/work/EMG_Common/INTEGRATION_STANDARDS/INTEGRATION_PRINCIPLES_AND_STANDARDS.md)

---

*Document Version: 1.0.0*
*Last Updated: 2025-12-15*
