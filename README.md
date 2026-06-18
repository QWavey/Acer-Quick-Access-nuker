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
