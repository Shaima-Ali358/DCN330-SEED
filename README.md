```
index=main sourcetype="XmlWinEventLog:Microsoft-Windows-Sysmon/Operational" host=CLIENT01
| rex field=_raw "<EventID>(?<EventID>[0-9]+)</EventID>"
| stats count by EventID
| sort - count

index=main sourcetype="XmlWinEventLog:Microsoft-Windows-Sysmon/Operational" host=CLIENT01
| rex field=_raw "<EventID>(?<EventID>[0-9]+)</EventID>"
| search EventID=1
| rex field=_raw "<Data Name='Image'>(?<Image>[^<]+)</Data>"
| table _time Image
| head 20

index=main sourcetype="XmlWinEventLog:Microsoft-Windows-Sysmon/Operational" host=CLIENT01
| rex field=_raw "<EventID>(?<EventID>[0-9]+)</EventID>"
| search EventID=3
| rex field=_raw "<Data Name='DestinationIp'>(?<DestinationIp>[^<]+)</Data>"
| rex field=_raw "<Data Name='DestinationPort'>(?<DestinationPort>[^<]+)</Data>"
| table _time DestinationIp DestinationPort
| head 20

index=main sourcetype="XmlWinEventLog:Microsoft-Windows-Sysmon/Operational" host=CLIENT01
| rex field=_raw "<EventID>(?<EventID>[0-9]+)</EventID>"
| search EventID=13
| rex field=_raw "<Data Name='TargetObject'>(?<TargetObject>[^<]+)</Data>"
| table _time TargetObject
| head 20



index=main sourcetype="XmlWinEventLog:Microsoft-Windows-Sysmon/Operational" host=CLIENT01
| rex field=_raw "<EventID>(?<EventID>[0-9]+)</EventID>"
| search EventID=1
| rex field=_raw "<Data Name='Image'>(?<Image>[^<]+)</Data>"
| search Image="*notepad.exe"
| table _time Image
| sort - _time
```
https://github.com/SwiftOnSecurity/sysmon-config/blob/master/sysmonconfig-export.xml?utm_source=chatgpt.com




