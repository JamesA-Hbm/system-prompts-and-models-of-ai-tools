# Opening Claude Desktop on Windows (Equivalent of macOS `open -n`)

On macOS, you can open a new instance of Claude Desktop with:

```bash
open -n /Applications/Claude.app
```

## Windows Equivalents

The installation path depends on how Claude Desktop was installed:

### Standalone Installer (default path)

```powershell
Start-Process "$env:LOCALAPPDATA\Programs\claude-desktop\Claude.exe"
```

### Microsoft Store Installation

If installed from the Microsoft Store, the path is under `WindowsApps`:

```powershell
Start-Process "C:\Program Files\WindowsApps\Claude_1.1.9310.0_x64__pzs8sxrjxfijc\app\claude.exe"
```

> **Note:** The version number in the path (e.g., `1.1.9310.0`) changes with updates. Use the dynamic method below to avoid hardcoding it.

### Universal Method (works for any installation)

Use `where.exe` to automatically find the executable regardless of install method:

```powershell
Start-Process (where.exe Claude.exe)
```

### Command Prompt (cmd)

```cmd
start "" "%LOCALAPPDATA%\Programs\claude-desktop\Claude.exe"
```

> **Note:** The `""` after `start` is the window title parameter and is required when the path contains quotes.

### Finding Your Installation Path

If the default paths don't work, find where Claude is installed:

```powershell
where.exe Claude.exe
```

## Opening Multiple Instances

On Windows, running the command multiple times will open additional instances. The macOS `-n` flag (force new instance) has no direct equivalent because `start` already launches a new process each time.

## Troubleshooting

- **"Unable to move the cache" / "Unable to create cache" errors**: This happens when launching from an elevated (Administrator) terminal. Run the command from a **non-administrator** terminal instead.
- **"El sistema no puede encontrar el archivo"**: The path is wrong for your installation type. Run `where.exe Claude.exe` to find the correct path.

## Optional: Create a PowerShell Alias

Add this to your PowerShell profile (`$PROFILE`) for quick access:

```powershell
function claude { Start-Process (where.exe Claude.exe) }
```

Then simply run `claude` from any PowerShell terminal.
