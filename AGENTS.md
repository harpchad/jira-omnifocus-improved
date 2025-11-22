# AI Agent Context: Jira-OmniFocus Plugin

This document provides context for AI coding agents working on the jira-omnifocus-improved project.

## Project Overview

**Type**: OmniFocus plugin (JavaScript)  
**Purpose**: Sync Jira issues to OmniFocus tasks  
**Runtime**: OmniFocus Automation/Omni Automation (JavaScript for Automation)  
**Platform**: macOS, iOS, iPadOS  

## Key Constraints

### JavaScript Environment

This code runs in OmniFocus's JavaScript for Automation environment, which has important limitations:

1. **NO `const` or `let` at plugin level** - Use `var` only
   - `const`/`let` work inside functions but cause "uninitialized variable" errors at top level
   - This is a quirk of the OmniFocus JavaScript engine

2. **Traditional function syntax preferred** - Arrow functions can cause scoping issues
   ```javascript
   // Prefer this:
   function processNode(node) { }
   
   // Over this (can cause issues):
   const processNode = (node) => { }
   ```

3. **Avoid `forEach` with complex scoping** - Use traditional `for` loops
   ```javascript
   // Prefer this:
   for (var i = 0; i < items.length; i++) { }
   
   // Over this (can cause issues):
   items.forEach(item => { })
   ```

### API Constraints

1. **Jira Cloud API v3** - Uses `/rest/api/3/search/jql` endpoint
   - **MUST include `fields` parameter** or response only contains IDs
   - Example: `?jql=query&fields=key,summary,description,duedate`

2. **Atlassian Document Format (ADF)** - Jira descriptions use structured format
   - Not plain text strings
   - Must recursively extract text from nested content nodes
   - See `extractTextFromADF()` function

3. **OmniFocus Credentials API** - Secure storage in macOS Keychain
   ```javascript
   var credentials = new Credentials();  // Must be at top level
   var credential = credentials.read(credentialKey);
   credentials.write(credentialKey, user, password);
   credentials.remove(credentialKey);
   ```

## Architecture

### Core Flow

1. **Credential Management**
   - Check for existing credentials in Keychain
   - Prompt user if missing
   - Store securely (never in plugin file)

2. **API Request**
   - Build JQL query with optional fields (summary, description, duedate)
   - Authenticate with Basic Auth (username + API token)
   - Fetch issues from Jira Cloud API v3

3. **Task Synchronization**
   - **Existing tasks**: Mark incomplete, update due dates if enabled
   - **New tasks**: Create in OmniFocus inbox with tag, note, optional due date
   - **Removed tasks**: Mark complete if no incomplete children

4. **Error Handling**
   - HTTP errors with specific messages (401, 403, 404, 410)
   - Auto-clear invalid credentials on 401
   - JSON parsing errors
   - Date parsing errors

### Key Functions

#### `extractTextFromADF(doc)`
Recursively extracts plain text from Atlassian Document Format.

**Input**: ADF object `{ type: 'doc', content: [...] }`  
**Output**: Plain text string with paragraph breaks  
**Purpose**: Convert Jira's rich text descriptions to plain text for OmniFocus notes

#### Main Action Flow
1. Retrieve/prompt credentials
2. Build API URL with fields parameter
3. Fetch issues from Jira
4. Parse response (handle both old string and new ADF formats)
5. Sync tasks (create/update/complete)
6. Show success/failure alert

## Configuration Options

```javascript
var jiraUrl = "https://company.atlassian.net";  // Jira instance URL
var credentialKey = "jira-default";              // Keychain identifier
var omnifocusTagToUse = "jira";                  // Tag for synced tasks
var jiraQuery = "assignee=currentuser() and resolution is empty";  // JQL
var processComments = false;                     // Comment sync (legacy, disabled)
var omnifocusCommentTagToUse = "Jira-Comment";  // Comment tag (legacy)
var syncDueDates = false;                        // Sync Jira due dates to OmniFocus
```

### syncDueDates Behavior

When `true`:
- Requests `duedate` field from API
- Sets `newTask.dueDate` for new tasks
- Updates `existingTask.dueDate` for existing tasks
- Clears `dueDate` if Jira issue has none

When `false` (default):
- Doesn't request duedate field (slightly faster API calls)
- No due date operations

## Common Issues & Solutions

### Issue: Tasks showing only numbers (IDs)
**Cause**: Missing `fields` parameter in API request  
**Solution**: Always include `&fields=key,summary,description` (plus duedate if enabled)

### Issue: Descriptions are `[object Object]`
**Cause**: ADF format not being parsed  
**Solution**: Check if `description.type === 'doc'` and use `extractTextFromADF()`

### Issue: "Cannot access uninitialized variable"
**Cause**: Using `const` or `let` at plugin top level  
**Solution**: Change to `var` declarations

### Issue: HTTP 401 errors
**Cause**: Invalid/expired API token  
**Solution**: Plugin auto-clears credentials, prompts user to re-enter

### Issue: Due dates not syncing
**Cause**: `syncDueDates` is `false` or duedate field not requested  
**Solution**: Set `syncDueDates = true` and verify fields parameter includes `duedate`

## Testing Guidelines

### Test Scenarios

1. **First Run**
   - No credentials → Should prompt
   - Save credentials → Should store in Keychain
   - Run again → Should not re-prompt

