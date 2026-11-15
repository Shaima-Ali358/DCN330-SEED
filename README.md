```


$bin = "C:\Program Files\Splunk\bin\splunk.exe"
$etc = "C:\Program Files\Splunk\etc"

# Stop Splunk
& $bin stop

# Back up the existing credentials file
Copy-Item "$etc\passwd" "$etc\passwd.backup" -Force

# Rename the old credentials file
Rename-Item "$etc\passwd" "passwd.old"

# Create a temporary admin password
@"
[user_info]
USERNAME = admin
PASSWORD = SPR320Lab2026!
"@ | Set-Content "$etc\system\local\user-seed.conf" -Encoding ASCII

# Start Splunk
& $bin start --accept-license --answer-yes --no-prompt
```
https://github.com/SwiftOnSecurity/sysmon-config/blob/master/sysmonconfig-export.xml?utm_source=chatgpt.com


index=main sourcetype="XmlWinEventLog:Microsoft-Windows-Sysmon/Operational"
| stats count by host


