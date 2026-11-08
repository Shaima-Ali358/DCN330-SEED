افتحي PowerShell كـ Administrator على DC01 ونفذي:
```
cd "C:\Users\Administrator\Downloads\Sysmon"

Copy-Item ".\sysmonconfig-export.xml" "C:\Tools\SysmonConfig-Backup.xml"

Test-Path "C:\Tools\SysmonConfig-Backup.xml"

```
المفروض النتيجة:
True


Step 2 — Download Olaf Hartong Configuration
بما أن DC01 عندك على Internal Network، افتحي الرابط من جهازك الأساسي أو من VM فيها إنترنت:

```
https://github.com/olafhartong/sysmon-modular
```

حمّلي ملف sysmonconfig.xml الجاهز من المشروع، ثم انقليه إلى مجلد Sysmon داخل DC01.
Step 3 — Apply Configuration
بعد ما تتأكدين إن الملف موجود في نفس المجلد، نفذي:
```Test-Path ".\sysmonconfig.xml"```


إذا ظهرت True، نفذي:
```.\Sysmon64.exe -c ".\sysmonconfig.xml"```


ثم تحققي:
```Get-Service Sysmon64```


المفروض تظهر Running.
Step 4 — Verify Active Configuration
```.\Sysmon64.exe -c```


تأكدي أن Config file يشير إلى sysmonconfig.xml الجديد.
