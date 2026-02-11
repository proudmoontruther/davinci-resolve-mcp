# Linux Integration - Agent Starting Prompt

**Task:** Add full Linux compatibility to DaVinci Resolve MCP Server
**Reference:** See FUTURE_TASKS.md - Task #1
**Priority:** High
**Current Status:** 🔴 Not Started

---

## Mission

Your goal is to add complete Linux support to the DaVinci Resolve MCP Server. Currently, the project only works on macOS and Windows. We need to extend platform detection, path resolution, launcher scripts, and configuration to support Linux distributions.

---

## Context & Background

**What is this project?**
- MCP (Model Context Protocol) server that connects AI assistants to DaVinci Resolve
- Written in Python 3.6+
- Uses FastMCP framework
- Currently ~5,138 lines of code with 202 implemented features
- Relies on DaVinci Resolve's Python Scripting API

**Current Platform Support:**
- ✅ macOS (primary development platform, 8% features verified)
- ✅ Windows (secondary platform, needs more testing)
- ❌ Linux (NOT SUPPORTED - this is what you need to fix)

**Why Linux Support Matters:**
- Many video editors and developers use Linux
- DaVinci Resolve has a Linux version
- Expands user base significantly
- Community has requested this feature

---

## Critical First Steps

### Step 1: Verify DaVinci Resolve API Availability on Linux

**THIS IS CRITICAL - DO THIS FIRST!**

Before implementing anything, you MUST verify:

1. **Does DaVinci Resolve on Linux include the Scripting API?**
   - Check if the Python API modules exist
   - Verify if the fusionscript library is available
   - Confirm which Resolve versions on Linux support scripting

2. **Where are the API files located on Linux?**
   - Research typical installation paths
   - Check both free and Studio versions
   - Look for distribution-specific variations (Ubuntu vs Fedora vs Arch)

3. **Document your findings:**
   - If API is NOT available on Linux, this task cannot proceed
   - If API exists, document exact paths and requirements
   - Note any version restrictions or limitations

**How to research this:**
- Search DaVinci Resolve Linux documentation
- Check Blackmagic Design developer forums
- Look for existing Python scripts that work on Linux
- If possible, test on an actual Linux system with Resolve installed

**Expected paths to investigate:**
```bash
# Possible Linux locations (VERIFY THESE!)
/opt/resolve/Developer/Scripting/
/usr/share/DaVinciResolve/
/opt/DaVinciResolve/
~/.local/share/DaVinciResolve/
```

---

## Step 2: Understand Current Platform Detection

**Read and analyze these files:**

1. **`src/utils/platform.py`** (PRIMARY FILE)
   - Currently detects macOS and Windows
   - Returns platform-specific paths for Resolve API
   - You need to add Linux detection here

2. **`src/main.py`** (Entry point)
   - Sets environment variables based on platform
   - Needs to handle Linux paths

3. **`scripts/check-resolve-ready.sh`** (macOS script)
   - Checks if Resolve is running
   - Verifies environment variables
   - Use as template for Linux version

**Current platform detection logic:**
```python
# Examines sys.platform
# macOS: 'darwin'
# Windows: 'win32'
# Linux: 'linux' or 'linux2'
```

---

## Step 3: Implementation Plan

Once you've verified the API exists on Linux, proceed with:

### A. Update Platform Detection

**File:** `src/utils/platform.py`

Add Linux support to:
- `get_platform()` - detect 'linux'
- `get_resolve_paths()` - return Linux-specific paths
- Handle multiple distributions if paths vary

**Expected return format:**
```python
{
    "api_path": "/path/to/Scripting",
    "lib_path": "/path/to/fusionscript.so",
    "modules_path": "/path/to/Scripting/Modules"
}
```

### B. Create Linux Launcher Scripts

**Create:** `scripts/mcp_resolve-linux_start`

Based on existing launchers:
- `scripts/mcp_resolve-cursor_start` (macOS)
- `scripts/mcp_resolve-claude_start` (macOS)

