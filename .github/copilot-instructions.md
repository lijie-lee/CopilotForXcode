# GitHub Copilot for Xcode - Coding Agent Instructions

## Project Overview

**GitHub Copilot for Xcode** is a Swift-based Xcode Source Editor Extension that brings GitHub Copilot's AI-powered code suggestions directly into Xcode. This project consists of multiple components working together to provide real-time code completion and chat functionality.

### Repository Statistics
- **Primary Language**: Swift (with supporting Node.js components)
- **Platforms**: macOS 12+, Xcode 8+
- **Architecture**: Multi-target Xcode project with Swift Package Manager dependencies
- **Size**: ~50MB codebase with comprehensive test suite
- **Dependencies**: Node.js/npm required for build process

## Build Instructions

### Prerequisites (CRITICAL)
**Node.js and npm MUST be available in system PATH** - the build will fail without these:
```bash
# Verify prerequisites
which npm && which node
# If missing, ensure they're symlinked to /usr/local/bin:
sudo ln -s $(which npm) /usr/local/bin
sudo ln -s $(which node) /usr/local/bin
```

### Build Commands
**Always use the workspace file, not the project file:**
```bash
# Clean build (recommended first step)
xcodebuild clean -workspace "Copilot for Xcode.xcworkspace" -scheme "Copilot for Xcode"

# Build for development
xcodebuild build \
  -workspace "Copilot for Xcode.xcworkspace" \
  -scheme "Copilot for Xcode" \
  -configuration Debug

# Archive for release
xcodebuild archive \
  -workspace "Copilot for Xcode.xcworkspace" \
  -scheme "Copilot for Xcode" \
  -configuration Release \
  -archivePath build/Archives/CopilotForXcode.xcarchive \
  CODE_SIGNING_ALLOWED="NO"
```

### Testing
**Always run tests using the test plan:**
```bash
# Run all unit tests (takes ~2-3 minutes)
xcodebuild test \
  -workspace "Copilot for Xcode.xcworkspace" \
  -scheme "Copilot for Xcode" \
  -testPlan TestPlan

# Run specific test target
xcodebuild test \
  -workspace "Copilot for Xcode.xcworkspace" \
  -scheme "Copilot for Xcode" \
  -only-testing:ServiceTests
```

### Code Formatting
**Always run SwiftFormat before committing:**
```bash
# Format all Swift code (uses .swiftformat config)
swiftformat .

# Check formatting without changes
swiftformat . --lint
```

## Project Architecture

### Core Components
- **Copilot for Xcode** (`/Copilot for Xcode/`) - Main host application providing settings UI
- **EditorExtension** (`/EditorExtension/`) - Xcode Source Editor Extension (sandboxed)
- **ExtensionService** (`/ExtensionService/`) - Background service implementing core features
- **CommunicationBridge** (`/CommunicationBridge/`) - XPC communication between components

### Swift Packages
- **Core** (`/Core/`) - Service logic, UI components, and host app implementations
- **Tool** (`/Tool/`) - Shared utilities, preferences, and provider implementations

### Key Configuration Files
- `Version.xcconfig` - Controls app version for all targets
- `Config.xcconfig` - Bundle identifiers, URLs, and app configuration
- `.swiftformat` - Code formatting rules (4 spaces, 100 char width)
- `TestPlan.xctestplan` - Comprehensive test configuration

## Build System Details

### npm Integration
The build system automatically downloads the Copilot Language Server via npm:
```bash
# This runs automatically during Xcode build
npm -C Server install
```

### Common Build Issues & Solutions

1. **"npm command not found" during build**
   - Ensure npm is in PATH: `sudo ln -s $(which npm) /usr/local/bin`
   - Restart Xcode after fixing PATH

2. **Package resolution failures**
   - Use `-disableAutomaticPackageResolution` flag
   - Delete `Package.resolved` files and rebuild

3. **SwiftUI Preview issues with Objective-C packages**
   - Switch to package product targets in scheme selector
   - Known limitation documented in DEVELOPMENT.md

4. **Extension not appearing in Xcode**
   - Check System Settings > Extensions > Xcode Source Editor
   - Restart Xcode after enabling extension

## Validation Pipeline

### GitHub Actions
- **CodeQL Analysis** - Runs on push/PR to main branch
- **Swift Build** - Uses Xcode 15.3 by default
- **Security Scanning** - Automated via CodeQL

### Local Validation Steps
```bash
# 1. Format code
swiftformat .

# 2. Build cleanly
xcodebuild clean build -workspace "Copilot for Xcode.xcworkspace" -scheme "Copilot for Xcode"

# 3. Run tests
xcodebuild test -workspace "Copilot for Xcode.xcworkspace" -scheme "Copilot for Xcode" -testPlan TestPlan

# 4. Check for common issues
grep -r "TODO\|FIXME\|HACK" --include="*.swift" .
```

## Directory Structure

```
/
├── .github/
│   ├── workflows/codeql.yml        # Security analysis
│   └── actions/set-xcode-version/  # Xcode version management
├── Core/                           # Main service package
│   ├── Package.swift
│   ├── Sources/
│   │   ├── Service/               # Extension service logic
│   │   ├── HostApp/               # Main app implementation
│   │   ├── SuggestionWidget/      # UI components
│   │   └── Client/                # XPC client
│   └── Tests/                     # Unit tests
├── Tool/                          # Utilities package
│   ├── Package.swift
│   ├── Sources/
│   │   ├── GitHubCopilotService/  # Copilot integration
│   │   ├── Preferences/           # Settings management
│   │   └── Workspace/             # File system utilities
│   └── Tests/
├── Server/                        # Node.js dependencies
│   ├── package.json               # Copilot Language Server
│   └── node_modules/              # Auto-generated
├── Script/                        # Utility scripts
│   └── uninstall-app.sh          # Development helper
├── Copilot for Xcode/            # Main app target
├── EditorExtension/              # Xcode extension
├── ExtensionService/             # Background service
├── CommunicationBridge/          # XPC service
└── TestPlan.xctestplan          # Test configuration
```

## Development Notes

### Dependencies to Note
- Swift Package Manager manages most dependencies
- Node.js packages are embedded during build
- Custom forks used for KeyboardShortcuts, CGEventOverride, and Highlightr
- Sparkle framework for auto-updates

### Extension Architecture
Due to Xcode extension sandboxing limitations:
1. EditorExtension forwards content to ExtensionService via XPCService
2. CommunicationBridge handles XPC communication
3. ExtensionService implements all features outside sandbox

### Permissions Required
- **Accessibility Permission** - For real-time editor content reading
- **Xcode Source Editor Extension** - For integration with Xcode
- Both must be enabled for full functionality

## Trust These Instructions

These instructions are comprehensive and tested. Only perform additional exploration if:
- Information here is incomplete for your specific task
- You encounter errors not covered in the troubleshooting section
- Build commands fail unexpectedly

For any issues, check TROUBLESHOOTING.md first, then DEVELOPMENT.md for additional context.

# Code Review
Always add hello wording "Hi, this is the review comment" for the review result message
