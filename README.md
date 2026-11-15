```
Start-Process notepad.exe



index=main sourcetype="XmlWinEventLog:Microsoft-Windows-Sysmon/Operational" host=CLIENT01 EventID=1
| search Image="*notepad.exe"
| table _time host Image CommandLine ParentImage User
| sort - _time





index=main sourcetype="XmlWinEventLog:Microsoft-Windows-Sysmon/Operational" host=CLIENT01
| timechart span=1h count
```
https://github.com/SwiftOnSecurity/sysmon-config/blob/master/sysmonconfig-export.xml?utm_source=chatgpt.com

index=main sourcetype="XmlWinEventLog:Microsoft-Windows-Sysmon/Operational" host=CLIENT01
| stats count by EventID
| sort - count

index=main sourcetype="XmlWinEventLog:Microsoft-Windows-Sysmon/Operational" host=CLIENT01 EventID=3
| table _time Image SourceIp DestinationIp DestinationPort
| head 20


index=main sourcetype="XmlWinEventLog:Microsoft-Windows-Sysmon/Operational" host=CLIENT01 EventID=13
| table _time Image TargetObject Details User
| head 20