Requirements:
- Bash script with shebang `#!/usr/bin/env bash`
- Detect if Resolve is running (`ps aux | grep -i "resolve"`)
- Set environment variables
- Activate virtual environment
- Launch `python src/main.py`

### C. Update Universal Launcher

**File:** `scripts/mcp_resolve_launcher.sh`

Add Linux branch to platform detection:
```bash
case "$(uname -s)" in
    Darwin*)  # macOS
    MINGW*)   # Windows
    Linux*)   # Add this!
esac
```

### D. Create Linux Configuration

**Create:** `config/linux/`

Based on existing configs:
- `config/macos/`
- `config/windows/`

Include:
- Sample MCP configuration for Claude Desktop
- Sample MCP configuration for Cursor
- README with Linux-specific setup instructions

**Note:** Linux paths in JSON configs:
- Use forward slashes `/`
- Provide both system-wide and user-local examples
- Consider XDG Base Directory specification

### E. Update Check Scripts

**Update:** `scripts/check-resolve-ready.sh`

Add Linux-specific checks:
- Process detection for Resolve
- Environment variable validation
- Python version check
- Dependency verification

**Consider creating:** `scripts/check-resolve-ready-linux.sh` if needed

---

## Step 4: Testing & Validation

### Test on Multiple Distributions

**Minimum test targets:**
- Ubuntu 22.04 LTS or newer (Debian-based)
- Fedora 38 or newer (Red Hat-based)

**Ideal additional testing:**
- Arch Linux
- Linux Mint
- Pop!_OS

### Verification Checklist

- [ ] `src/utils/platform.py` detects Linux correctly
- [ ] Resolve API paths are found automatically
- [ ] Environment variables set correctly
- [ ] `python src/main.py` starts without errors
- [ ] Can connect to running DaVinci Resolve instance
- [ ] Basic operations work (list projects, get timeline info)
- [ ] Launcher scripts have execute permissions
- [ ] Configuration examples work with Claude Desktop
- [ ] Configuration examples work with Cursor

### Feature Testing

Test critical features from `docs/FEATURES.md`:
- Project operations (list, open, create)
- Timeline operations (get info, create, modify)
- Media pool operations (import, organize)
- Basic rendering operations

**Goal:** Achieve at least 90% feature compatibility

---

## Step 5: Documentation Updates

Update these files with Linux information:

1. **`README.md`**
   - Add Linux to supported platforms
   - Include Linux installation instructions
   - Add Linux troubleshooting section

2. **`INSTALL.md`**
   - Add Linux installation guide
   - Include distribution-specific notes
   - Document dependency installation (Ubuntu: apt, Fedora: dnf)

3. **`config/README.md`**
   - Add Linux configuration examples
   - Explain path differences from macOS/Windows

4. **`CLAUDE.md`** (AI Assistant Guide)
   - Update platform support section
   - Add Linux-specific paths
   - Include Linux troubleshooting

5. **`FUTURE_TASKS.md`**
   - Mark task as completed
   - Move to "Completed Tasks" section
   - Add completion date and notes

6. **`CHANGELOG.md`**
   - Add entry for Linux support
   - Note any limitations or known issues

---

## Important Considerations

### Distribution Differences

**Package names vary:**
- Ubuntu/Debian: `python3-pip`, `python3-venv`
- Fedora/RHEL: `python3-pip`, `python3-virtualenv`
- Arch: `python-pip`, `python-virtualenv`

**Installation paths may differ:**
- DEB packages often use `/opt/`
- RPM packages might use `/usr/share/`
- Manual installs could be anywhere

**Solution:** Document multiple paths, use automatic detection

### File Permissions

Linux requires execute permissions on scripts:
```bash
chmod +x scripts/mcp_resolve-linux_start
chmod +x scripts/check-resolve-ready.sh
```

Remember to set these in your implementation.

### Virtual Environment

Standard approach works on Linux:
```bash
python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt
```

Ensure launcher scripts activate venv correctly.

### Desktop Integration

Consider adding:
- `.desktop` file for application launcher
- Installation script that sets up paths
- System service file for background running (optional)

---

## Known Challenges & Risks

### High Risk Issues

