```


\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\

$downloads = "C:\Users\labuser\Downloads"
$sysmon = Join-Path $downloads "Sysmon"

$configs = Get-ChildItem "$downloads\sysmon-config-master" `
    -Recurse -File -Filter "*.xml" |
    Where-Object {
        $_.Name -match "sysmon.*config|config.*sysmon"
    }

$configs | Select-Object FullName, Length



$validConfigs = @(
    foreach ($file in $configs) {
        try {
            [xml]$xml = Get-Content $file.FullName -Raw

            if (
                $xml.DocumentElement.LocalName -eq "Sysmon" -and
                $xml.DocumentElement.SelectSingleNode(
                    "*[local-name()='EventFiltering']"
                )
            ) {
                $file
            }
        }
        catch {}
    }
)

$validConfigs | Select-Object FullName, Length

if ($validConfigs.Count -ne 1) {
    throw "Expected one valid Sysmon configuration. Found $($validConfigs.Count). Do not continue until the correct file is selected."
}

$configFile = $validConfigs[0].FullName

Copy-Item $configFile "$sysmon\sysmonconfig.xml" -Force

Write-Host "Valid Sysmon XML copied successfully."



cd "C:\Users\labuser\Downloads\Sysmon"

.\Sysmon64.exe -c ".\sysmonconfig.xml"

if ($LASTEXITCODE -ne 0) {
    throw "Sysmon configuration update failed."
}


.\Sysmon64.exe -c |
Select-String "Config file|Service name|Driver name"

Get-Service Sysmon64

Start-Process notepad.exe

Start-Sleep -Seconds 3

Get-WinEvent -FilterHashtable @{
    LogName = "Microsoft-Windows-Sysmon/Operational"
    Id = 1
} -MaxEvents 10 |
Select-Object TimeCreated, Id, Message |
Format-List

Get-WinEvent -FilterHashtable @{
    LogName = "Microsoft-Windows-Sysmon/Operational"
    Id = 3
} -MaxEvents 20 -ErrorAction SilentlyContinue |
Where-Object {
    $_.Message -match "192\.168\.56\.11"
} |
Format-List TimeCreated, Id, Message




```

```
# 1. Find Sysmon services
Get-Service |
Where-Object {
    $_.Name -like "*Sysmon*" -or
    $_.DisplayName -like "*Sysmon*"
} |
Format-Table Name, DisplayName, Status

# 2. Check Sysmon drivers
Get-CimInstance Win32_SystemDriver |
Where-Object {
    $_.Name -like "*Sysmon*"
} |
Select-Object Name, State, StartMode

# 3. Check Sysmon configuration
cd "C:\Users\labuser\Downloads\Sysmon"
.\Sysmon64.exe -c

# 4. Check Sysmon event log
Get-WinEvent -ListLog "Microsoft-Windows-Sysmon/Operational" |
Select-Object LogName, IsEnabled, RecordCount

# 5. Generate a new process event
Start-Process notepad.exe
Start-Sleep -Seconds 5

# 6. Find the Notepad event
Get-WinEvent -FilterHashtable @{
    LogName = "Microsoft-Windows-Sysmon/Operational"
    Id = 1
    StartTime = (Get-Date).AddMinutes(-10)
} -ErrorAction SilentlyContinue |
Where-Object {
    $_.Message -match "notepad.exe"
} |
Select-Object -First 3 TimeCreated, Id, Message |
Format-List
```
https://github.com/SwiftOnSecurity/sysmon-config/blob/master/sysmonconfig-export.xml?utm_source=chatgpt.com
