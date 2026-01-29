# Linux Setup Guide for DaVinci Resolve MCP Server

This guide provides Linux-specific instructions for setting up and using the DaVinci Resolve MCP Server.

## Prerequisites

### 1. DaVinci Resolve Installation

DaVinci Resolve should be installed in the default location:
- **Default path**: `/opt/resolve/`
- **Scripts API**: `/opt/resolve/Developer/Scripting/`
- **Fusion library**: `/opt/resolve/libs/Fusion/fusionscript.so`

If you have DaVinci Resolve installed in a custom location, you'll need to update the paths in `src/utils/platform.py`.

### 2. Python 3.6+

Ensure you have Python 3.6 or later installed:
```bash
python3 --version
```

If Python is not installed, install it using your distribution's package manager:

**Ubuntu/Debian:**
```bash
sudo apt update
sudo apt install python3 python3-pip python3-venv
```

**Fedora/RHEL:**
```bash
sudo dnf install python3 python3-pip
```

**Arch Linux:**
```bash
sudo pacman -S python python-pip
```

### 3. Development Tools

Some distributions may require additional development tools:
```bash
# Ubuntu/Debian
sudo apt install build-essential

# Fedora/RHEL
sudo dnf groupinstall "Development Tools"

# Arch Linux
sudo pacman -S base-devel
```

## Installation

### Quick Installation (Recommended)

1. Clone the repository:
```bash
git clone https://github.com/samuelgursky/davinci-resolve-mcp.git
cd davinci-resolve-mcp
```

2. Make scripts executable:
```bash
chmod +x scripts/*.sh scripts/setup/*.sh
```

3. Start DaVinci Resolve

4. Run the installation script:
```bash
./scripts/setup/install.sh
```

This will automatically:
- Create a Python virtual environment
- Install all dependencies
- Set up environment variables
- Configure Cursor integration
- Verify the installation

### Manual Installation

If you prefer to install manually:

1. Create a virtual environment:
```bash
python3 -m venv venv
source venv/bin/activate
```

2. Install dependencies:
```bash
pip install -r requirements.txt
```

3. Set up environment variables:
```bash
export RESOLVE_SCRIPT_API="/opt/resolve/Developer/Scripting"
export RESOLVE_SCRIPT_LIB="/opt/resolve/libs/Fusion/fusionscript.so"
export PYTHONPATH="$PYTHONPATH:$RESOLVE_SCRIPT_API/Modules/"
```

Add these to your `~/.bashrc` or `~/.zshrc` for persistence:
```bash
echo 'export RESOLVE_SCRIPT_API="/opt/resolve/Developer/Scripting"' >> ~/.bashrc
echo 'export RESOLVE_SCRIPT_LIB="/opt/resolve/libs/Fusion/fusionscript.so"' >> ~/.bashrc
echo 'export PYTHONPATH="$PYTHONPATH:$RESOLVE_SCRIPT_API/Modules/"' >> ~/.bashrc
source ~/.bashrc
```

4. Configure Cursor (create `~/.cursor/mcp.json`):
```json
{
  "mcpServers": {
    "davinci-resolve": {
      "name": "DaVinci Resolve MCP",
      "command": "/path/to/davinci-resolve-mcp/venv/bin/python",
      "args": ["/path/to/davinci-resolve-mcp/src/main.py"]
    }
  }
}
```

## Running the Server

### Quick Start

1. Ensure DaVinci Resolve is running
2. Run the server:
```bash
./scripts/run-now.sh
```

### Pre-Launch Check

Verify your environment before starting:
```bash
./scripts/check-resolve-ready.sh
```

This will check:
- DaVinci Resolve is running
- Environment variables are set
- Virtual environment is configured
- Cursor configuration is correct

## Troubleshooting

### DaVinci Resolve Not Found

If the server can't find DaVinci Resolve, verify:

1. DaVinci Resolve is running:
```bash
pgrep -i resolve
```

