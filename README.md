```

. .\Merge-SysmonXml.ps1



Get-Command Merge-AllSysmonXml -Syntax
Get-Command Merge-SysmonXml -Syntax


Get-Help Merge-AllSysmonXml -Full
Get-Help Merge-SysmonXml -Full


(Get-Command Merge-AllSysmonXml).Parameters.Keys
(Get-Command Merge-SysmonXml).Parameters.Keys
```


```
# 1. Define folders
$root = "C:\Users\Administrator\Downloads\sysmon-modular-master\sysmon-modular-master"
$sysmon = "C:\Users\Administrator\Downloads\Sysmon"

# 2. Find complete Sysmon configuration files
$configs = Get-ChildItem $root -Recurse -Filter "*.xml" |
Where-Object {
    try {
        [xml]$xml = Get-Content $_.FullName -Raw
        $xml.DocumentElement.LocalName -eq "Sysmon" -and
        $null -ne $xml.DocumentElement.SelectSingleNode("EventFiltering")
    }
    catch {
        $false
    }
}

# 3. Show possible configuration files
$configs | Select-Object FullName

# 4. Select a complete template if available
$template = $configs |
Where-Object { $_.Name -eq "sysmon_template.xml" } |
Select-Object -First 1

# 5. Stop safely if the template was not found
if (-not $template) {
    Write-Host "Template not found. No changes were made." -ForegroundColor Yellow
}
else {
    # 6. Copy the template
    Copy-Item $template.FullName "$sysmon\sysmonconfig.xml" -Force

    # 7. Verify XML
    [xml]$check = Get-Content "$sysmon\sysmonconfig.xml" -Raw
    Write-Host "XML created successfully."

    # 8. Apply Sysmon configuration
    & "$sysmon\Sysmon64.exe" -c "$sysmon\sysmonconfig.xml"

    # 9. Verify active configuration
    & "$sysmon\Sysmon64.exe" -c
}

```


```
Get-Service Sysmon64

Get-WinEvent -FilterHashtable @{
    LogName = 'Microsoft-Windows-Sysmon/Operational'
    Id = 1,3,13
} -MaxEvents 10 -ErrorAction SilentlyContinue |
Select-Object TimeCreated, Id, ProviderName |
Format-Table -AutoSize

```
