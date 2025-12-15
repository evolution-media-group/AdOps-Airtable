# Comment Submission Workflow Pattern
## Bypassing Native Airtable Comment Limitations

**Version:** 1.0.0
**Created:** 2025-12-15
**Purpose:** Enable automation triggers and external API integration for Airtable comments

**Source:** AdOps-Airtable pattern library
**Related:** [Logger Implementation](Logger_Implementation.md)

---

## Problem Statement

Airtable's native comments have significant limitations:
- **Comments cannot trigger automations** - They're treated as metadata
- **No "list all comments" endpoint** - Must query per-record
- **Polling is inefficient** - High API cost for large tables
- **No webhook support** - Cannot push comment events to external systems

This pattern solves these limitations by creating a **comment submission workflow** that:
1. Captures comments via a form
2. Creates native Airtable comments via API
3. Logs copies for automated processing
4. Triggers external webhooks in real-time

---

## Architecture Overview

```
┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐
│  Airtable Form  │────▶│   Automation    │────▶│  Comments API   │
│  (User submits) │     │   (Triggered)   │     │  (Create)       │
└─────────────────┘     └────────┬────────┘     └─────────────────┘
                                 │
                                 ▼
                        ┌─────────────────┐     ┌─────────────────┐
                        │  Comment_Log    │────▶│  Your API /     │
                        │  Table (copy)   │     │  Webhook        │
                        └─────────────────┘     └─────────────────┘
```

---

## Implementation

### Step 1: Create Comment Submission Table

Create a table to capture comment submissions from forms.

**Table Name:** `Comment_Submissions`

| Field Name | Type | Purpose |
|------------|------|---------|
| `Target_Record` | Link to another record | Record to attach comment to |
| `Target_Table` | Single Select | Which table (if multi-table) |
| `Comment_Text` | Long Text | The actual comment content |
| `Submitted_By` | Collaborator | Auto-captured from form |
| `Submitted_At` | Created Time | Timestamp |
| `Status` | Single Select | Pending/Posted/Failed |
| `API_Response` | Long Text | Store API result for debugging |
| `Airtable_Comment_ID` | Text | Returned comment ID from API |

**Single Select Options for `Status`:**
- `Pending` - Just submitted, awaiting processing
- `Posted` - Successfully posted to Airtable
- `Failed` - Error occurred during posting

**Single Select Options for `Target_Table`:**
- Add each table name that can receive comments (e.g., `Projects`, `Campaigns`, `Tasks`)

---

### Step 2: Create the Form

In Airtable Interface Designer or Form view, create a comment submission form:

```
┌─────────────────────────────────────────────────────────┐
│              Add Comment                                │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  Select Record: [Dropdown - linked records]             │
│                                                         │
│  Your Comment:                                          │
│  ┌─────────────────────────────────────────────────┐   │
│  │                                                 │   │
│  │                                                 │   │
│  └─────────────────────────────────────────────────┘   │
│                                                         │
│                              [ Submit Comment ]         │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

**Form Configuration:**
- Show `Target_Record` field (required)
- Show `Target_Table` field (required if multi-table)
- Show `Comment_Text` field (required)
- Hide all other fields (auto-populated)

---

### Step 3: Automation Script - Process Submission

**Automation Configuration:**
- **Trigger:** When record is created in `Comment_Submissions`
- **Action:** Run script

```javascript
/**
 * Comment Submission Processor
 *
 * @version 1.0.0
 * @release-date 2025-12-15
 * @status Production Ready
 * @deployment Airtable Automation Script
 *
 * TRIGGERS: When record created in Comment_Submissions
 * ACTIONS:
 *   1. Create comment via Airtable API
 *   2. Copy to Comment_Log for processing
 *   3. Update submission status
 *
 * INPUT VARIABLES REQUIRED:
 *   - recordId: The trigger record ID
 *   - apiToken: Airtable API token (Personal Access Token)
 *   - baseId: Current base ID
 */

// === CONFIGURATION ===
// CRITICAL: input.config() can only be called ONCE
const config = input.config();
const SUBMISSION_RECORD_ID = config.recordId;
const API_TOKEN = config.apiToken;
const BASE_ID = config.baseId;

// Table IDs for comment targets - UPDATE THESE FOR YOUR BASE
const TABLE_MAP = {
    'Projects': 'tblXXXXXXXXXXXXXX',
    'Campaigns': 'tblYYYYYYYYYYYYYY',
    'Tasks': 'tblZZZZZZZZZZZZZZ'
};

