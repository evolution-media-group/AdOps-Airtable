# AdOps-Airtable
## Claude Code Configuration and Development Guide

**Platform Family**: AdOps Automation Platforms
**Platform**: Airtable API & Automation
**Related Projects**: AdOps-TTD, AdOps-SFC, AdOps-Camphouse
**Status**: Initial Setup v1.0.0

---

## 📚 Related Documentation

**IMPORTANT: This repository follows universal standards documented in EMG Common:**

- **[Integration Principles & Standards](file:///Users/jlindsay/work/EMG_Common/INTEGRATION_STANDARDS/INTEGRATION_PRINCIPLES_AND_STANDARDS.md)** - Universal integration guidelines for all projects
  - Location: `~/work/EMG_Common/INTEGRATION_STANDARDS/INTEGRATION_PRINCIPLES_AND_STANDARDS.md`
  - Covers: Universal identifier management, cross-platform integration patterns, field type conventions, decoupled integration best practices
  - Technology-agnostic standards for all integrations
  - **MANDATORY reading for all developers**
  - **Note:** This is a shared resource across ALL projects, regardless of git repository

- **[Logging Principles & Standards](file:///Users/jlindsay/work/EMG_Common/LOGGING_STANDARDS/LOGGING_PRINCIPLES_AND_STANDARDS.md)** - Universal logging guidelines for all projects
  - Location: `~/work/EMG_Common/LOGGING_STANDARDS/LOGGING_PRINCIPLES_AND_STANDARDS.md`
  - Covers: Log levels, API payload logging, execution traces, sanitization, troubleshooting
  - Technology-agnostic standards for validation and debugging
  - **MANDATORY reading for all developers**
  - **Note:** This is a shared resource across ALL projects, regardless of git repository

---

## 🏗️ AdOps Platform Family Conventions

**This project follows the standards established across the AdOps automation platform family:**

### Sister Projects
- **AdOps-TTD**: The Trade Desk creative, audience, and campaign automation
  - Location: `~/work/AdOps-TTD/`
  - Established patterns: Logging, error handling, version control, Airtable automation standards
  - Reference: CreativeAutomation patterns for Airtable scripting

- **AdOps-SFC**: Salesforce API integration tools
  - Location: `~/work/AdOps-SFC/`
  - Established patterns: Python CLI tools, AWS Secrets Manager integration

- **AdOps-Camphouse**: Camphouse/Mediatool platform automation
  - Location: `~/work/AdOps-Camphouse/`
  - Established patterns: API synchronization, batch processing

### Platform Family Standards
- **Consistent structure**: All AdOps projects use same directory layout
- **Semantic versioning**: MAJOR.MINOR.PATCH across all projects
- **Documentation patterns**: Setup guides, testing guides, field mapping, prompt history
- **Security practices**: Input variables for Airtable automations, no hard-coded secrets
- **Code quality**: Error handling, comprehensive logging, batch processing optimization
- **Universal identifiers**: Store IDs from ALL participating platforms

---

## Repository Overview

AdOps-Airtable is a collection of Airtable-focused automation tools, utilities, and integrations. This repository serves as the central location for:

- **Airtable API utilities** - Python and JavaScript tools for Airtable REST API
- **Shared automation libraries** - Reusable components for Airtable scripting
- **Cross-base synchronization** - Tools for syncing data between Airtable bases
- **Airtable automation patterns** - Production-ready script templates

---

## Project Structure

```
~/work/AdOps-Airtable/
├── CLAUDE.md                     # This file - repository configuration
├── README.md                     # Repository overview
├── .gitignore                    # Git ignore patterns
│
├── docs/                         # All documentation
│   ├── prompts/                  # Development history
│   │   └── AdOps-Airtable_PromptHistory.md
│   ├── api/                      # API documentation
│   ├── architecture/             # System design docs
│   └── patterns/                 # Shared implementation patterns
│       └── Logger_Implementation.md
│
├── scripts/                      # All automation scripts
│   ├── airtable/                 # Airtable automations
│   │   └── current/              # Production-ready scripts
│   ├── deployment/               # Deployment utilities
│   └── maintenance/              # Maintenance scripts
│
├── utility/                      # Tools, tests, and utilities
│   ├── testing/                  # Test scripts and data
│   ├── analysis/                 # Analysis and reporting tools
│   └── helpers/                  # Utility functions
│
├── deploy/                       # Deployment configurations
│   ├── aws/                      # AWS deployment files
│   ├── airtable/                 # Airtable deployment configs
│   └── environments/             # Environment-specific configs
│
├── data/                         # Data schemas and sample files
│   ├── schemas/                  # Airtable table schemas
│   ├── samples/                  # Sample data files
│   └── templates/                # Data templates
│
├── config/                       # Configuration files
│   ├── environments/             # Environment configs
│   ├── api/                      # API configurations
│   └── validation/               # Validation rules
│
├── shared/                       # Shared resources across sub-projects
│
└── deprecated/                   # Legacy code archives
```

---

## Quick Commands

### Repository-Level Commands

```bash
# Navigate to repository root
cd ~/work/AdOps-Airtable

# View all documentation
ls -la docs/

# Check git status
git status

# View recent commits
git log --oneline -10
```

### Development & Testing

```bash
# Validate JavaScript syntax
node -c scripts/airtable/current/*.js

# Run Python tests
python -m pytest utility/testing/

# Validate configuration files
python utility/testing/validate_configs.py
```

### Git Workflow

```bash
# Version increment commit pattern
git add -A
git commit -m "type: description (v1.2.3)

- Specific change 1
- Specific change 2
- Why: Business/technical reason for changes

🤖 Generated with [Claude Code](https://claude.ai/code)

Co-Authored-By: Claude <noreply@anthropic.com>"
git push
```

---

## Development Conventions

### Version Management

**MANDATORY: Include version in all commits affecting code:**
- **Commit message must include version number** `(v1.2.3)`
- **List specific changes** in commit body
- **Include WHY** the changes were made
- **Tag releases** for major versions: `git tag v1.0.0`

### Airtable Automation Standards

#### Version Header for All Scripts

```javascript
/**
 * Script Name
 * Description of functionality
 *
 * @version X.Y.Z - Brief description of changes
 * @release-date YYYY-MM-DD
 * @status Production Ready|Development|Testing
 * @deployment Airtable Automation Script
 *
 * CHANGELOG:
 * vX.Y.Z (YYYY-MM-DD) - Enhancement description
 *   - Specific change 1
 *   - Specific change 2
 *   - Why: Business/technical reason
 */

// Version constants - MANDATORY for all scripts
const SCRIPT_VERSION = "X.Y.Z";
const SCRIPT_NAME = "ScriptName";
const RELEASE_DATE = "YYYY-MM-DD";
const SCRIPT_STATUS = "Production Ready|Development|Testing";
```

#### Critical Airtable Automation Input Handling

**MANDATORY: All Airtable automation scripts MUST explicitly handle input variables**

**⚠️ CRITICAL RESTRICTION: input.config() can only be called ONCE per script execution**

Airtable enforces a strict limitation where `input.config()` can only be called once during script execution. Attempting to call it multiple times will result in the error: `"input.config may only be called once"`.

**REQUIRED Pattern:**

```javascript
// ✅ CORRECT: Single input.config() call at script initialization
const config = input.config();
const BASE_ID = config.base_id || "default_base_id";
const TABLE_ID = config.table_id || "default_table_id";
const API_TOKEN = config.api_token || "default_token";
const TRIGGER_RECORD_ID = config.recordId;  // Extract ALL values in one place

async function main() {
    try {
        if (TRIGGER_RECORD_ID) {
            // Single record mode
            await processSingleRecord(TRIGGER_RECORD_ID);
        } else {
            // Batch mode
            await processBatchRecords();
        }
    } catch (error) {
        console.error(`Script error: ${error.message}`);
        throw error;
    }
}

// ❌ WRONG: Multiple input.config() calls
async function main() {
    const config = input.config();  // First call
    const baseId = config.base_id;

    // ... later in code ...
    const recordId = input.config()?.recordId;  // ERROR: Second call fails!
}
```

**Why This Pattern is Critical:**
1. **Automation Compatibility** - Script works with record triggers, button triggers, and scheduled triggers
2. **Performance** - Single record mode avoids scanning entire tables
3. **Real-time Processing** - Immediate feedback when records are created/updated
4. **Deployment Flexibility** - Same script can be deployed in multiple trigger modes
5. **Error Prevention** - Prevents runtime failures when deployed as automation

### Logging Standards

**See:** [Logger Implementation](docs/patterns/Logger_Implementation.md) for complete Logger class with:
- Console.log based stdout logging (Airtable native)
- Console output buffering for permanent audit trail
- Request ID correlation across log entries
- API request/response logging with timing
- Automatic header sanitization
- Multiple log levels with emoji indicators
- Execution time tracking

**Quick Reference - Logger Initialization:**
```javascript
const logger = new Logger(SCRIPT_NAME, SCRIPT_VERSION, RELEASE_DATE, SCRIPT_STATUS);
logger.logStart();
```

**Quick Reference - API Logging:**
```javascript
// Log API request
logger.logApiRequest('Airtable', 'POST', url, headers, body);

// Log API response
const duration = Date.now() - startTime;
logger.logApiResponse('Airtable', response.status, responseData, duration);

// Log API error
logger.logApiError('Airtable', error, { operation: 'create', recordId: 'rec123' });
```

### File Naming Conventions

**MANDATORY: NO version numbers in filenames - use internal version variables instead**

- **Scripts**: Clean names without versions (e.g., `SyncScript.js`, not `SyncScript_v1.0.0.js`)
- **Version tracking**: In header comments and SCRIPT_VERSION constant
- **Environment suffix for non-production**: `script-dev.js`, `script-staging.js`
- **Production has NO suffix**: `script.js` (clean filename)

---

## Prompt History Documentation

**MANDATORY: Every project MUST maintain a complete prompt history document**

**File Location:** `docs/prompts/AdOps-Airtable_PromptHistory.md`

See parent CLAUDE.md conventions or EMG_Common for full template.

---

## CLAUDE.md Version History

### v1.0.0 (2025-12-15) - Initial Setup
- Created AdOps-Airtable repository structure
- Established EMG Common compliance
- Created standard directory layout following AdOps-TTD patterns
- Added Airtable automation best practices from CreativeAutomation
- Documented input.config() critical restriction
- Added logging standards reference
- Why: Establish central repository for Airtable-focused automation tools

---

*Last Updated: 2025-12-15*
*Claude Configuration Version: 1.0.0*
*Platform Family: AdOps Automation Platforms*
*Platform: Airtable API & Automation*
