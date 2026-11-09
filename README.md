```

Get-Service Sysmon64 -ErrorAction SilentlyContinue


Get-AppLockerPolicy -Effective |
Select-Object -ExpandProperty RuleCollections |
Select-Object RuleCollectionType, EnforcementMode

https://learn.microsoft.com/en-us/sysinternals/downloads/sysmon


cd "C:\Users\Administrator\Downloads\Sysmon"

.\Sysmon64.exe -accepteula -i

.\Sysmon64.exe -c ".\sysmonconfig.xml"

Get-Service Sysmon64


Get-WinEvent -LogName "Microsoft-Windows-Sysmon/Operational" -MaxEvents 10 |
Select-Object TimeCreated, Id, ProviderName |
Format-Table -AutoSize

```
