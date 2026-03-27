# Opening Claude Desktop on Windows (Equivalent of macOS `open -n`)

On macOS, you can open a new instance of Claude Desktop with:

```bash
open -n /Applications/Claude.app
```

## Windows Equivalents

### PowerShell

```powershell
Start-Process "$env:LOCALAPPDATA\Programs\claude-desktop\Claude.exe"
```

### Command Prompt (cmd)

```cmd
start "" "%LOCALAPPDATA%\Programs\claude-desktop\Claude.exe"
```

> **Note:** The `""` after `start` is the window title parameter and is required when the path contains quotes.

## Opening Multiple Instances

On Windows, running the command multiple times will open additional instances. The macOS `-n` flag (force new instance) has no direct equivalent because `start` already launches a new process each time.

## Optional: Create a PowerShell Alias

Add this to your PowerShell profile (`$PROFILE`) for quick access:

```powershell
function claude { Start-Process "$env:LOCALAPPDATA\Programs\claude-desktop\Claude.exe" }
```

Then simply run `claude` from any PowerShell terminal.
