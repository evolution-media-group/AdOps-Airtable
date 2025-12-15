# Airtable Comments & Chat System Reference
## API Capabilities, Limitations, and Documentation Sources

**Version:** 1.0.0
**Created:** 2025-12-15
**Purpose:** Comprehensive reference for Airtable comments/chat system capabilities and limitations

**Source:** AdOps-Airtable documentation
**Related:** [Comment Submission Workflow](../patterns/Comment_Submission_Workflow.md)

---

## Table of Contents

1. [Executive Summary](#executive-summary)
2. [Two Distinct "Chat" Concepts](#two-distinct-chat-concepts)
3. [Record Comments System](#record-comments-system)
4. [Omni AI Assistant](#omni-ai-assistant)
5. [API Endpoints Reference](#api-endpoints-reference)
6. [Rate Limits & Quotas](#rate-limits--quotas)
7. [Permissions & OAuth Scopes](#permissions--oauth-scopes)
8. [Critical Limitations](#critical-limitations)
9. [What Airtable Chat is NOT](#what-airtable-chat-is-not)
10. [Enterprise Features](#enterprise-features)
11. [Workarounds & Patterns](#workarounds--patterns)
12. [Documentation Sources](#documentation-sources)

---

## Executive Summary

Airtable has **two distinct "chat" concepts** that are often conflated:

| Concept | Purpose | API Support |
|---------|---------|-------------|
| **Record Comments** | Collaboration comments attached to records | Full CRUD via REST API |
| **Omni AI Assistant** | AI-powered conversational interface | No direct API (internal only) |

**Key Insight:** Neither is designed as a real-time messaging/chat platform like Slack or Teams. Airtable comments are record-level collaboration tools, not a messaging system.

---

## Two Distinct "Chat" Concepts

### 1. Record Comments

Traditional collaboration comments attached to individual records.

**Characteristics:**
- Attached to specific records (not tables or bases)
- Support threading via `parentCommentId`
- Support reactions (emoji) - read-only via API
- Support @mentions for notifications
- Visible in record expand view and activity feed

### 2. Omni AI Assistant

AI-powered conversational interface for building and querying Airtable.

**Characteristics:**
- Natural language app building
- Data analysis and querying
- Record creation/modification via conversation
- Credit-based usage system
- No external API access

---

## Record Comments System

### Available Features

| Feature | Description | API Support |
|---------|-------------|-------------|
| **Create Comments** | Add new comments to records | ✅ POST |
| **Read Comments** | List comments on a record | ✅ GET |
| **Update Comments** | Edit existing comments | ✅ PATCH |
| **Delete Comments** | Remove comments | ✅ DELETE |
| **Threading** | Reply to specific comments | ✅ `parentCommentId` |
| **Reactions** | Emoji reactions to comments | ❌ Read-only |
| **@Mentions** | Tag users in comments | ✅ `@[userId]` syntax |
| **Comment Watching** | Subscribe to thread notifications | ❌ No API |

### Comment Object Structure

```json
{
    "id": "comXXXXXXXXXXXXXX",
    "author": {
        "id": "usrXXXXXXXXXXXXXX",
        "email": "user@example.com",
        "name": "John Doe"
    },
    "text": "Comment text with @[usrYYYYYYYYYYYYYY] mention",
    "createdTime": "2025-12-15T10:32:00.000Z",
    "lastUpdatedTime": "2025-12-15T10:32:00.000Z",
    "parentCommentId": null,
    "reactions": [
        {
            "emoji": {
                "name": "thumbs_up",
                "unicode": "👍"
            },
            "reactingUser": {
                "id": "usrZZZZZZZZZZZZZZ",
                "email": "reactor@example.com",
                "name": "Jane Smith"
            }
        }
    ]
}
```

### User Mentions Format

To mention a user in a comment, use the syntax:
```
@[usrXXXXXXXXXXXXXX]
```

The user ID must be a valid collaborator ID in the base.

---

## Omni AI Assistant

### Capabilities

| Capability | Description | Credits |
|------------|-------------|---------|
| **App Building** | Create tables, fields, automations | FREE |
| **Q&A** | Query data conversationally | 10 credits |
| **Record Creation** | Create records via conversation | 10 credits |
| **Data Analysis** | Analyze and summarize data | Variable |
| **Web Research** | Pull external data (toggleable) | Variable |

### Credit Allocations by Plan

| Plan | Credits/User/Month | Cost |
|------|-------------------|------|
| Free | ~500 | $0 |
| Team | 500 | $20/user/month |
| Business | 500+ | Custom |
| Enterprise Scale | 25,000 | Custom |

### Omni Limitations

- **Permission Mirroring:** Omni can only access data/actions the user has permission for
- **Credits Don't Roll Over:** Unused credits expire at month end
- **No Conditional Visibility:** Cannot use interface conditional visibility rules
- **Credit Exhaustion:** AI features blocked when credits depleted
- **No External API:** Cannot be accessed programmatically

---

## API Endpoints Reference

### Base URL
```
https://api.airtable.com/v0/{baseId}/{tableIdOrName}/{recordId}/comments
```

### List Comments

```http
GET /v0/{baseId}/{tableId}/{recordId}/comments
```

**Query Parameters:**

| Parameter | Type | Description |
|-----------|------|-------------|
| `pageSize` | integer | Number of comments per page (max 100, default 100) |
| `offset` | string | Pagination offset from previous response |

**Response:**
```json
{
    "comments": [...],
    "offset": "nextPageToken"
}
```

### Create Comment

```http
POST /v0/{baseId}/{tableId}/{recordId}/comments
Content-Type: application/json
Authorization: Bearer {token}

{
    "text": "Comment text",
    "parentCommentId": "comXXX"  // Optional, for threading
}
```

**Response:**
```json
{
    "id": "comXXXXXXXXXXXXXX",
    "author": {...},
    "text": "Comment text",
    "createdTime": "2025-12-15T10:32:00.000Z"
}
```

### Update Comment

```http
PATCH /v0/{baseId}/{tableId}/{recordId}/comments/{commentId}
Content-Type: application/json
Authorization: Bearer {token}

{
    "text": "Updated comment text"
}
```

**Restriction:** Non-admin API users can only update their own comments.

### Delete Comment

```http
DELETE /v0/{baseId}/{tableId}/{recordId}/comments/{commentId}
Authorization: Bearer {token}
```

**Restriction:** Non-admin API users can only delete their own comments. Enterprise Admins can delete any comment.

---

## Rate Limits & Quotas

### API Rate Limits

| Limit Type | Value | Scope |
|------------|-------|-------|
| **Requests per second** | 5 | Per base |
| **Requests per second** | 50 | Per user/service account (all bases) |
| **Comments per page** | 100 | Default pagination |
| **Throttle recovery** | 30 seconds | After 429 error |

### Rate Limit Response

When rate limited, you receive:
```http
HTTP/1.1 429 Too Many Requests
Retry-After: 30
```

**Best Practice:** Implement exponential backoff with 30-second minimum wait.

### Automation Limits

| Limit | Value |
|-------|-------|
| Automations per base | 50 (including disabled) |
| Actions per automation | 25 |
| Script execution time | 30 seconds |

---

## Permissions & OAuth Scopes

### Required OAuth Scopes

| Scope | Purpose |
|-------|---------|
| `data.records:read` | Read records from tables |
| `data.records:write` | Create/update/delete records |
| `data.recordComments:read` | Read comments on records |
| `data.recordComments:write` | Create/update/delete comments |

### Permission Levels for Comments

| Role | Read | Create | Edit Own | Edit Any | Delete Own | Delete Any |
|------|------|--------|----------|----------|------------|------------|
| Owner/Creator | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
| Editor | ✅ | ✅ | ✅ | ❌ | ✅ | ❌ |
| Commenter | ✅ | ✅ | ✅ | ❌ | ✅ | ❌ |
| Read-only | ✅ | ❌ | ❌ | ❌ | ❌ | ❌ |
| Enterprise Admin | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |

### Interface-Only Collaborator Restrictions

- Cannot be automatically set to watch/follow comments
- Only receive notifications for mentions created in interfaces (not base layer)
- Tagging from form submissions does NOT set them to watch the record

---

## Critical Limitations

### 1. Comments Cannot Trigger Automations

**Impact:** HIGH
**Description:** Comments are treated as metadata and cannot trigger Airtable automations.

```
❌ Trigger: "When comment is added" - DOES NOT EXIST
```

**Workaround:** Use the [Comment Submission Workflow](../patterns/Comment_Submission_Workflow.md) pattern.

### 2. No "List All Comments" Endpoint

**Impact:** HIGH
**Description:** Must query comments per-record. No endpoint to get all comments across a table or base.

```
❌ GET /v0/{baseId}/{tableId}/comments - DOES NOT EXIST
```

**Workaround:** Poll individual records or use a Comment_Log table to aggregate.

### 3. Reactions Are Read-Only

**Impact:** MEDIUM
**Description:** API can read reactions but cannot create them.

```javascript
// ✅ CAN READ
GET /comments → { reactions: [{emoji: "👍", ...}] }

// ❌ CANNOT CREATE
POST /comments/reactions - DOES NOT EXIST
```

**Workaround:** Store custom reactions in a separate table.

### 4. No Admin Control Over Notifications

**Impact:** MEDIUM
**Description:** Administrators cannot manage notification settings for users. Each user must configure their own per-record.

### 5. Comment Summarization Restrictions

**Impact:** LOW
**Description:**
- Only entire comment feed can be summarized (not subsets)
- API cannot pull metadata of summarized comments
- "Prevent external users from seeing each other" disables summarization

### 6. Pagination Complexity for Threads

**Impact:** LOW
**Description:** Threaded replies may not appear on the same page as their parent comment.

### 7. Interface-Only User Isolation

**Impact:** MEDIUM
**Description:** Comments/mentions made in base layer don't notify interface-only users. Separate notification systems for base vs. interface.

---

## What Airtable Chat is NOT

Based on official documentation and community feedback:

| Use Case | Airtable Suitability | Reason |
|----------|---------------------|--------|
| Real-time messaging | ❌ Not suitable | No WebSocket/push support |
| Multi-threaded support chat | ❌ Not suitable | Cannot handle concurrent conversations |
| Live customer support | ❌ Not suitable | No real-time interaction capabilities |
| Slack/Teams replacement | ❌ Not suitable | Different architecture and purpose |
| Ticket system with chat | ⚠️ Limited | Can track tickets, not live chat |

**Official Guidance:**
> "Airtable is not best suited for use as a messaging app, and that's not what it's designed for. That's what apps like Slack are designed for, and Airtable can be integrated with Slack in various ways."

---

## Enterprise Features

### Enterprise-Only Comment Features

| Feature | Description |
|---------|-------------|
| **Comment Deletion by Admins** | Enterprise admins can delete any comment |
| **Collaborator Visibility** | Hide user info for privacy (comments redacted but visible) |
| **25,000 AI Credits/User** | Significantly higher than Team/Business plans |
| **Domain Verification** | Required for full admin panel functionality |
| **Workspace Sharing Restrictions** | Prevent unauthorized collaborator invitations |

### Privacy Setting Implications

When **"Prevent external users from seeing each other"** is enabled:
- Comment mentions are redacted
- Comment authors are redacted
- Actual comment text remains visible
- **AI summarization is disabled**

---

## Workarounds & Patterns

### For Automation Triggers

Use the [Comment Submission Workflow](../patterns/Comment_Submission_Workflow.md):
1. Capture comments via form submission
2. Create native comment via API
3. Log to Comment_Log table (triggers automation)
4. Send to external webhook

### For Real-Time Notifications

```javascript
// Automation workaround: Email/Slack on field update
// Trigger: When record updated
// Condition: Check if notification field changed
// Action: Send email or Slack message with record link
```

### For Comment Monitoring

Options ranked by efficiency:
1. **Form-based submission** - Event-driven, no polling
2. **View-filtered polling** - Poll only active records
3. **Registration queue** - Users subscribe records to monitoring
4. **Full table polling** - Only for small tables (<100 records)

### For Reactions

Store in separate `Comment_Reactions` table:
```
Comment_Reactions
├── Comment (link to Comment_Log)
├── Emoji (single select: 👍👎❤️😄🎉👀)
├── Reacted_By (collaborator)
└── Reacted_At (created time)
```

### Third-Party Integration Tools

| Tool | Comment Support |
|------|-----------------|
| Make (Integromat) | ✅ Can create comments via HTTP module |
| Zapier | ✅ Comment creation capability |
| n8n | ✅ Via HTTP request node |
| Custom scripts | ✅ Direct API calls |

---

## Documentation Sources

### Official Airtable Documentation

| Resource | URL | Description |
|----------|-----|-------------|
| **Web API Introduction** | https://airtable.com/developers/web/api/introduction | Main API reference guide |
| **Comments API - List** | https://airtable.com/developers/web/api/list-comments | List comments endpoint |
| **API Usage Guide** | https://www.airtable.com/guides/scale/using-airtable-api | Tutorials and best practices |
| **Getting Started with API** | https://support.airtable.com/docs/getting-started-with-airtables-web-api | Help articles on API usage |
| **Commenting in Airtable** | https://support.airtable.com/docs/commenting-in-airtable | Official commenting documentation |
| **Managing API Call Limits** | https://support.airtable.com/docs/managing-api-call-limits-in-airtable | Rate limit documentation |
| **Airtable Automations** | https://support.airtable.com/docs/getting-started-with-airtable-automations | Automation documentation |
| **Managing Notifications** | https://support.airtable.com/docs/managing-airtable-notifications | Notification settings |
| **Permissions Overview** | https://support.airtable.com/v1/docs/airtable-permissions-overview | Permission levels |
| **Interface Designer Permissions** | https://support.airtable.com/docs/interface-designer-permissions | Interface-specific permissions |

### Omni AI Documentation

| Resource | URL | Description |
|----------|-----|-------------|
| **Airtable AI Platform** | https://www.airtable.com/platform/ai | AI features overview |
| **Using Omni AI** | https://support.airtable.com/docs/using-omni-ai-in-airtable | Omni usage guide |
| **AI Billing** | https://support.airtable.com/docs/airtable-ai-billing | Credit system documentation |
| **Cobuilder Launch** | https://blog.airtable.com/airtable-cobuilder-launch/ | Cobuilder announcement |

### Community & Discussion

| Resource | URL | Description |
|----------|-----|-------------|
| **Airtable Community** | https://community.airtable.com/ | Official user forum |
| **Comments Update Announcement** | https://community.airtable.com/announcements-6/updates-to-comments-reactions-threads-and-comment-watching-1545 | Threading/reactions announcement |
| **Better Chat Function Request** | https://community.airtable.com/t5/other-questions/better-chat-function-on-individual-records/td-p/80080 | Community feedback |
| **API Access to Comments** | https://community.airtable.com/t5/development-apis/api-access-to-record-comments/td-p/90947 | API discussion |
| **AI Credits Guide** | https://community.airtable.com/announcements-6/understanding-your-ai-credits-real-world-examples-usage-45968 | Credit usage examples |
| **Airtable Reddit** | https://www.reddit.com/r/Airtable/ | Reddit community |

### Integration Documentation

| Resource | URL | Description |
|----------|-----|-------------|
| **Slack Integration** | https://support.airtable.com/docs/airtable-automation-actions-slack | Slack automation actions |
| **Twilio Integration** | https://www.airtable.com/integrations/twilio | Communication workflows |

### Third-Party Analysis

| Resource | URL | Description |
|----------|-----|-------------|
| **Airtable Permissions Guide** | https://www.softr.io/blog/airtable-permissions | Comprehensive permissions analysis |
| **Airtable AI Review 2025** | https://reply.io/blog/airtable-ai-review/ | AI features review |
| **AI Agents Pricing Guide** | https://o-mega.ai/articles/airtable-agents-pricing-what-does-it-cost-you-2025 | Pricing breakdown |
| **Realtime Messages with Airtable** | https://ably.com/blog/airtable-database-realtime-messages | Real-time patterns |

---

## Quick Reference Card

### Can Do ✅

- Create, read, update, delete comments via API
- Create threaded replies with `parentCommentId`
- @mention users in comment text
- Read reactions on existing comments
- Paginate through comments (100 per page)
- Authenticate via Personal Access Token or OAuth

### Cannot Do ❌

- Trigger automations from new comments
- Create reactions via API
- List all comments across a table/base
- Control notification settings for other users
- Access Omni AI via external API
- Use comments for real-time chat/messaging

### Rate Limits

- 5 requests/second per base
- 50 requests/second per user (all bases)
- 30 second wait after 429 error
- 100 comments per page maximum

### Required Scopes

```
data.recordComments:read   # For reading
data.recordComments:write  # For creating/editing/deleting
```

---

*Document Version: 1.0.0*
*Last Updated: 2025-12-15*
*Status: Reference Documentation*