2. Installation path is correct:
```bash
ls -l /opt/resolve/
```

3. Environment variables are set:
```bash
echo $RESOLVE_SCRIPT_API
echo $RESOLVE_SCRIPT_LIB
```

### Custom Installation Path

If DaVinci Resolve is installed in a non-standard location, update `src/utils/platform.py`:

```python
elif platform_name == 'linux':  # Linux
    api_path = "/your/custom/path/Developer/Scripting"
    lib_path = "/your/custom/path/libs/Fusion/fusionscript.so"
    modules_path = os.path.join(api_path, "Modules")
```

### Permission Issues

If you encounter permission errors:

1. Make scripts executable:
```bash
chmod +x scripts/*.sh scripts/setup/*.sh
```

2. Ensure DaVinci Resolve files are readable:
```bash
ls -l /opt/resolve/Developer/Scripting/
ls -l /opt/resolve/libs/Fusion/fusionscript.so
```

### Python Import Errors

If you get import errors:

1. Verify Python path:
```bash
echo $PYTHONPATH
```

2. Check if DaVinciResolveScript module exists:
```bash
ls -l /opt/resolve/Developer/Scripting/Modules/
```

3. Test import manually:
```bash
source venv/bin/activate
python3 -c "import sys; sys.path.append('/opt/resolve/Developer/Scripting/Modules/'); import DaVinciResolveScript"
```

### Cursor Integration Issues

If Cursor can't connect to the server:

1. Verify configuration file exists:
```bash
cat ~/.cursor/mcp.json
```

2. Check paths in configuration are absolute paths
3. Restart Cursor after making configuration changes

## Platform-Specific Notes

### Ubuntu/Debian

- Default shell is usually bash
- Environment variables should be in `~/.bashrc`

### Fedora/RHEL/CentOS

- May use bash or zsh
- Check which shell: `echo $SHELL`
- Add variables to appropriate config file

### Arch Linux

- Default shell varies by installation
- May need to install `python-pip` separately

### System-wide Installation

If DaVinci Resolve was installed system-wide with different permissions, you may need to run some commands with `sudo`. However, the MCP server itself should run as your user, not as root.

## Advanced Configuration

### Running as a Service

To run the MCP server as a systemd service:

1. Create `/etc/systemd/system/davinci-resolve-mcp.service`:
```ini
[Unit]
Description=DaVinci Resolve MCP Server
After=network.target

[Service]
Type=simple
User=YOUR_USERNAME
WorkingDirectory=/path/to/davinci-resolve-mcp
Environment="RESOLVE_SCRIPT_API=/opt/resolve/Developer/Scripting"
Environment="RESOLVE_SCRIPT_LIB=/opt/resolve/libs/Fusion/fusionscript.so"
Environment="PYTHONPATH=/opt/resolve/Developer/Scripting/Modules/"
ExecStart=/path/to/davinci-resolve-mcp/venv/bin/python /path/to/davinci-resolve-mcp/src/main.py
Restart=on-failure

[Install]
WantedBy=multi-user.target
```

2. Enable and start the service:
```bash
sudo systemctl daemon-reload
sudo systemctl enable davinci-resolve-mcp
sudo systemctl start davinci-resolve-mcp
```

3. Check status:
```bash
sudo systemctl status davinci-resolve-mcp
```

### Multiple DaVinci Resolve Versions

If you have multiple versions of DaVinci Resolve installed, update the paths to point to the version you want to use.

## Getting Help

If you encounter issues:

1. Check the logs: `cat scripts/cursor_resolve_server.log`
2. Run the verification script: `./scripts/check-resolve-ready.sh`
3. Check environment variables are set correctly
4. Ensure DaVinci Resolve is running
5. Create an issue on GitHub with details about your:
   - Linux distribution and version
   - DaVinci Resolve version
   - Python version
   - Error messages

## Contributing

Linux-specific improvements are welcome! If you encounter issues or have suggestions for better Linux support, please open an issue or pull request on GitHub.