// === MAIN EXECUTION ===
async function main() {
    const submissionsTable = base.getTable('Comment_Submissions');
    const commentLogTable = base.getTable('Comment_Log');

    // Get the submission record
    const submission = await submissionsTable.selectRecordAsync(SUBMISSION_RECORD_ID);

    if (!submission) {
        console.error('Submission record not found');
        return;
    }

    const targetRecordLinks = submission.getCellValue('Target_Record');
    const targetTableName = submission.getCellValueAsString('Target_Table');
    const commentText = submission.getCellValueAsString('Comment_Text');
    const submittedBy = submission.getCellValue('Submitted_By');

    // Validate inputs
    if (!targetRecordLinks || targetRecordLinks.length === 0) {
        await updateStatus(submissionsTable, SUBMISSION_RECORD_ID, 'Failed', 'No target record selected');
        return;
    }

    if (!commentText || commentText.trim() === '') {
        await updateStatus(submissionsTable, SUBMISSION_RECORD_ID, 'Failed', 'Comment text is empty');
        return;
    }

    const targetRecordId = targetRecordLinks[0].id;
    const targetTableId = TABLE_MAP[targetTableName];

    if (!targetTableId) {
        await updateStatus(submissionsTable, SUBMISSION_RECORD_ID, 'Failed', `Unknown table: ${targetTableName}`);
        return;
    }

    try {
        // === Step 1: Create comment via Airtable API ===
        console.log(`Creating comment on ${targetTableName}/${targetRecordId}`);

        const commentResult = await createAirtableComment(
            BASE_ID,
            targetTableId,
            targetRecordId,
            commentText,
            API_TOKEN
        );

        console.log(`Comment created: ${commentResult.id}`);

        // === Step 2: Copy to Comment_Log for automated processing ===
        const logRecord = await commentLogTable.createRecordAsync({
            'Source_Record_ID': targetRecordId,
            'Source_Table': { name: targetTableName },
            'Comment_Text': commentText,
            'Comment_ID': commentResult.id,
            'Author_Name': submittedBy ? submittedBy[0].name : 'Unknown',
            'Author_Email': submittedBy ? submittedBy[0].email : null,
            'Created_At': new Date().toISOString(),
            'Processing_Status': { name: 'Pending' },
            'Submission_ID': SUBMISSION_RECORD_ID
        });

        console.log(`Logged to Comment_Log: ${logRecord}`);

        // === Step 3: Update submission status ===
        await updateStatus(
            submissionsTable,
            SUBMISSION_RECORD_ID,
            'Posted',
            JSON.stringify(commentResult, null, 2),
            commentResult.id
        );

        console.log('Comment submission processed successfully');

    } catch (error) {
        console.error(`Error processing comment: ${error.message}`);
        await updateStatus(
            submissionsTable,
            SUBMISSION_RECORD_ID,
            'Failed',
            error.message
        );
    }
}

// === HELPER FUNCTIONS ===

async function createAirtableComment(baseId, tableId, recordId, text, apiToken) {
    const url = `https://api.airtable.com/v0/${baseId}/${tableId}/${recordId}/comments`;

    const response = await fetch(url, {
        method: 'POST',
        headers: {
            'Authorization': `Bearer ${apiToken}`,
            'Content-Type': 'application/json'
        },
        body: JSON.stringify({ text })
    });

    if (!response.ok) {
        const errorData = await response.json();
        throw new Error(`API Error ${response.status}: ${JSON.stringify(errorData)}`);
    }

    return await response.json();
}

async function updateStatus(table, recordId, status, apiResponse = null, commentId = null) {
    const updates = {
        'Status': { name: status }
    };

    if (apiResponse) {
        updates['API_Response'] = apiResponse;
    }

    if (commentId) {
        updates['Airtable_Comment_ID'] = commentId;
    }

    await table.updateRecordAsync(recordId, updates);
}

