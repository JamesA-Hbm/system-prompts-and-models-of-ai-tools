# Opening Claude Desktop on Windows (Equivalent of macOS `open -n`)

On macOS, you can open a new instance of Claude Desktop with:

```bash
open -n /Applications/Claude.app
```

## Windows: Running Two Instances Side by Side

The solution uses `--user-data-dir` to bypass Electron's single-instance lock. Both instances share the same authentication by pointing to the original profile directory.

### Setup (one time)

Create three `.bat` files on your Desktop by running these commands in PowerShell:

**Claude (1).bat** - First instance:
```powershell
Set-Content "$env:USERPROFILE\Desktop\Claude (1).bat" @'
@echo off
for /f "delims=" %%i in ('powershell -NoProfile -Command "(Get-ChildItem 'C:\Program Files\WindowsApps\Claude*' -Recurse -Filter 'claude.exe' -ErrorAction SilentlyContinue | Select-Object -First 1).FullName"') do set CLAUDE=%%i
start "" "%CLAUDE%" --user-data-dir=%APPDATA%\Claude
'@
```

**Claude (2).bat** - Second instance (uses a copy of the profile):
```powershell
Set-Content "$env:USERPROFILE\Desktop\Claude (2).bat" @'
@echo off
for /f "delims=" %%i in ('powershell -NoProfile -Command "(Get-ChildItem 'C:\Program Files\WindowsApps\Claude*' -Recurse -Filter 'claude.exe' -ErrorAction SilentlyContinue | Select-Object -First 1).FullName"') do set CLAUDE=%%i
start "" "%CLAUDE%" --user-data-dir=%APPDATA%\Claude2
'@
```

**Fix Claude.bat** - Rescue script for when Claude gets stuck after updates (run as Administrator):
```powershell
Set-Content "$env:USERPROFILE\Desktop\Fix Claude.bat" @'
@echo off
echo Cerrando Claude y servicios...
taskkill /F /IM "claude.exe" /T 2>nul
taskkill /F /IM "cowork-svc.exe" /T 2>nul
taskkill /F /IM "chrome-native-host.exe" /T 2>nul
net stop CoworkVMService 2>nul
timeout /t 5 /nobreak
echo.
echo Claude cerrado. Ahora abre Claude (1).bat y Claude (2).bat
pause
'@
```

Copy the profile for the second instance:
```powershell
robocopy "$env:APPDATA\Claude" "$env:APPDATA\Claude2" /MIR /R:0 /W:0 /NFL /NDL /NJH /NJS
```

### Daily Usage

1. Double-click **Claude (1).bat**
2. Double-click **Claude (2).bat**
3. Both instances open authenticated with your account

### After a Claude Update

If the `.bat` files show "Otro programa está utilizando este archivo":

1. Right-click **Fix Claude.bat** > **Run as Administrator**
2. Wait for "Claude cerrado"
3. Double-click **Claude (1).bat**, then **Claude (2).bat**

> **Note:** The `.bat` files auto-detect the Claude version path, so they survive updates without modification.

### Why `.bat` Files Instead of the Start Menu

- The Start Menu uses Electron's default launch which enforces single-instance lock
- Using `--user-data-dir` explicitly bypasses the lock, allowing multiple instances
- Both instances use the same `%APPDATA%\Claude` profile, so authentication is shared
- The `.bat` files must be run from PowerShell (`& "path\to\file.bat"`) or from a non-admin terminal

### Important: Profile Copying Limitations

- Copying the profile to a **different path** (e.g., `%LOCALAPPDATA%\claude-instance-2`) does NOT transfer the session in newer versions of Claude Desktop
- This is because Chromium's LevelDB databases (IndexedDB, Local Storage) get corrupted during copy and Electron deletes/recreates them
- The DPAPI encryption key (`os_crypt.encrypted_key` in `Local State`) is identical across copies, but the database corruption causes auth loss
- The solution is to use the **same original profile path** (`%APPDATA%\Claude`) for the first instance, and a robocopy mirror (`%APPDATA%\Claude2`) for the second

## CoworkVMService (Background Service)

Claude Desktop (Microsoft Store version) registers a Windows service called `CoworkVMService` that:
- Starts automatically with Windows
- Is managed by `services.exe` (PID 1220)
- Can block the Claude executable after updates
- Cannot be reconfigured via `sc.exe` (Access Denied) but can via registry:

```powershell
# Disable the service (value 4 = Disabled, 3 = Manual, 2 = Auto)
reg add "HKLM\SYSTEM\CurrentControlSet\Services\CoworkVMService" /v Start /t REG_DWORD /d 4 /f

# Re-enable after fixing
reg add "HKLM\SYSTEM\CurrentControlSet\Services\CoworkVMService" /v Start /t REG_DWORD /d 2 /f
```

## Finding Your Installation

```powershell
# Microsoft Store installation path
Get-ChildItem "C:\Program Files\WindowsApps\Claude*" -Recurse -Filter "claude.exe" -ErrorAction SilentlyContinue | Select-Object FullName

# App ID for Store apps
Get-StartApps | Where-Object { $_.Name -like '*Claude*' }

# Standalone installer path
"$env:LOCALAPPDATA\Programs\claude-desktop\Claude.exe"
```

## Troubleshooting

| Problem | Solution |
|---------|----------|
| "Otro programa está utilizando este archivo" | Run `Fix Claude.bat` as Admin, then reopen |
| "Unable to create cache" errors | Don't launch from Admin terminal |
| Claude won't open after update | Run `Fix Claude.bat` as Admin, may need reboot |
| `.bat` files don't work on double-click | Run from PowerShell: `& "$env:USERPROFILE\Desktop\Claude (1).bat"` |
| Second instance asks for authentication | Use `%APPDATA%\Claude2` with robocopy from original, not a different path |
| `CoworkVMService` keeps restarting | Disable via registry (see above), reboot, then re-enable after opening Claude |
