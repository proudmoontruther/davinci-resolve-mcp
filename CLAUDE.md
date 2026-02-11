# DaVinci Resolve MCP Server - AI Assistant Guide

**Version:** 1.3.8
**Last Updated:** 2026-01-28
**Purpose:** Comprehensive guide for AI assistants working with this codebase

---

## Table of Contents

1. [Project Overview](#project-overview)
2. [Architecture](#architecture)
3. [Directory Structure](#directory-structure)
4. [Core Components](#core-components)
5. [Development Workflows](#development-workflows)
6. [Coding Conventions](#coding-conventions)
7. [Environment Setup](#environment-setup)
8. [Testing Strategy](#testing-strategy)
9. [Common Tasks](#common-tasks)
10. [Important Constraints](#important-constraints)
11. [Troubleshooting](#troubleshooting)

---

## Project Overview

### What is This?

The DaVinci Resolve MCP Server is a **Model Context Protocol (MCP)** server that bridges AI coding assistants (Cursor, Claude Desktop) with Blackmagic Design's DaVinci Resolve video editing software. It enables AI assistants to query and control DaVinci Resolve through natural language commands.

### Key Statistics

- **Current Version:** 1.3.8
- **Lines of Code:** ~5,138 Python LOC
- **Implemented Features:** 202 (100% implementation coverage)
- **Verified Features:** 17 (8% verification coverage on macOS)
- **Platform Support:** macOS ✅, Windows ✅, Linux ❌

### Technology Stack

- **Language:** Python 3.6+
- **Protocol:** Model Context Protocol (MCP)
- **Framework:** FastMCP (from MCP Python SDK)
- **External APIs:** DaVinci Resolve Scripting API
- **Package Management:** pip + requirements.txt

---

## Architecture

### High-Level Design

```
┌─────────────────────────────────────────────┐
│  AI Assistant (Cursor/Claude Desktop)       │
└─────────────────┬───────────────────────────┘
                  │ MCP Protocol (stdin/stdout)
┌─────────────────▼───────────────────────────┐
│  MCP Server (FastMCP)                       │
│  - Tool registration                        │
│  - Request routing                          │
│  - Response formatting                      │
└─────────────────┬───────────────────────────┘
                  │ Python API Calls
┌─────────────────▼───────────────────────────┐
│  DaVinci Resolve Python API                 │
│  - Project Management                       │
│  - Timeline Operations                      │
│  - Media Pool Management                    │
│  - Color Grading                            │
│  - Rendering                                │
└─────────────────────────────────────────────┘
```

### Component Layers

1. **Entry Point Layer** (`src/main.py`)
   - Environment setup and validation
   - Logging configuration
   - Server initialization

2. **Server Layer** (`resolve_mcp_server.py`)
   - FastMCP server instance
   - Tool registration and routing
   - Error handling and logging

3. **API Layer** (`src/api/`)
   - Project operations
   - Timeline operations
   - Media operations
   - Color operations
   - Delivery operations

4. **Utility Layer** (`src/utils/`)
   - Platform detection and configuration
   - Resolve connection management
   - Object inspection
   - Layout presets
   - Cloud operations
   - App control

---

## Directory Structure

```
davinci-resolve-mcp/
├── src/                          # Main source code
│   ├── main.py                   # Primary entry point (USE THIS)
│   ├── resolve_mcp_server.py     # Core MCP server (symlinked to root)
│   ├── api/                      # API operation modules
│   │   ├── __init__.py
│   │   ├── project_operations.py
│   │   ├── timeline_operations.py
│   │   ├── media_operations.py
│   │   ├── color_operations.py
│   │   └── delivery_operations.py
│   ├── utils/                    # Utility modules
│   │   ├── __init__.py
│   │   ├── platform.py           # Platform detection
│   │   ├── resolve_connection.py # Resolve API connection
│   │   ├── object_inspection.py  # API introspection
│   │   ├── layout_presets.py     # UI layout management
│   │   ├── app_control.py        # App lifecycle control
│   │   ├── cloud_operations.py   # Cloud project management
│   │   └── project_properties.py # Project settings
│   └── bin/                      # Binary/executable helpers
│
├── scripts/                      # Automation and setup scripts
│   ├── setup/                    # Installation scripts
│   ├── mcp_resolve-cursor_start  # Cursor launcher
│   ├── mcp_resolve-claude_start  # Claude Desktop launcher
│   ├── mcp_resolve_launcher.sh   # Universal launcher
│   ├── check-resolve-ready.sh    # Pre-launch validation
│   ├── run-now.sh                # Quick start script
│   └── verify-installation.sh    # Installation verification
│
├── config/                       # Configuration templates
│   ├── macos/                    # macOS-specific configs
│   ├── windows/                  # Windows-specific configs
│   ├── sample_config.json        # Example configuration
│   └── README.md                 # Config documentation
│
├── docs/                         # Documentation
│   ├── FEATURES.md               # Feature implementation status
│   ├── CHANGELOG.md              # Version history
│   ├── VERSION.md                # Current version info
│   ├── TOOLS_README.md           # MCP tools documentation
│   └── PROJECT_MCP_SETUP.md      # Project setup guide
│
├── examples/                     # Example scripts
│   ├── getting_started.py
│   ├── markers/                  # Marker examples
│   ├── timeline/                 # Timeline examples
│   └── media/                    # Media pool examples
│
├── tests/                        # Test scripts
│   └── (test files)
│
├── logs/                         # Runtime logs
│
├── README.md                     # Primary documentation
├── INSTALL.md                    # Installation guide
├── CHANGES.md                    # Recent changes
├── CHANGELOG.md                  # Version history
├── requirements.txt              # Python dependencies
├── .cursorrules                  # Cursor IDE configuration
└── .gitignore                    # Git ignore patterns
```

### Key Files

| File | Purpose | Usage |
|------|---------|-------|
| `src/main.py` | **Primary entry point** | Use this for all MCP client configurations |
| `resolve_mcp_server.py` | Main server (symlink) | For backward compatibility only |
| `requirements.txt` | Python dependencies | `pip install -r requirements.txt` |
| `.cursorrules` | Cursor IDE rules | Defines shortcuts and navigation commands |
| `docs/FEATURES.md` | Feature status tracker | 202 features with implementation/verification status |

---

## Core Components

### 1. Entry Point (`src/main.py`)

**Purpose:** Standardized entry point for the MCP server

**Key Functions:**
- `check_setup()` - Validates environment variables
- `run_server(debug=False)` - Starts the FastMCP server
- `main()` - CLI entry point with argument parsing

**Environment Variables Set:**
- `RESOLVE_SCRIPT_API` - Path to DaVinci Resolve API scripts
- `RESOLVE_SCRIPT_LIB` - Path to Resolve library (fusionscript.so/dll)
- `PYTHONPATH` - Includes Resolve API modules

### 2. MCP Server (`src/resolve_mcp_server.py`)

**Purpose:** Core FastMCP server with all tool registrations

**Structure:**
```python
from mcp.server.fastmcp import FastMCP

mcp = FastMCP("DaVinci Resolve MCP Server")

@mcp.tool()
def tool_name(param: type) -> type:
    """Tool description for AI assistant"""
    # Implementation
    return result
```

**Key Sections:**
- Server initialization and configuration
- Resolve API connection setup
- Tool registrations (202 tools)
- Error handling and logging

### 3. API Operations (`src/api/`)

**Module Organization:**

| Module | Purpose | Key Functions |
|--------|---------|---------------|
| `project_operations.py` | Project management | list_projects, create_project, open_project, save_project |
| `timeline_operations.py` | Timeline editing | create_timeline, add_marker, get_timeline_items |
| `media_operations.py` | Media pool | import_media, create_bin, add_clip_to_timeline |
| `color_operations.py` | Color grading | apply_lut, set_color_grade, manage_nodes |
| `delivery_operations.py` | Rendering | add_render_job, start_rendering, get_render_status |

### 4. Utilities (`src/utils/`)

**Module Breakdown:**

| Module | Purpose |
|--------|---------|
| `platform.py` | Cross-platform path detection (macOS/Windows) |
| `resolve_connection.py` | Resolve API connection and validation |
| `object_inspection.py` | API introspection and documentation generation |
| `layout_presets.py` | UI layout save/load/export/import |
| `app_control.py` | Application lifecycle (quit, restart, settings) |
| `cloud_operations.py` | Cloud project management |
| `project_properties.py` | Advanced project settings and metadata |

---

## Development Workflows

### Standard Development Flow

1. **Environment Setup**
   ```bash
   # Create virtual environment
   python -m venv venv
   source venv/bin/activate  # macOS/Linux
   # or
   venv\Scripts\activate     # Windows

   # Install dependencies
   pip install -r requirements.txt
   ```

2. **Verify DaVinci Resolve is Running**
   ```bash
   ./scripts/check-resolve-ready.sh  # macOS
   # or
   scripts\check-resolve-ready.bat   # Windows
   ```

3. **Start Development Server**
   ```bash
   python src/main.py --debug
   ```

4. **Testing Changes**
   - Use example scripts in `examples/`
   - Test with Cursor or Claude Desktop
   - Run verification: `./scripts/verify-installation.sh`

5. **Commit Changes**
   ```bash
   git add .
   git commit -m "Brief description"
   git push origin branch-name
   ```

### Adding New Features

**Step-by-Step Process:**

1. **Identify API Method**
   - Check `docs/FEATURES.md` for status
   - Review DaVinci Resolve API documentation
   - Use `inspect_object` tool for introspection

2. **Implement in Appropriate Module**
   ```python
   # In src/api/appropriate_operations.py

   def new_operation(resolve, param1: str, param2: int) -> dict:
       """
       Description of what this operation does.

       Args:
           resolve: Resolve API instance
           param1: Description of param1
           param2: Description of param2

       Returns:
           Dictionary with operation result
       """
       try:
           # Get necessary Resolve objects
           project_manager = resolve.GetProjectManager()
           project = project_manager.GetCurrentProject()

           # Perform operation
           result = project.SomeMethod(param1, param2)

           return {
               "success": True,
               "result": result,
               "message": "Operation completed successfully"
           }
       except Exception as e:
           logger.error(f"Error in new_operation: {str(e)}")
           return {
               "success": False,
               "error": str(e)
           }
   ```

3. **Register MCP Tool**
   ```python
   # In src/resolve_mcp_server.py

   @mcp.tool()
   def new_operation_tool(param1: str, param2: int) -> dict:
       """AI-friendly description of operation"""
       resolve = get_resolve()
       if not resolve:
           return {"success": False, "error": "Failed to connect to Resolve"}

       return new_operation(resolve, param1, param2)
   ```

4. **Update Documentation**
   - Add to `docs/FEATURES.md` with status
   - Update `docs/TOOLS_README.md` if needed
   - Add example to `examples/` if applicable

5. **Test Thoroughly**
   - Test on target platform (macOS/Windows)
   - Test with both Cursor and Claude Desktop
   - Verify error handling

### Release Process

1. **Update Version**
   - Modify version in `docs/VERSION.md`
   - Update version in `resolve_mcp_server.py`
   - Update `CHANGELOG.md`

2. **Create Release**
   ```bash
   # macOS/Linux
   ./scripts/create-release-zip.sh

   # Windows
   scripts\create-release-zip.bat
   ```

3. **Tag Release**
   ```bash
   git tag -a v1.3.9 -m "Release version 1.3.9"
   git push origin v1.3.9
   ```

---

## Coding Conventions

### Python Style Guide

**Follow PEP 8 with these specifics:**

1. **Imports**
   ```python
   # Standard library
   import os
   import sys
   import logging
   from typing import List, Dict, Any, Optional

   # Third-party
   from mcp.server.fastmcp import FastMCP

   # Local
   from src.utils.platform import get_platform
   from src.api.project_operations import list_projects
   ```

2. **Function Definitions**
   ```python
   def function_name(
       param1: str,
       param2: Optional[int] = None,
       param3: bool = False
   ) -> Dict[str, Any]:
       """
       Brief one-line description.

       Longer description if needed, explaining what the function does,
       any important side effects, and usage notes.

       Args:
           param1: Description of param1
           param2: Description of param2 (default: None)
           param3: Description of param3 (default: False)

       Returns:
           Dictionary containing:
           - success: Boolean indicating operation success
           - result: Operation result data
           - error: Error message if success is False

       Raises:
           ValueError: If param1 is invalid
           ConnectionError: If Resolve connection fails
       """
       # Implementation
   ```

3. **Error Handling**
   ```python
   try:
       # Risky operation
       result = resolve.SomeMethod()
       return {"success": True, "result": result}
   except Exception as e:
       logger.error(f"Error in function_name: {str(e)}")
       return {"success": False, "error": str(e)}
   ```

4. **Logging**
   ```python
   logger = logging.getLogger("davinci-resolve-mcp.module_name")

   logger.debug("Detailed debug information")
   logger.info("General information")
   logger.warning("Warning message")
   logger.error("Error message")
   logger.critical("Critical error")
   ```

### Naming Conventions

| Type | Convention | Example |
|------|-----------|---------|
| Module | `snake_case` | `project_operations.py` |
| Class | `PascalCase` | `ResolveConnection` |
| Function | `snake_case` | `get_current_project()` |
| Variable | `snake_case` | `project_name` |
| Constant | `UPPER_SNAKE_CASE` | `RESOLVE_API_PATH` |
| Private | `_leading_underscore` | `_internal_function()` |

### Documentation Standards

**Module Docstrings:**
```python
"""
Module Name - Brief Description

This module provides functionality for XYZ operations in DaVinci Resolve.
It includes functions for A, B, and C.

Key Functions:
    - function_a: Does A
    - function_b: Does B
    - function_c: Does C

Dependencies:
    - DaVinci Resolve API
    - Some external library

Author: Samuel Gursky
Version: 1.3.8
"""
```

**MCP Tool Docstrings:**
```python
@mcp.tool()
def tool_name(param: str) -> dict:
    """
    Brief description for AI assistant to understand when to use this tool.

    This tool is used when the user wants to perform a specific operation
    in DaVinci Resolve. It requires parameter X and returns Y.

    Example usage: "Create a new project called 'My Project'"
    """
```

---

## Environment Setup

### Platform-Specific Paths

**macOS:**
```python
RESOLVE_SCRIPT_API = "/Library/Application Support/Blackmagic Design/DaVinci Resolve/Developer/Scripting"
RESOLVE_SCRIPT_LIB = "/Applications/DaVinci Resolve/DaVinci Resolve.app/Contents/Libraries/Fusion/fusionscript.so"
RESOLVE_MODULES_PATH = f"{RESOLVE_SCRIPT_API}/Modules/"
```

**Windows:**
```python
RESOLVE_SCRIPT_API = "C:\\ProgramData\\Blackmagic Design\\DaVinci Resolve\\Support\\Developer\\Scripting"
RESOLVE_SCRIPT_LIB = "C:\\Program Files\\Blackmagic Design\\DaVinci Resolve\\fusionscript.dll"
RESOLVE_MODULES_PATH = f"{RESOLVE_SCRIPT_API}\\Modules"
```

### Automatic Path Detection

The `src/utils/platform.py` module automatically detects the platform and sets appropriate paths:

```python
from src.utils.platform import get_resolve_paths

paths = get_resolve_paths()
# Returns: {"api_path": "...", "lib_path": "...", "modules_path": "..."}
```

### Virtual Environment

**Always use a virtual environment:**

```bash
# Create
python -m venv venv

# Activate
source venv/bin/activate  # macOS/Linux
venv\Scripts\activate     # Windows

# Install dependencies
pip install -r requirements.txt

# Verify
pip list
```

---

## Testing Strategy

### Current Testing Status

- **Total Features:** 202
- **Implemented:** 202 (100%)
- **Verified on macOS:** 17 (8%)
- **Needs Testing:** 166 (82%)
- **Known Issues:** 19 (10%)

### Testing Approach

1. **Unit Testing** (Planned)
   - Test individual API operations
   - Mock Resolve API responses
   - Test error handling

2. **Integration Testing** (Current)
   - Test with actual DaVinci Resolve
   - Use example scripts in `examples/`
   - Manual verification through Cursor/Claude

3. **Platform Testing**
   - Primary: macOS (currently most tested)
   - Secondary: Windows (needs more coverage)
   - Not supported: Linux

### Running Tests

```bash
# Verify installation
./scripts/verify-installation.sh

# Test basic operations
python examples/getting_started.py

# Test specific features
python examples/timeline/create_timeline.py
python examples/markers/add_markers.py
```

### Known Issues (as of 1.3.8)

See `docs/FEATURES.md` for detailed status. Key issues:

1. **Color Operations** - "Cannot access grade object" errors
2. **Render Jobs** - AddRenderJob fails in testing
3. **Timeline Creation** - Fails with existing names without clear errors
4. **Proxy Operations** - Various "clip not found" errors
5. **Project Settings** - Parameter type mismatches

---

## Common Tasks

### For AI Assistants Working on This Codebase

#### Task 1: Adding a New DaVinci Resolve Operation

1. Check if it's listed in `docs/FEATURES.md`
2. Determine which API module it belongs to (`src/api/`)
3. Implement the operation function
4. Register it as an MCP tool in `resolve_mcp_server.py`
5. Update `docs/FEATURES.md` with implementation status
6. Add example usage to `examples/` directory

#### Task 2: Fixing a Bug

1. Reproduce the issue
2. Check logs in `logs/` directory
3. Add debug logging if needed
4. Identify root cause (often API parameter mismatches)
5. Implement fix with proper error handling
6. Test on target platform(s)
7. Update status in `docs/FEATURES.md`

#### Task 3: Improving Documentation

1. Read existing docs to understand context
2. Identify gaps or outdated information
3. Update relevant files:
   - `README.md` - User-facing info
   - `docs/FEATURES.md` - Feature status
   - `docs/TOOLS_README.md` - Tool documentation
   - `CLAUDE.md` - AI assistant guide (this file)
4. Ensure consistency across all docs

#### Task 4: Cross-Platform Compatibility

1. Check platform detection in `src/utils/platform.py`
2. Test path resolution on target platform
3. Update platform-specific configs in `config/`
4. Test launcher scripts (`scripts/mcp_resolve-*_start`)
5. Update installation docs if needed

#### Task 5: Adding Configuration Options

1. Determine if config is global or project-specific
2. Update templates in `config/`
3. Update `config/README.md` with new options
4. Test with both Cursor and Claude Desktop
5. Update main `README.md` with usage examples

### Quick Commands (from .cursorrules)

| Command | Action |
|---------|--------|
| `/project` or `/structure` | View project structure |
| `/show server` | Display main server file |
| `/run` | Run server in dev mode |
| `/setup` | Setup server environment |
| `/check resolve` | Check Resolve environment variables |
| `/is resolve running` | Check if DaVinci Resolve is running |

---

## Important Constraints

### Technical Constraints

1. **DaVinci Resolve Must Be Running**
   - Server cannot start if Resolve is not running
   - No offline mode available
   - Resolve API only works with running instance

2. **Python Version**
   - Requires Python 3.6+
   - Tested primarily with Python 3.8-3.11
   - May not work with Python 3.12+ (untested)

3. **DaVinci Resolve Version**
   - Requires DaVinci Resolve 18.5+
   - Some features require Studio version
   - Free version has limited API access

4. **Platform Limitations**
   - Linux: Not supported (Resolve API unavailable)
   - Windows: Paths use forward slashes in JSON configs
   - macOS: Requires execute permissions on scripts

5. **MCP Protocol**
   - Communication via stdin/stdout only
   - No HTTP/REST interface
   - Designed for local execution only

### Design Constraints

1. **Entry Point**
   - MUST use `src/main.py` (not `resolve_mcp_server.py` directly)
   - Configurations should point to `main.py`

2. **Virtual Environment**
   - MUST use virtual environment
   - Absolute paths required in MCP configs

3. **Environment Variables**
   - Set automatically by `main.py`
   - Manual setting only needed for direct API testing

4. **Error Handling**
   - All operations should return `{"success": bool, ...}` format
   - Never raise unhandled exceptions to MCP client
   - Log all errors for debugging

### API Constraints

1. **Resolve API Quirks**
   - Many methods return `None` on failure (no exceptions)
   - Some operations require specific UI state
   - Color operations need existing grade objects
   - Timeline operations need valid frame ranges

2. **Threading**
   - Resolve API is not thread-safe
   - All operations must be synchronous
   - No parallel API calls

3. **Object Lifecycle**
   - Resolve objects can become invalid
   - Always get fresh references from API
   - Don't cache Resolve object references

---

## Troubleshooting

### Common Issues and Solutions

#### Issue: "Failed to connect to Resolve"

**Causes:**
- DaVinci Resolve not running
- Environment variables not set
- Wrong Resolve version

**Solutions:**
```bash
# Check if Resolve is running
ps aux | grep "DaVinci Resolve"  # macOS/Linux
tasklist | findstr "Resolve"     # Windows

# Verify environment
python -c "import os; print(os.environ.get('RESOLVE_SCRIPT_API'))"

# Run pre-flight check
./scripts/check-resolve-ready.sh
```

#### Issue: "Module 'DaVinciResolveScript' not found"

**Causes:**
- PYTHONPATH not set correctly
- Missing Resolve API modules

**Solutions:**
```bash
# Check PYTHONPATH
echo $PYTHONPATH

# Verify modules exist
ls "$RESOLVE_SCRIPT_API/Modules/"

# Use main.py entry point (sets paths automatically)
python src/main.py
```

#### Issue: "Operation returns None or fails silently"

**Causes:**
- Invalid parameters
- Wrong Resolve UI state
- Missing prerequisites (e.g., no current project)

**Solutions:**
1. Check parameter types match API expectations
2. Verify prerequisites (project open, timeline selected, etc.)
3. Add debug logging to trace execution
4. Use `inspect_object` tool to verify API methods

#### Issue: "Color operations fail with 'Cannot access grade object'"

**Causes:**
- No clip selected in Color page
- Clip has no grade information
- Wrong page active

**Solutions:**
```python
# Ensure Color page is active
resolve.OpenPage("Color")

# Get current timeline and items
timeline = project.GetCurrentTimeline()
items = timeline.GetItemListInTrack("video", 1)

# Work with clips that have grades
```

#### Issue: "Render job creation fails"

**Known Issue:** Currently being investigated (see `docs/FEATURES.md`)

**Workaround:**
- Use Resolve UI to set up render job manually
- Use `StartRendering()` to start existing jobs
- Monitor status with `IsRenderingInProgress()`

### Debugging Tips

1. **Enable Debug Logging**
   ```bash
   python src/main.py --debug
   ```

2. **Check Logs**
   ```bash
   tail -f logs/cursor_resolve_server.log
   ```

3. **Use Object Inspection**
   ```python
   # From Cursor/Claude
   "Inspect the current project object and show me all available methods"
   ```

4. **Test Operations Directly**
   ```python
   # Create test script
   import sys
   sys.path.append('/path/to/Resolve/Modules')
   import DaVinciResolveScript as dvr

   resolve = dvr.scriptapp("Resolve")
   pm = resolve.GetProjectManager()
   project = pm.GetCurrentProject()
   # Test operation
   ```

### Getting Help

1. **Check Documentation**
   - `README.md` - General overview
   - `INSTALL.md` - Installation issues
   - `docs/FEATURES.md` - Feature status
   - `docs/TOOLS_README.md` - Tool usage

2. **GitHub Issues**
   - Search existing issues
   - Create new issue with logs and platform info

3. **Logs and Diagnostics**
   ```bash
   # Run diagnostics
   ./scripts/verify-installation.sh

   # Check system info
   python --version
   pip list
   echo $RESOLVE_SCRIPT_API
   ```

---

## Advanced Topics

### MCP Protocol Details

**Communication Flow:**
```
Cursor/Claude → stdin → MCP Server → Process → stdout → Cursor/Claude
```

**Message Format:**
```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "method": "tools/call",
  "params": {
    "name": "tool_name",
    "arguments": {"param": "value"}
  }
}
```

### Extending the Server

**Adding Custom Utilities:**

1. Create new module in `src/utils/`
2. Implement utility functions
3. Import in `resolve_mcp_server.py`
4. Register as MCP tools if needed

**Example:**
```python
# src/utils/custom_utility.py
def custom_operation(resolve, param: str) -> dict:
    """Custom operation implementation"""
    # Implementation
    return {"success": True, "result": data}

# In resolve_mcp_server.py
from src.utils.custom_utility import custom_operation

@mcp.tool()
def custom_tool(param: str) -> dict:
    """AI-friendly description"""
    resolve = get_resolve()
    return custom_operation(resolve, param)
```

### Performance Optimization

**Tips:**
1. Minimize Resolve API calls (they're expensive)
2. Batch operations when possible
3. Cache static data (available LUTs, presets, etc.)
4. Use `inspect_object` once, not repeatedly
5. Avoid getting full object lists when IDs suffice

### Security Considerations

1. **No Authentication** - MCP server trusts client completely
2. **Local Only** - Not designed for network access
3. **File System Access** - Can read/write files as user
4. **Resolve Control** - Full control over DaVinci Resolve
5. **No Sandboxing** - Runs with user permissions

**Best Practices:**
- Only use on trusted systems
- Don't expose to network
- Review operations before implementing
- Validate file paths in operations
- Don't store sensitive data in logs

---

## Appendix

### Useful References

- [DaVinci Resolve Scripting Documentation](https://www.blackmagicdesign.com/developer/product/davinci-resolve)
- [Model Context Protocol Specification](https://github.com/modelcontextprotocol)
- [FastMCP Documentation](https://github.com/modelcontextprotocol/python-sdk)

### Version History Highlights

- **1.3.8** - Cursor integration improvements, standardized entry point
- **1.3.7** - Installation experience improvements, path resolution fixes
- **1.3.6** - Complete object method implementation (202 features)
- **1.3.5** - Cursor integration enhancements
- **1.3.4** - Configuration template improvements

### Contributing Guidelines

1. **Code Quality**
   - Follow PEP 8 style guide
   - Add type hints
   - Write docstrings
   - Handle errors gracefully

2. **Testing**
   - Test on macOS and Windows if possible
   - Update `docs/FEATURES.md` with test results
   - Add examples for new features

3. **Documentation**
   - Update relevant docs
   - Add inline comments for complex logic
   - Update CHANGELOG.md
   - Keep this file (CLAUDE.md) current

4. **Git Practices**
   - Clear commit messages
   - Small, focused commits
   - Reference issues in commits
   - Test before pushing

### Contact

- **Author:** Samuel Gursky
- **Email:** samgursky@gmail.com
- **GitHub:** [github.com/samuelgursky](https://github.com/samuelgursky)
- **Repository:** [github.com/samuelgursky/davinci-resolve-mcp](https://github.com/samuelgursky/davinci-resolve-mcp)

---

**End of AI Assistant Guide**

*This document is maintained for AI assistants (like Claude and Cursor AI) working with this codebase. Keep it updated as the project evolves.*
