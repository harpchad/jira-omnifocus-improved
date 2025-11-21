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

1. Download `jira.omnifocusjs` from this repository
2. Copy it to your OmniFocus Plugin Folder:
   - Open OmniFocus
   - Go to Automation → Configure Plugins → Open Plug-ins Folder
   - Copy `jira.omnifocusjs` to this folder
3. Configure the plugin by editing the `configs` array in the plugin file
4. On first run, you'll be prompted to enter your Jira credentials (stored securely in Keychain)

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

1. Run the plugin from OmniFocus: **Automation → Jira: Sync**
2. On first run, enter your Jira credentials when prompted
3. The plugin will:
   - Fetch open tickets from Jira matching your query
   - Create OmniFocus tasks for new tickets
   - Reopen completed tasks for tickets that are still open
   - Complete tasks for tickets no longer in your query
   - Optionally sync comments as subtasks

## Jira Credentials

- **Username**: Your Jira email address
- **API Token**: Generate at https://id.atlassian.com/manage-profile/security/api-tokens
- Credentials are stored securely in macOS Keychain
- To reset credentials, delete them from Keychain Access and re-run the plugin

## Notes & Limitations

- Tasks are identified by ticket ID at the start of the task name
- Tasks with incomplete children won't be auto-completed
- Comment sync only works for tickets you've previously synced
- Archived tasks in OmniFocus won't receive comment updates 

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

## Credits

- **Original Author**: [Bastian Kuhn](https://github.com/Bastian-Kuhn/jira-omnifocus)
- **Contributors**: @snwfdhmp (Martin Joly) - JavaScript improvements
- **Improved Fork**: [harpchad](https://github.com/harpchad/jira-omnifocus-improved) - Security and quality enhancements

## License

See LICENSE file (inherited from original project)