2. **API Connectivity**
   - Valid credentials → Success alert with counts
   - Invalid credentials → 401 error, credentials cleared, user prompted
   - Network error → Clear error message

3. **Task Creation**
   - New Jira issue → Creates OmniFocus task
   - Task name format: `PROJECT-123 Issue Summary`
   - Task note: URL + description (plain text from ADF)
   - Task tags: Includes configured tag

4. **Task Updates**
   - Existing task → Marks incomplete
   - With `syncDueDates=true` → Updates due date
   - Jira issue removed → Marks complete (if no children)

5. **Due Date Syncing** (when enabled)
   - Jira issue with due date → OmniFocus task gets due date
   - Existing task, date changed → OmniFocus task updated
   - Jira due date removed → OmniFocus due date cleared

### Debug Logging

Console.app shows debug output:
```
DEBUG: Fetching URL: ...
DEBUG: User: ...
DEBUG: Response status: 200
DEBUG: Response MIME type: application/json
DEBUG: Response keys: issues, isLast
DEBUG: Found 5 issues
DEBUG: First issue keys: id, key, self, fields
DEBUG: Set due date for PROJECT-123: Fri Nov 22 2025...
```

## Code Style

- **Variables**: Use `var` at top level, descriptive names
- **Constants**: Uppercase with underscores (e.g., `HTTP_UNAUTHORIZED`)
- **Error handling**: Try-catch with specific error messages
- **Logging**: Use `console.log()` for DEBUG, `console.error()` for ERROR
- **Alerts**: User-friendly messages with actionable guidance

## File Structure

```
jira.omnifocusjs              # Main plugin file
├── Metadata (lines 1-10)     # Plugin info, version, identifier
├── Helper Functions          # extractTextFromADF()
├── Action Function           # Main sync logic
│   ├── Configuration        # User-editable settings
│   ├── Constants            # API paths, HTTP codes
│   ├── Credential Mgmt      # Keychain operations
│   ├── API Request          # Build URL, fetch issues
│   ├── Response Parsing     # Handle Jira API response
│   ├── Task Sync            # Create/update/complete tasks
│   └── Error Handling       # Catch and display errors
└── Validation Function       # Always return true
```

## API Reference

### Jira Cloud API v3

**Endpoint**: `GET /rest/api/3/search/jql`  
**Auth**: Basic (email + API token)  
**Query Params**:
- `jql`: JQL query string (URL encoded)
- `fields`: Comma-separated field list (REQUIRED)

**Response**:
```json
{
  "issues": [
    {
      "id": "12345",
      "key": "PROJECT-123",
      "fields": {
        "summary": "Issue title",
        "description": {
          "type": "doc",
          "content": [...]  // ADF format
        },
        "duedate": "2025-11-30"  // ISO 8601 date
      }
    }
  ],
  "isLast": true
}
```

### OmniFocus Automation API

**Task Creation**:
```javascript
var task = new Task(name, inbox.beginning);
task.addTag(tag);
task.note = "text";
task.dueDate = new Date("2025-11-30");
```

**Task Modification**:
```javascript
existingTask.markIncomplete();
existingTask.markComplete();
existingTask.dueDate = null;  // Clear due date
```

**Tags**:
```javascript
var tag = tags.byName("tagName") || new Tag("tagName");
```

## Version History

- **v2.3**: Added due date syncing feature
- **v2.2**: Migrated to API v3, added ADF parser, fixed field selection
- **v2.0**: Security improvements, Keychain storage, error handling

## Resources

- [Omni Automation Docs](https://omni-automation.com/omnifocus/)
- [Jira Cloud REST API v3](https://developer.atlassian.com/cloud/jira/platform/rest/v3/)
- [JQL Reference](https://support.atlassian.com/jira-service-management-cloud/docs/use-advanced-search-with-jira-query-language-jql/)
- [Atlassian Document Format](https://developer.atlassian.com/cloud/jira/platform/apis/document/structure/)

## Agent Instructions

When modifying this plugin:

1. **Always use `var`** for top-level declarations
2. **Test credential flow** - first run, re-authentication
3. **Verify field selection** - include all needed fields in API request
4. **Handle ADF format** - check `typeof` and `doc.type` before parsing
5. **Add debug logging** - help users troubleshoot issues
6. **Update version number** - in metadata when making changes
7. **Preserve backward compatibility** - default new features to `false`
8. **Test with actual Jira** - curl commands with real API tokens
9. **Document configuration** - add comments for new options
10. **Update README.md** - reflect new features/changes

## Common Enhancement Patterns

### Adding a New Configuration Option

1. Add to configuration section with comment
2. Add default value (usually `false` for new features)
3. Use in conditional logic throughout sync
4. Update README.md Configuration section
5. Update version number
6. Commit with descriptive message

### Adding a New Jira Field to Sync

1. Add field name to `fields` variable (conditionally if optional)
2. Parse field from `issue.fields.fieldname` 
3. Set on OmniFocus task (new and existing)
4. Add error handling for parsing
5. Add debug logging
6. Test with issues that have/don't have the field

### Improving Error Handling

1. Identify error scenario (HTTP code, parse error, etc.)
2. Add specific catch or check
3. Create user-friendly error message
4. Log detailed error to console
5. Provide actionable guidance (e.g., "Check credentials")
6. Test error scenario
