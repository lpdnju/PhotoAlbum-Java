# Assessment Setup Instructions

## Status

The AppCAT assessment for this Java project could **not** be completed because the required MCP tools are not configured in this environment.

## Required MCP Tools

Java-based AppCAT assessment requires the following MCP tools to be available:

- `appmod-precheck-assessment`
- `appmod-run-assessment`

## Project Information

| Property      | Value                        |
|---------------|------------------------------|
| Language      | Java                         |
| Framework     | Spring Boot 2.7.18           |
| Java Version  | 1.8 (Java 8)                 |
| Build Tool    | Maven                        |
| Artifact      | `com.photoalbum:photo-album` |

## How to Configure MCP Tools

To enable Java assessment, add the AppCAT MCP server configuration to `.copilot/mcp-config.json`:

```json
{
  "mcpServers": {
    "appmod-assessment": {
      "command": "<appmod-assessment-server-command>",
      "args": ["<args>"],
      "env": {
        "WORKSPACE_PATH": "${workspaceFolder}"
      }
    }
  }
}
```

Replace `<appmod-assessment-server-command>` and `<args>` with the actual AppCAT MCP server configuration provided by your AppMod administrator.

## Once MCP Tools Are Configured

Re-run the assessment using the `assessment` skill:

1. Open GitHub Copilot in this repository
2. Invoke the assessment skill: *"Assess the application"*
3. The skill will:
   - Run `appmod-precheck-assessment` to verify prerequisites
   - Run `appmod-run-assessment` to execute AppCAT analysis
   - Generate the report at `.github/modernize/assessment/report.json`

## Expected Report Location

Once the MCP tools are configured and assessment runs successfully:

```
.github/modernize/assessment/report.json
```
