نفذي على DC01:
```Unregister-ScheduledTask -TaskName "SysmonLabTask" -Confirm:$false```


بعدها نصدّر أحداث Sysmon إلى CSV:
```
New-Item -Path "C:\Tools" -ItemType Directory -Force

Get-WinEvent -FilterHashtable @{
    LogName = 'Microsoft-Windows-Sysmon/Operational'
    StartTime = (Get-Date).AddDays(-2)
} | Select-Object TimeCreated, Id, ProviderName, Message |
Export-Csv -Path "C:\Tools\SysmonEvents.csv" -NoTypeInformation -Encoding UTF8
```

نتأكد من الملف:
```
Get-Item "C:\Tools\SysmonEvents.csv"

Import-Csv "C:\Tools\SysmonEvents.csv" |
Select-Object -First 5 TimeCreated, Id, ProviderName |
Format-Table -AutoSize
```
