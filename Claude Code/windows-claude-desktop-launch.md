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

If installed from the Microsoft Store, the path is under `WindowsApps`. The version number changes with every update:

```powershell
# Find your current path
Get-ChildItem "C:\Program Files\WindowsApps\Claude*" -Recurse -Filter "claude.exe" -ErrorAction SilentlyContinue | Select-Object FullName
```

### Launching via App ID (recommended for Store apps)

```powershell
explorer.exe shell:AppsFolder\Claude_pzs8sxrjxfjjc!Claude
```

> **Note:** `Start-Process` with the full WindowsApps path often fails with "Otro programa está utilizando este archivo" due to the `CoworkVMService` Windows service. Use `explorer.exe shell:AppsFolder\...` or launch from the Start Menu instead.

### Finding Your App ID

```powershell
Get-StartApps | Where-Object { $_.Name -like '*Claude*' }
```

## Opening Multiple Instances

Claude Desktop enforces a single-instance lock via Electron's `app.requestSingleInstanceLock()`. To open a **second independent instance**, use the `--user-data-dir` flag:

```powershell
Start-Process "C:\Program Files\WindowsApps\Claude_<VERSION>_x64__pzs8sxrjxfjjc\app\claude.exe" -ArgumentList "--user-data-dir=$env:LOCALAPPDATA\claude-instance-2"
```

### Important Notes on Authentication

- Each `--user-data-dir` profile requires **its own authentication** (one time only).
- Copying the profile from the main instance does NOT transfer the session because Electron/Chromium encrypts cookies and tokens using DPAPI, which binds the encryption to the specific profile path.
- Using NTFS junctions (`mklink /J`) to the original profile fails because the single-instance lockfile blocks the second instance.
- After authenticating once, the second instance remembers your session permanently.

### Desktop Shortcut for Second Instance (auto-detects version)

Create a `.bat` file that automatically finds the current Claude version:

```powershell
Set-Content "$env:USERPROFILE\Desktop\Claude (2).bat" @'
@echo off
for /f "delims=" %%i in ('powershell -NoProfile -Command "(Get-ChildItem 'C:\Program Files\WindowsApps\Claude*' -Recurse -Filter 'claude.exe' -ErrorAction SilentlyContinue | Select-Object -First 1).FullName"') do set CLAUDE=%%i
start "" "%CLAUDE%" --user-data-dir=%LOCALAPPDATA%\claude-instance-2
'@
```

This shortcut survives Claude updates because it dynamically finds the executable path.

## CoworkVMService (Background Service)

Claude Desktop registers a Windows service called `CoworkVMService` that starts automatically and can block the executable:

```powershell
# Check the service
Get-WmiObject Win32_Service | Where-Object { $_.PathName -like '*cowork*' } | Select-Object Name, State, StartMode

# Stop the service (requires Administrator)
Stop-Service -Name "CoworkVMService" -Force
```

### Kill Everything (Nuclear Option)

When Claude is stuck, won't open, or shows "Otro programa está utilizando este archivo":

```powershell
# Run in Administrator terminal
taskkill /F /IM "claude.exe" /T; taskkill /F /IM "cowork-svc.exe" /T; taskkill /F /IM "chrome-native-host.exe" /T; Stop-Service -Name "CoworkVMService" -Force -ErrorAction SilentlyContinue; Start-Sleep 3; Get-Process | Where-Object { $_.Path -like '*Claude*' -or $_.Path -like '*cowork*' } | Stop-Process -Force -ErrorAction SilentlyContinue
```

Then reopen Claude from the Start Menu.

## Troubleshooting

- **"Otro programa está utilizando este archivo"**: The `CoworkVMService` or a ghost Claude process is running. Use the "Kill Everything" command above, then reopen.
- **"Unable to move the cache" / "Unable to create cache"**: Launched from an elevated (Administrator) terminal. Use a non-admin terminal or the Start Menu instead.
- **Claude doesn't open after update**: The version number in the path changed. Use `Get-ChildItem` to find the new path, or use the `.bat` shortcut which auto-detects it.
- **Blank window on second instance**: Corrupted profile data. Delete the profile directory and re-authenticate:
  ```powershell
  Remove-Item "$env:LOCALAPPDATA\claude-instance-2" -Recurse -Force
  ```
- **`where.exe Claude.exe` returns nothing**: Only works from elevated terminals for Store apps. Use `Get-ChildItem` instead.
- **`CoworkVMService` keeps restarting**: It's registered as an Auto-start Windows service managed by `services.exe` (PID 1220). Stop it with `Stop-Service -Name "CoworkVMService" -Force` from an Admin terminal.

## Optional: PowerShell Alias

Add this to your PowerShell profile (`$PROFILE`) for quick access:

```powershell
function claude {
    $exe = Get-ChildItem "C:\Program Files\WindowsApps\Claude*" -Recurse -Filter "claude.exe" -ErrorAction SilentlyContinue | Select-Object -First 1 -ExpandProperty FullName
    if ($exe) { Start-Process $exe } else { Write-Error "Claude Desktop not found" }
}
```

Then simply run `claude` from any PowerShell terminal.
