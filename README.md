```
index=main sourcetype="XmlWinEventLog:Microsoft-Windows-Sysmon/Operational" host=CLIENT01
| head 5
| table _time host EventID EventCode Image _raw


```
https://github.com/SwiftOnSecurity/sysmon-config/blob/master/sysmonconfig-export.xml?utm_source=chatgpt.com




