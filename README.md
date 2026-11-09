```

Get-CimInstance Win32_Service |
Where-Object {
    $_.Name -match 'Sysmon' -or
    $_.PathName -match 'Sysmon'
} |
Select-Object Name, State, PathName


Get-WinEvent -LogName "Microsoft-Windows-Sysmon/Operational" -MaxEvents 10 |
Select-Object TimeCreated, Id, ProviderName |
Format-Table -AutoSize


cd "$env:USERPROFILE\Downloads\Sysmon"

.\Sysmon64.exe -c
```
