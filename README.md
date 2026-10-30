تمام، بنبدأ على DC01 (Windows Server 2022) ونمشي بترتيب تعليمات الدكتور في SPR320 — Lab 2. ما راح ننتقل إلى Part 2 إلا بعد ما نخلص Part 1 ونتأكد إن Sysmon شغال.
Part 1 — Event Viewer and Sysmon
الهدف: نثبت Sysmon على DC01، نضبط الـConfiguration، وبعدها نتأكد إنه يسجل الأحداث في Event Viewer.
Step 1 — Check Windows Event Viewer
على DC01:
1. سجلي دخول بحساب Administrator.
2. اضغطي Win + R.
3. اكتبي:
```
eventvwr.msc
```

4. افتحي Windows Logs > Security.
5. تأكدي إن الأحداث موجودة.
Screenshot 1 — Windows Security Event Logs

صوري نافذة Event Viewer مع ظهور الأحداث.
Step 2 — Check Sysmon
افتحي PowerShell as Administrator ونفذي الأوامر التالية:
```
hostname
```

المفروض يظهر DC01.
```
Get-Service *sysmon*
```

إذا ظهر Running، فـSysmon مثبت. إذا ما ظهرت خدمة، نكمل التثبيت.

وللتأكد من نسخة Windows:
```
Get-ComputerInfo | Select-Object WindowsProductName, OsVersion
```

Step 3 — Download Sysmon
في PowerShell كـAdministrator:
```
New-Item -Path C:\Tools -ItemType Directory -Force
```

بعدها:
```
Invoke-WebRequest -Uri "https://download.sysinternals.com/files/Sysmon.zip" -OutFile "C:\Tools\Sysmon.zip"
```

ثم فكّي الضغط:
```
Expand-Archive -Path "C:\Tools\Sysmon.zip" -DestinationPath "C:\Tools\Sysmon" -Force
```

تأكدي من الملفات:
```
Get-ChildItem C:\Tools\Sysmon
```

لازم تشوفين ملف Sysmon64.exe ضمن الملفات.
Step 4 — Download SwiftOnSecurity Configuration
هذا ملف Configuration من المصادر اللي ذكرها الدكتور. بنستخدمه كبداية على DC01، وبعدها نتعرف على Olaf Hartong ونقارن الإعدادات.
```
Invoke-WebRequest -Uri "https://raw.githubusercontent.com/SwiftOnSecurity/sysmon-config/master/sysmonconfig-export.xml" -OutFile "C:\Tools\Sysmon\sysmonconfig-export.xml"
```

تأكدي:
```
Test-Path "C:\Tools\Sysmon\sysmonconfig-export.xml"
```

المفروض يظهر:
True


Step 5 — Install Sysmon
نفذي هذا فقط إذا Step 2 أكد أن Sysmon غير مثبت.
```
cd C:\Tools\Sysmon
```

ثم:
```
.\Sysmon64.exe -accepteula -i .\sysmonconfig-export.xml
```

إذا نجح، المفروض تشوفين رسالة تثبيت ناجح، مثل:

Sysmon64 installed.
SysmonDrv installed.
Sysmon64 started.


قد تختلف الرسائل حسب إصدار Sysmon.
Screenshot 2 — Sysmon Installation

صوري PowerShell بعد ظهور نتيجة التثبيت الناجح.

إذا ظهر خطأ Configuration أو Schema، وقفي هنا وأرسلي الخطأ قبل ما نغير أي إعدادات.
Step 6 — Verify Sysmon
نفذي:
```
Get-Service *sysmon*
```

وبعدها:
```
.\Sysmon64.exe -c
```

الأول يفحص الخدمة، والثاني يعرض معلومات الـConfiguration الحالي.

ثم افتحي Event Viewer:
```
eventvwr.msc
```

روحي إلى:

Applications and Services Logs > Microsoft > Windows > Sysmon > Operational
Screenshot 3 — Sysmon Operational Logs

صوري الأحداث الظاهرة، ويفضل تختارين Event ID 1 إذا كان موجوداً.
Step 7 — Test Sysmon
افتحي PowerShell ونفذي:
Start-Process notepad.exe


بعدها:
```
Get-WinEvent -FilterHashtable @{
    LogName = 'Microsoft-Windows-Sysmon/Operational'
    Id = 1
} -MaxEvents 10 | Format-List TimeCreated, Id, Message
```

دوري في النتائج على notepad.exe.

هذا يثبت أن Sysmon سجل تشغيل برنامج جديد.
Screenshot 4 — Sysmon Event ID 1 (Process Creation)

صوري الحدث بحيث يظهر اسم notepad.exe وEvent ID 1.

Part 1 — متابعة الإنجاز
DC01 Progress

0 / 7



[ ] Event Viewer Security Logs
[ ] Check Sysmon status
[ ] Download Sysmon
[ ] Download Configuration
[ ] Install Sysmon
[ ] Verify Operational Logs
[ ] Test Event ID 1

الحين ابدئي من Step 1 إلى Step 7 على DC01. إذا Sysmon موجود مسبقاً، لا تعيدين تثبيته؛ أرسلي لي نتيجته عشان نقرر هل نحتاج تحديث الـConfiguration فقط.

بعد ما ترسلين الصور والنتائج، بنتأكد من Part 1 على DC01، ونكمل بقية أجهزة Windows، خصوصاً Metasploitable3 اللي يحتاج إصدار Sysmon أقدم. وبعدها نبدأ Part 2 واختبارات Event IDs المطلوبة من الدكتور.
