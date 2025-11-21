# jira-omnifocus-improved

An improved and secure OmniFocus plugin for syncing Jira issues. This is a fork of [Bastian-Kuhn/jira-omnifocus](https://github.com/Bastian-Kuhn/jira-omnifocus) with significant security, performance, and reliability enhancements.

## What's Improved?

This fork addresses critical security and quality issues found in the original plugin:

### Security Enhancements
- **Secure Credential Storage**: Credentials are now stored in macOS Keychain instead of plain text in the plugin file
- **No Hardcoded Passwords**: Credentials are prompted on first run and encrypted by macOS

### Error Handling & Reliability
- **Comprehensive Error Handling**: Validates HTTP responses, handles authentication failures, and provides clear error messages
- **User-Friendly Alerts**: Shows success/failure notifications with actionable information
- **Detailed Logging**: Distinguishes ERROR, WARNING, INFO, and DEBUG messages

### Performance Improvements
- **Batch API Requests**: Comment fetching uses Promise.all() for parallel requests instead of sequential
- **Reduced Sync Time**: Significantly faster for tickets with many comments

### Code Quality
- **Input Validation**: Validates all configuration fields including URL format and required fields
- **Improved Regex**: Proper Jira ticket format matching (PROJECT-123) instead of overly permissive patterns
- **Named Constants**: Magic strings extracted to meaningful constants
- **Modern JavaScript**: Consistent use of const/let, template literals

## Installation

### Method 1: Double-Click Installation (Easiest)

1. Download `jira.omnifocusjs` from this repository
2. **macOS**: Double-click the file or select it and choose File → Open
   - An installation dialog will appear
   - Choose where to install (On My Mac or iCloud Drive)
   - Click "Install"
3. **iOS/iPadOS**: Tap the file in Files app
   - Installation dialog will appear
   - Choose storage location
   - Tap "Install"

### Method 2: Manual Installation

1. Download `jira.omnifocusjs` from this repository
2. Open OmniFocus
3. Go to **Automation → Configure...** (or **Automation → Plug-Ins...**)
4. Click the **Plug-Ins** tab
5. Click **Reveal in Finder** (or **Add Linked Folder...** to use a custom location)
6. Copy `jira.omnifocusjs` into the revealed folder
7. The plugin will appear in the Automation menu

### Method 3: Linked Folder (Advanced)

For easier management, you can use a linked folder:

1. Create a folder for your OmniFocus plugins (e.g., `~/Documents/OmniFocus Plug-Ins`)
2. Place `jira.omnifocusjs` in this folder
3. In OmniFocus: **Automation → Configure... → Plug-Ins tab**
4. Click **Add Linked Folder...**
5. Select your created folder
6. All plugins in this folder will now be available

### First Run Setup

1. Open OmniFocus
2. Run the plugin: **Automation → 🔹 Jira: Sync**
3. You'll be prompted to enter credentials:
   - **Username**: Your Jira email address
   - **API Token**: Generate at https://id.atlassian.com/manage-profile/security/api-tokens
4. Credentials are stored securely in macOS Keychain
5. Configure the plugin by editing the `configs` array (see Configuration below)

## Configuration

Edit the `configs` array in `jira.omnifocusjs`:

```javascript
const configs = [{
    jiraUrl: "https://yourcompany.atlassian.net",  // Your Jira instance URL
    credentialKey: "jira-default",                  // Unique key for Keychain storage
    omnifocusTagToUse: "Jira",                      // Tag for synced tasks (supports "Parent : Child")
    jiraQuery: "assignee=currentuser() and resolution is empty",  // JQL query
    processComments: false,                          // Enable comment syncing as subtasks
    omnifocusCommentTagToUse: "Jira-Comment",       // Tag for comment subtasks
}]
```

For multiple Jira instances, add additional config objects to the array.

## How It Works

The plugin identifies tasks by the Jira ticket number (e.g., `PROJECT-123`), which must be at the beginning of the task name. You can edit everything after the ticket number. The note contains the Jira link and ticket description. If you remove the tag from a task, the plugin will no longer sync it.

## Features

- **Automatic Sync**: Queries all open tickets assigned to you from Jira
- **Task Management**: Creates new tasks or reopens completed tasks when they appear in Jira
- **Auto-Complete**: Closes tasks that no longer appear in your Jira query
- **Multi-Instance Support**: Sync from multiple Jira instances simultaneously
- **Comment Sync**: Optional syncing of Jira comments as subtasks
- **Custom JQL Queries**: Filter tickets using any JQL query
- **Hierarchical Tags**: Supports nested tags (e.g., "Work : Projects : Jira")
- **Smart Ticket Detection**: Validates proper Jira ticket format (PROJECT-123)
- **Secure**: Credentials stored in macOS Keychain, never in plain text

## Usage

1. **Run the plugin**: **Automation → 🔹 Jira: Sync**
2. **First run**: Enter credentials when prompted
3. **Subsequent runs**: The plugin will automatically:
   - ✓ Fetch open tickets from Jira matching your query
   - ✓ Create OmniFocus tasks for new tickets
   - ✓ Reopen completed tasks for tickets that are still open
   - ✓ Complete tasks for tickets no longer in your query
   - ✓ Optionally sync comments as subtasks

### Success/Failure Notifications

The plugin shows user-friendly alerts:
- **Success**: Shows count of checked tickets, created tasks, and comment subtasks
- **Failure**: Shows specific error with actionable guidance (e.g., "Check credentials")

## Managing Credentials

### Setting Credentials
- **Username**: Your Jira email address
- **API Token**: Generate at https://id.atlassian.com/manage-profile/security/api-tokens
- Stored securely in macOS Keychain (never in plain text)

### Resetting Credentials

**Method 1 - Using Keychain Access (macOS):**
1. Open **Keychain Access** app
2. Search for your credential key (e.g., "jira-default")
3. Delete the credential entry
4. Re-run the plugin to enter new credentials

**Method 2 - Using Plugin (macOS):**
1. Hold **Control** key when selecting plugin from Automation menu
2. Click "Reset" in confirmation dialog
3. Re-run the plugin to enter new credentials

## Troubleshooting

### Plugin Not Appearing in Automation Menu

**Check plugin location:**
1. Go to **Automation → Configure... → Plug-Ins tab**
2. Verify the plugin appears in the list
3. If not, check you copied the file to the correct folder
4. Click **Reveal in Finder** to see the plugins folder

**macOS: Use double-click installation:**
- Download the `.omnifocusjs` file
- Double-click to trigger automatic installation

**Verify file extension:**
- File must end with `.omnifocusjs`
- Not `.js` or `.txt`

### Authentication Errors (401, 403)

**Error 401 - Authentication Failed:**
- Verify your Jira email address is correct
- Regenerate your API token at https://id.atlassian.com/manage-profile/security/api-tokens
- Old API tokens may have expired
- Reset credentials and enter new ones

**Error 403 - Access Forbidden:**
- Your API token may lack necessary permissions
- Ensure you have permission to view the Jira project
- Contact your Jira administrator

### Configuration Errors

**Plugin shows validation error:**
- Check `jiraUrl` starts with `http://` or `https://`
- Verify `jiraUrl` doesn't have trailing slash
- Ensure all required fields are filled in
- Check `credentialKey` is unique for each Jira instance

**Example correct config:**
```javascript
jiraUrl: "https://yourcompany.atlassian.net",  // ✓ Correct
jiraUrl: "yourcompany.atlassian.net",          // ✗ Missing https://
jiraUrl: "https://yourcompany.atlassian.net/", // ✗ Trailing slash
```

### Sync Issues

**No tasks created:**
- Verify your JQL query returns results in Jira web interface
- Check you're logged into the correct Jira instance
- Ensure tickets are assigned to your account
- Check Console.app for detailed error messages

**Tasks not completing:**
- Tasks with incomplete subtasks won't auto-complete (by design)
- Manually complete subtasks first

**Comment sync not working:**
- Ensure `processComments: true` in config
- Comment sync only works for tickets you've synced before
- Check `omnifocusCommentTagToUse` is configured

### Viewing Detailed Logs

**macOS:**
1. Open **Console.app**
2. Filter for "OmniFocus"
3. Run the plugin
4. Look for ERROR, WARNING, or INFO messages

**Error messages include:**
- Specific HTTP status codes
- JQL query issues
- Credential problems
- Network connectivity issues

## Notes & Limitations

- **Task Identification**: Tasks identified by ticket ID at start of name (e.g., "PROJECT-123 Task title")
- **Auto-Complete Protection**: Tasks with incomplete children won't be auto-completed
- **Comment Sync Scope**: Only works for tickets you've previously synced
- **Archived Tasks**: Won't receive comment updates
- **Network Dependency**: Requires internet connection to Jira instance
- **Rate Limiting**: Batch requests help, but very large syncs may hit Jira API limits 

## Changelog

### Version 2.0 (Improved Fork - 2025)
- **Security**: Moved credentials to macOS Keychain storage
- **Error Handling**: Comprehensive validation and user-friendly error messages  
- **Performance**: Batched API requests using Promise.all()
- **Validation**: Input validation for all configuration fields
- **Code Quality**: Improved regex, named constants, modern JavaScript
- **UX**: Success/failure Alert notifications

### Version 1.3 (Original)
- Tickets with children are no longer closed automatically

### Version 1.2 (Original)
- Added support to query multiple JIRA Instances

### Version 1.1 (Original)
- Added Function to add Jira Comments as subtask to the task

## Plugin Development

Want to modify the plugin? Here's what you need to know:

### Editing the Plugin

**macOS:**
1. **Automation → Configure... → Plug-Ins tab**
2. Select the plugin
3. Click the **Edit** button (pencil icon) or **Reveal in Finder**
4. Edit with your preferred text editor
5. Save changes - they take effect immediately

**Testing Changes:**
- Run the plugin after each edit
- Check **Console.app** for errors
- Use `console.log()` for debugging

### Key Files to Understand

- **Lines 1-10**: Plugin metadata (version, identifier, description)
- **Lines 12-34**: Constants (API paths, status codes)
- **Lines 42-60**: Configuration array (edit jiraUrl, queries, tags)
- **Lines 164-177**: Credentials handling (Keychain integration)
- **Lines 273-305**: Error handling and validation
- **Lines 363-405**: Batch comment fetching

### Best Practices

- Always test with a small dataset first
- Use descriptive `credentialKey` names for multiple instances
- Keep backups before making major changes
- Consult [Omni Automation docs](https://omni-automation.com) for API reference

### Useful Resources

- [Omni Automation Documentation](https://omni-automation.com)
- [OmniFocus Automation](https://omni-automation.com/omnifocus/)
- [Jira REST API](https://developer.atlassian.com/cloud/jira/platform/rest/v2/)
- [JQL Query Guide](https://support.atlassian.com/jira-service-management-cloud/docs/use-advanced-search-with-jira-query-language-jql/)

## Credits

- **Original Author**: [Bastian Kuhn](https://github.com/Bastian-Kuhn/jira-omnifocus)
- **Contributors**: @snwfdhmp (Martin Joly) - JavaScript improvements
- **Improved Fork**: [harpchad](https://github.com/harpchad/jira-omnifocus-improved) - Security and quality enhancements

## License

See LICENSE file (inherited from original project)

## Support

- **Issues**: [GitHub Issues](https://github.com/harpchad/jira-omnifocus-improved/issues)
- **Discussions**: Use GitHub Discussions for questions
- **Original Plugin**: [Bastian-Kuhn/jira-omnifocus](https://github.com/Bastian-Kuhn/jira-omnifocus)
