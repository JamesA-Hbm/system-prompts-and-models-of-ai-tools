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

### Dynamic Method (works for any installation)

Automatically finds the executable regardless of install method or version:

```powershell
$exe = Get-ChildItem "C:\Program Files\WindowsApps\Claude*" -Recurse -Filter "claude.exe" -ErrorAction SilentlyContinue | Select-Object -First 1 -ExpandProperty FullName
Start-Process $exe
```

> **Note:** `where.exe Claude.exe` only works from elevated (Administrator) terminals for Microsoft Store apps. The `Get-ChildItem` method works from any terminal.

### Command Prompt (cmd)

```cmd
start "" "%LOCALAPPDATA%\Programs\claude-desktop\Claude.exe"
```

> **Note:** The `""` after `start` is the window title parameter and is required when the path contains quotes.

### Finding Your Installation Path

```powershell
# For Microsoft Store installations
Get-ChildItem "C:\Program Files\WindowsApps\Claude*" -Recurse -Filter "claude.exe" -ErrorAction SilentlyContinue | Select-Object FullName

# For standalone installations (also works from elevated terminals for Store apps)
where.exe Claude.exe
```

## Opening Multiple Instances

Claude Desktop enforces a single-instance lock by default. Running the same command again will just focus the existing window instead of opening a new one. To open a **second independent instance**, use the `--user-data-dir` flag to specify a separate profile directory:

```powershell
# Open second instance with its own profile
Start-Process "C:\Program Files\WindowsApps\Claude_1.1.9310.0_x64__pzs8sxrjxfjjc\app\claude.exe" -ArgumentList "--user-data-dir=$env:TEMP\claude-instance-2"
```

> **Important:** Each instance with a different `--user-data-dir` requires its own authentication. Copying profile data from the main instance is not reliable because Electron locks session database files while running. Authenticate once on the second instance and it will remember your session for future launches.

## Troubleshooting

- **"Unable to move the cache" / "Unable to create cache" errors**: This happens when launching from an elevated (Administrator) terminal. Run the command from a **non-administrator** terminal instead.
- **"El sistema no puede encontrar el archivo"**: The path is wrong for your installation type. Use `Get-ChildItem` to find the correct path (see above).
- **Blank window on second instance**: This usually means corrupted profile data (e.g., from copying files while Claude was running). Close all instances with `Stop-Process -Name "claude" -Force`, delete the profile directory, and relaunch with a clean profile.
- **Cannot delete profile directory**: Close all Claude instances first with `Stop-Process -Name "claude" -Force` before deleting.

## Optional: Create a PowerShell Alias

Add this to your PowerShell profile (`$PROFILE`) for quick access:

```powershell
function claude {
    $exe = Get-ChildItem "C:\Program Files\WindowsApps\Claude*" -Recurse -Filter "claude.exe" -ErrorAction SilentlyContinue | Select-Object -First 1 -ExpandProperty FullName
    if ($exe) { Start-Process $exe } else { Write-Error "Claude Desktop not found" }
}
```

Then simply run `claude` from any PowerShell terminal.
