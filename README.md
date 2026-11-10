```


$root = "$env:USERPROFILE\Downloads\sysmon-modular-master\sysmon-modular-master"
$sysmon = "$env:USERPROFILE\Downloads\Sysmon"

# Find the Sysmon Modular merge script
$script = Join-Path $root "scripts\Merge-SysmonXml.ps1"

if (-not (Test-Path $script)) {
    throw "Sysmon Modular folder or merge script not found. Extract the ZIP into Downloads first."
}

if (-not (Test-Path (Join-Path $sysmon "Sysmon64.exe"))) {
    throw "Sysmon64.exe not found in the Sysmon folder."
}

# Allow the script in this PowerShell session
Set-ExecutionPolicy -Scope Process -ExecutionPolicy Bypass -Force

# Load the merge functions
. $script

# Merge all Sysmon Modular rules
$merged = Merge-AllSysmonXml -BasePath $root -AsString

# Check that the result is not empty
if ([string]::IsNullOrWhiteSpace($merged)) {
    throw "Merge failed. No configuration was generated."
}

# Validate the XML
[xml]$config = $merged

if ($config.DocumentElement.LocalName -ne "Sysmon" -or
    -not $config.DocumentElement.SelectSingleNode("EventFiltering")) {
    throw "Invalid Sysmon configuration. No changes made."
}

# Save the configuration
$output = Join-Path $sysmon "sysmonconfig.xml"
$merged | Set-Content $output -Encoding UTF8

Write-Host "XML created successfully: $output"

# Apply the configuration
Set-Location $sysmon
.\Sysmon64.exe -c $output

if ($LASTEXITCODE -ne 0) {
    throw "Sysmon configuration update failed."
}

Write-Host "Sysmon configuration applied successfully."






Get-Service -Name "*sysmon*" -ErrorAction SilentlyContinue

.\Sysmon64.exe -c 2>&1 |
Out-File "$env:TEMP\SysmonStatus.txt" -Width 250

Get-Content "$env:TEMP\SysmonStatus.txt" -TotalCount 35






Start-Process notepad.exe

Start-Sleep -Seconds 3

Get-WinEvent -FilterHashtable @{
    LogName = 'Microsoft-Windows-Sysmon/Operational'
    Id = 1
    StartTime = (Get-Date).AddMinutes(-10)
} -ErrorAction SilentlyContinue |
Where-Object { $_.Message -match 'notepad.exe' } |
Select-Object -First 3 TimeCreated, Id, Message |
Format-List





Test-NetConnection -ComputerName 192.168.56.10 -Port 445

Get-WinEvent -FilterHashtable @{
    LogName = 'Microsoft-Windows-Sysmon/Operational'
    Id = 3
    StartTime = (Get-Date).AddMinutes(-10)
} -ErrorAction SilentlyContinue |
Select-Object -First 3 TimeCreated, Id, Message |
Format-List
\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\


Get-ChildItem "$env:USERPROFILE\Downloads" -Recurse -Filter "Merge-SysmonXml.ps1" -ErrorAction SilentlyContinue |
Select-Object FullName


$sysmon = "$env:USERPROFILE\Downloads\Sysmon"

$script = Get-ChildItem "$env:USERPROFILE\Downloads" -Recurse -Filter "Merge-SysmonXml.ps1" -File -ErrorAction SilentlyContinue |
Select-Object -First 1

if (-not $script) {
    throw "Merge-SysmonXml.ps1 not found. Download and extract Sysmon Modular first."
}

$root = Split-Path $script.DirectoryName -Parent

Set-ExecutionPolicy -Scope Process -ExecutionPolicy Bypass -Force

. $script.FullName

$merged = Merge-AllSysmonXml -BasePath $root -AsString

if ([string]::IsNullOrWhiteSpace($merged)) {
    throw "Merge failed."
}

[xml]$config = $merged

if ($config.DocumentElement.LocalName -ne "Sysmon" -or
    -not $config.DocumentElement.SelectSingleNode("EventFiltering")) {
    throw "Invalid XML configuration."
}

$output = Join-Path $sysmon "sysmonconfig.xml"

$merged | Set-Content $output -Encoding UTF8

Write-Host "XML created successfully: $output"

Set-Location $sysmon

.\Sysmon64.exe -c $output

if ($LASTEXITCODE -ne 0) {
    throw "Sysmon configuration update failed."
}

Write-Host "Sysmon configuration applied successfully."


.\Sysmon64.exe -c 2>&1 |
Out-File "$env:TEMP\SysmonStatus.txt" -Width 250

Get-Content "$env:TEMP\SysmonStatus.txt" -TotalCount 35
```
