Step 1 — Registry Parser
افتحي PowerShell كـ Administrator على DC01، وانسخي الأوامر:
```
$events = Import-Csv "C:\Tools\SysmonEvents.csv"

$events |
Where-Object { $_.Id -in @('12','13','14') } |
Select-Object -First 10 TimeCreated, Id, Message |
Format-List
```

هذا يبحث عن أحداث Registry داخل ملف CSV.

إذا ما ظهرت نتائج، نفذي البديل المباشر من Sysmon:
```
Get-WinEvent -FilterHashtable @{
    LogName = 'Microsoft-Windows-Sysmon/Operational'
    Id = 12,13,14
} -MaxEvents 10 |
Format-List TimeCreated, Id, Message
```

📸 Screenshot Required — Registry Events

خذي لقطة شاشة إذا ظهرت أحداث Registry، ويفضل Event ID 13.

عنوان الصورة:```
Parsing Sysmon Registry Events
Step 2 — Scheduled Task Parser
نفذي:
```
Get-ScheduledTask -TaskName "SysmonLabTask" -ErrorAction SilentlyContinue |
Format-List TaskName, State, TaskPath
```

إذا ما ظهرت نتيجة لأنك حذفتي المهمة سابقاً، أنشئي مهمة تجريبية:
```
$action = New-ScheduledTaskAction -Execute "notepad.exe"

$trigger = New-ScheduledTaskTrigger -AtLogOn

Register-ScheduledTask -TaskName "SysmonLabTask" -Action $action -Trigger $trigger -Description "SPR320 Sysmon Lab Test"
```


ثم تحققي:
```
Get-ScheduledTask -TaskName "SysmonLabTask" |
Format-List TaskName, State, TaskPath
```


📸 Screenshot Required — Scheduled Task

خذي لقطة شاشة للنتيجة التي تظهر SysmonLabTask وحالتها Ready.

عنوان الصورة:

Verifying Scheduled Task Persistence

بعد التصوير، احذفي المهمة التجريبية:
```
Unregister-ScheduledTask -TaskName "SysmonLabTask" -Confirm:$false
```