1. **API May Not Exist**
   - DaVinci Resolve on Linux might not include Scripting API
   - If true, Linux support is impossible without Blackmagic changes
   - VERIFY THIS FIRST!

2. **Library Dependencies**
   - `fusionscript.so` might have unmet system dependencies
   - May require specific glibc versions
   - Could conflict with system Python

3. **Version Fragmentation**
   - Free vs Studio versions may differ
   - Different Resolve versions may have different paths
   - Distribution packaging could affect structure

### Medium Risk Issues

1. **Path Detection Complexity**
   - Multiple possible installation locations
   - User installations vs system installations
   - Flatpak/Snap/AppImage installations need special handling

2. **Testing Limitations**
   - May not have access to all distributions
   - Resolve might not run in containerized environments
   - Need real hardware for testing

---

## Success Criteria

You will have successfully completed this task when:

1. ✅ Verified DaVinci Resolve Scripting API exists on Linux
2. ✅ Platform detection works on Linux
3. ✅ All paths automatically detected correctly
4. ✅ Launcher scripts work on at least 2 distributions
5. ✅ Configuration examples provided
6. ✅ All documentation updated
7. ✅ At least 90% of features tested and working
8. ✅ No regressions on macOS or Windows
9. ✅ Clean commit with proper testing

---

## Files to Modify/Create

**Modify:**
- `src/utils/platform.py` - Add Linux detection
- `src/main.py` - Handle Linux environment
- `scripts/mcp_resolve_launcher.sh` - Add Linux support
- `scripts/check-resolve-ready.sh` - Linux compatibility
- `README.md` - Document Linux support
- `INSTALL.md` - Linux installation guide
- `CLAUDE.md` - Update platform info
- `docs/FEATURES.md` - Update testing status
- `CHANGELOG.md` - Add version entry

**Create:**
- `scripts/mcp_resolve-linux_start` - Linux launcher
- `config/linux/` - Configuration directory
- `config/linux/claude-desktop-config.json` - Example config
- `config/linux/cursor-config.json` - Example config
- `config/linux/README.md` - Linux setup guide

---

## Getting Started

**Your first actions should be:**

1. Read `CLAUDE.md` to understand the codebase
2. Read `src/utils/platform.py` to see current implementation
3. Research DaVinci Resolve on Linux (API availability)
4. If API exists, document the paths you find
5. Create a branch: `git checkout -b feature/linux-support`
6. Begin implementation following the steps above

**Research Resources:**
- Blackmagic Design Developer Documentation
- DaVinci Resolve Linux forums
- GitHub issues in similar projects
- Linux Resolve user communities

---

## Questions to Answer During Implementation

Document answers to these:

1. Does the Scripting API exist on Linux? (YES/NO - CRITICAL)
2. What are the exact paths on Ubuntu?
3. What are the exact paths on Fedora?
4. Are paths different for Resolve vs Resolve Studio?
5. What Python version does Linux Resolve use?
6. Are there any additional system dependencies?
7. Does the API work the same as macOS/Windows?
8. Are there any Linux-specific limitations?
9. What's the minimum Resolve version that works?
10. Do Flatpak/Snap versions work differently?

---

## Final Notes

- **Be thorough** - Linux users expect things to "just work"
- **Test extensively** - Don't assume behavior from macOS/Windows
- **Document everything** - Linux has many distributions
- **Ask for help** - If API doesn't exist, escalate immediately
- **Keep it simple** - Don't over-engineer the solution

**Remember:** The #1 blocker is whether the Scripting API exists on Linux. Verify this before writing any code!

---

## Completion Checklist

When you finish, ensure:

- [ ] All code changes committed
- [ ] All tests pass
- [ ] Documentation updated
- [ ] Examples provided
- [ ] No regressions on macOS/Windows
- [ ] FUTURE_TASKS.md updated
- [ ] CHANGELOG.md entry added
- [ ] Pull request created
- [ ] Linux testing results documented

---

**Good luck! This is an important feature that will help many users.**

**Agent, you may begin. Start by researching DaVinci Resolve API availability on Linux.**
