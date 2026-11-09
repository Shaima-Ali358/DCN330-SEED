```
cd "C:\Users\Administrator\Downloads\Sysmon"

Copy-Item ".\sysmonconfig-export.xml" "C:\Tools\SysmonConfig-Backup.xml"

Test-Path "C:\Tools\SysmonConfig-Backup.xml"

```
True


Step 2 — Download Olaf Hartong Configuration

```
https://github.com/olafhartong/sysmon-modular
```

Step 3 — Apply Configuration
```Test-Path ".\sysmonconfig.xml"```


```.\Sysmon64.exe -c ".\sysmonconfig.xml"```


```Get-Service Sysmon64```


Step 4 — Verify Active Configuration
```.\Sysmon64.exe -c```



```
cd "C:\Users\Administrator\Downloads\Sysmon"

Invoke-WebRequest -Uri "https://raw.githubusercontent.com/olafhartong/sysmon-modular/master/sysmonconfig.xml" -OutFile ".\sysmonconfig.xml"
```

```
Get-ChildItem "C:\Users\Administrator\Downloads\sysmon-modular-master" -Recurse -Filter "*.xml" |
Select-Object -First 15 FullName
```

```
Get-ChildItem "C:\Users\Administrator\Downloads\sysmon-modular-master" -Recurse -Filter "Merge-SysmonXml.ps1" |
Select-Object FullName
```

```
cd "C:\Users\Administrator\Downloads\Sysmon"

Test-Path ".\sysmonconfig-export.xml"
```


```
.\Sysmon64.exe -c ".\sysmonconfig-export.xml"


.\Sysmon64.exe -c


Get-WinEvent -FilterHashtable @{
    LogName = 'Microsoft-Windows-Sysmon/Operational'
    Id = 1,3,13
} -MaxEvents 10 |
Select-Object TimeCreated, Id, ProviderName |
Format-Table -AutoSize
```