// Execute
await main();
```

**Input Variables Configuration:**

| Variable Name | Type | Source |
|---------------|------|--------|
| `recordId` | Record ID | Trigger record |
| `apiToken` | Text | Input variable (secure) |
| `baseId` | Text | Input variable |

---

### Step 4: Create Comment_Log Table

This table stores copies of all comments for automated processing.

**Table Name:** `Comment_Log`

| Field Name | Type | Purpose |
|------------|------|---------|
| `Source_Record_ID` | Text | Record the comment was added to |
| `Source_Table` | Single Select | Which table |
| `Comment_Text` | Long Text | Full comment content |
| `Comment_ID` | Text | Airtable comment ID |
| `Author_Name` | Text | Who submitted |
| `Author_Email` | Email | Author's email |
| `Created_At` | DateTime | When submitted |
| `Processing_Status` | Single Select | Pending/Processed/Sent/Failed |
| `Submission_ID` | Text | Link back to submission |
| `External_API_Response` | Long Text | Response from your webhook |
| `Processed_At` | DateTime | When processed |

**Single Select Options for `Processing_Status`:**
- `Pending` - Awaiting external processing
- `Processed` - Successfully processed internally
- `Sent` - Sent to external webhook
- `Failed` - External processing failed

---

### Step 5: Second Automation - Process Comment_Log

**Automation Configuration:**
- **Trigger:** When record is created in `Comment_Log`
- **Action:** Run script

```javascript
/**
 * Comment Log Processor
 *
 * @version 1.0.0
 * @release-date 2025-12-15
 * @status Production Ready
 * @deployment Airtable Automation Script
 *
 * TRIGGERS: When record created in Comment_Log
 * ACTIONS: Send to external API/webhook for processing
 *
 * INPUT VARIABLES REQUIRED:
 *   - recordId: The trigger record ID
 *   - webhookUrl: Your external API endpoint
 */

// CRITICAL: input.config() can only be called ONCE
const config = input.config();
const LOG_RECORD_ID = config.recordId;
const WEBHOOK_URL = config.webhookUrl;

async function main() {
    const commentLogTable = base.getTable('Comment_Log');
    const logRecord = await commentLogTable.selectRecordAsync(LOG_RECORD_ID);

    if (!logRecord) {
        console.error('Log record not found');
        return;
    }

    const payload = {
        commentId: logRecord.getCellValueAsString('Comment_ID'),
        sourceRecordId: logRecord.getCellValueAsString('Source_Record_ID'),
        sourceTable: logRecord.getCellValueAsString('Source_Table'),
        commentText: logRecord.getCellValueAsString('Comment_Text'),
        authorName: logRecord.getCellValueAsString('Author_Name'),
        authorEmail: logRecord.getCellValueAsString('Author_Email'),
        createdAt: logRecord.getCellValueAsString('Created_At'),
        airtableLogRecordId: LOG_RECORD_ID
    };

    console.log(`Sending comment to webhook: ${payload.commentId}`);

    try {
        const response = await fetch(WEBHOOK_URL, {
            method: 'POST',
            headers: {
                'Content-Type': 'application/json'
            },
            body: JSON.stringify(payload)
        });

        const responseData = await response.text();

        await commentLogTable.updateRecordAsync(LOG_RECORD_ID, {
            'Processing_Status': { name: response.ok ? 'Sent' : 'Failed' },
            'External_API_Response': responseData,
            'Processed_At': new Date().toISOString()
        });

        console.log(`Webhook response: ${response.status}`);

    } catch (error) {
        console.error(`Webhook error: ${error.message}`);

        await commentLogTable.updateRecordAsync(LOG_RECORD_ID, {
            'Processing_Status': { name: 'Failed' },
            'External_API_Response': error.message
        });
    }
}

await main();
```

**Input Variables Configuration:**

| Variable Name | Type | Source |
|---------------|------|--------|
| `recordId` | Record ID | Trigger record |
| `webhookUrl` | Text | Input variable |

---

## Complete Flow Diagram

```
User fills form
       │
       ▼
┌──────────────────┐
│ Comment_Submissions │ ◄─── Record created
└────────┬─────────┘
         │
         ▼
┌──────────────────┐     ┌──────────────────┐
│ Automation #1    │────▶│ Airtable         │
│ (Process Submit) │     │ Comments API     │
└────────┬─────────┘     │ POST /comments   │
         │               └──────────────────┘
         │
         ▼
┌──────────────────┐
│ Comment_Log      │ ◄─── Record created with copy
└────────┬─────────┘
         │
         ▼
┌──────────────────┐     ┌──────────────────┐
│ Automation #2    │────▶│ Your External    │
│ (Process Log)    │     │ API / Webhook    │
└──────────────────┘     └──────────────────┘
```

---

## Enhanced Version: Interface with Button

For a better UX, use an Interface with inline commenting on record detail pages.

**Use Case:** Users can add comments directly from a record without navigating to a form.

```javascript
/**
 * Interface Button Script - Add Comment
 *
 * @version 1.0.0
 * @release-date 2025-12-15
 * @status Production Ready
 * @deployment Airtable Interface Button Script
 *
 * PLACEMENT: Record detail page in Interface Designer
 *
 * INPUT VARIABLES REQUIRED:
 *   - recordId: Current record ID
 *   - tableId: Current table ID
 *   - baseId: Current base ID
 *   - apiToken: Airtable API token
 *   - currentUserId: Current user's collaborator ID
 */

