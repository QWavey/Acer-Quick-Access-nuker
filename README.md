# 1. Force kill all possible active UI components right now
Stop-Process -Name "AQAUserPS", "QAAdmin", "QAAgent" -Force -ErrorAction SilentlyContinue 

# 2. Hunt down every copy of AQAUserPS.exe and QAAdmin.exe, disable their services, and neutralize them
Get-ChildItem -Path "C:\Program Files\Acer" -Filter "*.exe" -Recurse -ErrorAction SilentlyContinue | 
    Where-Object { $_.Name -match "AQAUserPS|QAAdmin" } | ForEach-Object {
        # Extra kill command right before modification to ensure it's not locked
        Stop-Process -Name $_.BaseName -Force -ErrorAction SilentlyContinue
        Get-Service -Name $_.BaseName -ErrorAction SilentlyContinue | Set-Service -StartupType Disabled -ErrorAction SilentlyContinue
        Rename-Item -Path $_.FullName -NewName "$($_.Name).bak" -Force -ErrorAction SilentlyContinue
    }


# if acer predator sense seems stuck:

# 1. Bring the Acer Quick Access framework completely back to default factory settings 
# This rules out any missing engine errors causing PredatorSense to stall
Get-ChildItem -Path "C:\ProgramData\OEM\*" -Filter "*.bak" -Recurse -ErrorAction SilentlyContinue | ForEach-Object {
    Rename-Item -Path $_.FullName -NewName ($_.Name -replace '\.bak$', '') -Force -ErrorAction SilentlyContinue
}
"AASSvc", "AcerQAAgentSvis", "ASMSvc" | ForEach-Object {
    Set-Service -Name $_ -StartupType Automatic -ErrorAction SilentlyContinue
    Start-Service -Name $_ -ErrorAction SilentlyContinue
}

# 2. Fix the known Audio Pipeline Hook Conflict (DTSX Ultra)
# We set it to Manual, force kill it, and let PredatorSense wake it up cleanly
Set-Service -Name "DtsApo4Service" -StartupType Manual -ErrorAction SilentlyContinue
Stop-Service -Name "DtsApo4Service" -Force -ErrorAction SilentlyContinue

# 3. Aggressively clear out frozen system sensor hooks
Stop-Process -Name "PredatorSense*", "PSSvc*", "AQAUserPS*" -Force -ErrorAction SilentlyContinue
lodctr /R; Set-Location C:\Windows\SysWow64; lodctr /R
Restart-Service -Name "ASMSvc" -Force -ErrorAction SilentlyContinue

# 4. Apply the silent block to the keybind popup executable without breaking any background files
$IFEOPath = "HKLM:\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Image File Execution Options\AQAUserPS.exe"
if (-not (Test-Path $IFEOPath)) { New-Item -Path $IFEOPath -Force | Out-Null }
Set-ItemProperty -Path $IFEOPath -Name "Debugger" -Value "cmd.exe /c exit"

Write-Host "DEEP REPAIR COMPLETE. PLEASE RESTART YOUR LAPTOP NOW." -ForegroundColor Green
