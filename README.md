```
New-ItemProperty -Path "HKCU:\Software\Microsoft\Windows\CurrentVersion\Run" -Name "SysmonLabTest" -Value "C:\Windows\System32\notepad.exe" -PropertyType String -Force
```

```
Get-WinEvent -FilterHashtable @{
    LogName = 'Microsoft-Windows-Sysmon/Operational'
    Id = 13
} -MaxEvents 100 |
Where-Object { $_.Message -match 'SysmonLabTest' } |
Format-List TimeCreated, Id, Message
```

```
$action = New-ScheduledTaskAction -Execute "notepad.exe"
$trigger = New-ScheduledTaskTrigger -AtLogOn

Register-ScheduledTask -TaskName "SysmonLabTask" -Action $action -Trigger $trigger -Description "SPR320 Sysmon Persistence Test"
```

```
Get-ScheduledTask -TaskName "SysmonLabTask" |
Format-List TaskName, State, TaskPath

```

```
Remove-ItemProperty -Path "HKCU:\Software\Microsoft\Windows\CurrentVersion\Run" -Name "SysmonLabTest"

Unregister-ScheduledTask -TaskName "SysmonLabTask" -Confirm:$false
```