// CRITICAL: input.config() can only be called ONCE
const config = input.config();
const CURRENT_RECORD_ID = config.recordId;
const TABLE_ID = config.tableId;
const BASE_ID = config.baseId;
const API_TOKEN = config.apiToken;

// Prompt user for comment
const commentText = await input.textAsync('Enter your comment:');

if (!commentText || commentText.trim() === '') {
    output.markdown('⚠️ **No comment entered**');
    return;
}

// Get current user info
const currentUser = base.activeCollaborators.find(
    c => c.id === config.currentUserId
);

try {
    // Create comment via API
    const response = await fetch(
        `https://api.airtable.com/v0/${BASE_ID}/${TABLE_ID}/${CURRENT_RECORD_ID}/comments`,
        {
            method: 'POST',
            headers: {
                'Authorization': `Bearer ${API_TOKEN}`,
                'Content-Type': 'application/json'
            },
            body: JSON.stringify({ text: commentText })
        }
    );

    const result = await response.json();

    if (response.ok) {
        // Log to Comment_Log table
        const commentLogTable = base.getTable('Comment_Log');
        await commentLogTable.createRecordAsync({
            'Source_Record_ID': CURRENT_RECORD_ID,
            'Comment_Text': commentText,
            'Comment_ID': result.id,
            'Author_Name': currentUser?.name || 'Unknown',
            'Author_Email': currentUser?.email || null,
            'Created_At': new Date().toISOString(),
            'Processing_Status': { name: 'Pending' }
        });

        output.markdown(`✅ **Comment posted successfully!**\n\n> ${commentText}`);
    } else {
        output.markdown(`❌ **Failed to post comment**\n\n\`${JSON.stringify(result)}\``);
    }

} catch (error) {
    output.markdown(`❌ **Error:** ${error.message}`);
}
```

---

## Advantages of This Approach

| Benefit | Description |
|---------|-------------|
| **Automation triggers** | Form submission triggers automation (comments alone cannot) |
| **Complete audit trail** | Every comment logged in queryable table |
| **External integration** | Easy webhook to any external system |
| **No polling needed** | Event-driven, real-time processing |
| **User attribution** | Capture who submitted via form |
| **Error handling** | Track failed submissions, retry capability |
| **Searchable** | Comment_Log is fully searchable/filterable |
| **Analytics** | Can report on comment volume, response times |

---

## Comparison: Native Comments vs Form-Based

| Feature | Native Comments | Form-Based |
|---------|-----------------|------------|
| Triggers automation | ❌ No | ✅ Yes |
| Real-time | ✅ Yes | ✅ Yes |
| Threading | ✅ Yes | ❌ No (flat) |
| Reactions | ✅ Yes | ❌ No |
| @mentions | ✅ Yes | ⚠️ Manual |
| Searchable | ❌ Limited | ✅ Full |
| External API | ❌ Polling only | ✅ Direct |
| Audit trail | ❌ No | ✅ Yes |
| Analytics | ❌ Yes | ✅ Yes |

---

## OAuth Scopes Required

If using OAuth authentication instead of Personal Access Tokens:

| Scope | Purpose |
|-------|---------|
| `data.records:read` | Read records from tables |
| `data.records:write` | Create records in Comment_Log |
| `data.recordComments:write` | Create comments via API |

---

## Security Considerations

1. **API Token Storage**: Store the API token as an Airtable automation input variable, not in the script
2. **Webhook Authentication**: Add authentication headers to your webhook endpoint
3. **Input Validation**: The script validates inputs before API calls
4. **Error Logging**: Failed submissions are tracked with error details

---

## Troubleshooting

### Comment Not Created

**Symptom:** Submission shows "Failed" status

**Check:**
1. Verify `TABLE_MAP` contains correct table IDs
2. Confirm API token has `data.recordComments:write` scope
3. Check `API_Response` field for error details

### Webhook Not Receiving

**Symptom:** Comment_Log shows "Failed" for Processing_Status

**Check:**
1. Verify webhook URL is correct and accessible
2. Check `External_API_Response` for error details
3. Ensure webhook accepts POST with JSON body

### Automation Not Triggering

**Symptom:** Records created but no processing occurs

**Check:**
1. Verify automation is enabled
2. Check automation run history for errors
3. Confirm trigger is set to "When record is created"

---

## Related Documentation

- [Airtable Comments API](https://airtable.com/developers/web/api/list-comments)
- [Airtable Automations](https://support.airtable.com/docs/getting-started-with-airtable-automations)
- [Logger Implementation](Logger_Implementation.md) - Standard logging pattern

---

*Document Version: 1.0.0*
*Last Updated: 2025-12-15*
*Pattern Status: Production Ready*
