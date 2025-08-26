# VSTS Process Migrator for Node.js

This is a TypeScript CLI application that automates Azure DevOps process export/import across Azure DevOps organizations. It includes both a Node.js CLI tool and a browser extension component for Azure DevOps.

Always reference these instructions first and fallback to search or bash commands only when you encounter unexpected information that does not match the info here.

## Working Effectively

### Prerequisites and Setup
- Node.js version 8.11.2 or higher is required (current: Node.js 16.x+ recommended)
- Install dependencies: `npm install` -- takes ~1 second. NEVER CANCEL. Set timeout to 120+ seconds.

### Building the Application  
- Full build: `npm run build` -- takes ~3 seconds. NEVER CANCEL. Set timeout to 60+ seconds.
  - Compiles both Node.js CLI and browser extension components
  - Creates `build/nodejs/` and `build/browser/` directories
- Node.js only: `npm run bn` -- takes ~1.5 seconds. NEVER CANCEL. Set timeout to 30+ seconds.
- Browser only: `npm run bb` -- takes ~2 seconds. NEVER CANCEL. Set timeout to 30+ seconds.
- Alternative: Use individual TypeScript compilation:
  - `npx tsc -p src/nodejs/tsconfig.json` -- takes ~1.5 seconds. NEVER CANCEL.
  - `npx tsc -p src/browser/tsconfig.json` -- takes ~2 seconds. NEVER CANCEL.

### Running the Application
- ALWAYS run the building steps first before executing the application
- Execute CLI: `node build/nodejs/nodejs/Main.js [args]`
- CLI usage: `node build/nodejs/nodejs/Main.js [--mode=<migrate(default)/import/export>] [--config=<configuration-file-path>]`
- Configuration file auto-generation: Run without arguments to create default `configuration.json`

## Core Project Structure

### Repository Layout
```
├── src/
│   ├── nodejs/          # CLI application source
│   ├── common/          # Shared TypeScript interfaces and utilities
│   └── browser/         # Browser extension source (Azure DevOps extension)
├── build/               # Compiled JavaScript output (created by build)
├── .azure-pipelines/    # Azure DevOps CI/CD pipeline
├── .vscode/            # VS Code debugging configuration
├── configuration.json   # Auto-generated config file (contains PATs - protect this!)
└── output/             # Runtime log and export files (auto-created)
```

### Key Source Files
- `src/nodejs/Main.ts` - CLI entry point
- `src/nodejs/ConfigurationProcessor.ts` - Command line and config file processing
- `src/common/ProcessExporter.ts` - Azure DevOps process export logic
- `src/common/ProcessImporter.ts` - Azure DevOps process import logic
- `src/common/Interfaces.ts` - TypeScript interfaces and data structures

## Validation and Testing

### Configuration Validation
- Always test configuration file generation: Delete `configuration.json` and run `node build/nodejs/nodejs/Main.js`
- Verify config file structure matches expected format with proper JSONC comments
- Test CLI help: `node build/nodejs/nodejs/Main.js --help`

### Functionality Testing
- The application validates Azure DevOps URLs (must be https://dev.azure.com/orgname format)
- Without valid PAT tokens, the app will fail at connection stage with clear error messages
- Test modes: `--mode=export`, `--mode=import`, `--mode=migrate` (default)
- Log files are created at `output/processMigrator.log` (Windows-style path separators)

### Manual Validation Scenarios
Since this application requires live Azure DevOps accounts with proper PAT tokens:
- **Configuration Generation**: Run without args, verify `configuration.json` is created with proper structure
- **Validation Flow**: Test with dummy config to verify URL validation works
- **Help System**: Verify `--help` shows proper usage information
- **Mode Testing**: Test all three modes (export/import/migrate) reach proper validation stages

## Complete Developer Workflow Example

For a new developer working with this repository, follow this exact sequence:

### Initial Setup
1. `npm install` -- takes ~1 second. NEVER CANCEL. Set timeout to 120+ seconds.
2. `npm run build` -- takes ~3 seconds. NEVER CANCEL. Set timeout to 60+ seconds.
3. `node build/nodejs/nodejs/Main.js --help` -- verify CLI works

### Configuration Testing
1. Delete existing config: `rm configuration.json` (if exists)
2. Generate new config: `node build/nodejs/nodejs/Main.js`
3. Verify `configuration.json` was created with proper JSONC structure

### Functionality Validation  
1. Test all modes with dummy config:
   ```bash
   echo '{"sourceAccountUrl":"https://dev.azure.com/test","sourceAccountToken":"dummy","targetAccountUrl":"https://dev.azure.com/test","targetAccountToken":"dummy","sourceProcessName":"test","options":{}}' > test-config.json
   node build/nodejs/nodejs/Main.js --config=test-config.json --mode=export
   node build/nodejs/nodejs/Main.js --config=test-config.json --mode=import  
   node build/nodejs/nodejs/Main.js --config=test-config.json --mode=migrate
   ```
2. All should reach connection validation stage (not authentication errors)
3. Verify log files are created at `output/processMigrator.log`

## Common Tasks and Development Workflow
### Making Code Changes
1. Build and test the application BEFORE making changes: `npm install && npm run build` -- combined takes ~4 seconds total. NEVER CANCEL.
2. Run basic validation: `node build/nodejs/nodejs/Main.js --help`
3. Make your code changes
4. Rebuild immediately: `npm run build` -- takes ~3 seconds. NEVER CANCEL. Set timeout to 60+ seconds.
5. Test the specific functionality you changed

6. Always verify the CLI still shows help and validates configurations properly
- Use VS Code debug configuration: "Launch process migrate" (`.vscode/launch.json`)
- Source maps are enabled - you can debug TypeScript directly
- Set breakpoints in TypeScript source files
- Logs are written to `output/processMigrator.log` with timestamps and log levels

### Project Components
- **CLI Tool**: Main functionality for process migration between Azure DevOps orgs
- **Browser Extension**: Secondary component for Azure DevOps integration (placeholder implementation)
- **Configuration**: JSONC format supporting comments, auto-generated with template structure
- **Build System**: Dual TypeScript compilation for Node.js (CommonJS) and Browser (AMD)

## Important Notes

### Security and Configuration
- **CRITICAL**: Configuration files contain Personal Access Tokens (PATs) - never commit these to source control
- PATs require "Read, Write, & Manage" permissions for "Work Items" scope in Azure DevOps
- Always protect configuration files with sensitive data

### Known Issues
- Node.js v23.10+ has compatibility issues with util.isFunction (see issues #107, #108)
- Recommended to use Node.js 16.x-22.x for best compatibility

### Build and Test Approach
- NO existing test suite - manual validation required
- NO linting tools configured - focus on TypeScript compilation errors
- Build is very fast (under 5 seconds total) - always rebuild after changes
- Application has comprehensive validation and error reporting built-in

### Azure DevOps Integration
- Works only with "Inherited Process" templates (not XML processes)
- Supports process export, import, and full migration between Azure DevOps organizations
- Handles work item types, fields, rules, states, and layouts
- Requires network access to Azure DevOps REST APIs

## Validation Requirements

Before completing any work:
1. **Always build successfully**: `npm run build` must complete without errors
2. **Test CLI help**: `node build/nodejs/nodejs/Main.js --help` must show usage
3. **Test config generation**: Delete and regenerate `configuration.json`
4. **Verify TypeScript compilation**: Both nodejs and browser builds must succeed
5. **Check error handling**: Test with invalid configuration to verify validation works

The application is designed to be self-validating through its comprehensive configuration and connection validation systems.