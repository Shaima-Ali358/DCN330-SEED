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


```
cd "$env:USERPROFILE\Downloads\Sysmon"

Get-ChildItem

.\Sysmon64.exe -accepteula -i

Get-Service Sysmon64
```
/////////////////////////////////////



```
Get-Service -Name "*sysmon*" -ErrorAction SilentlyContinue

Get-CimInstance Win32_SystemDriver |
Where-Object { $_.Name -like "*Sysmon*" } |
Select-Object Name, State, StartMode

Get-ItemProperty "HKLM:\SYSTEM\CurrentControlSet\Services\Sysmon*" -ErrorAction SilentlyContinue |
Select-Object PSChildName, ImagePath

Get-WinEvent -ListLog "Microsoft-Windows-Sysmon/Operational" -ErrorAction SilentlyContinue |
Select-Object LogName, IsEnabled, RecordCount

```
