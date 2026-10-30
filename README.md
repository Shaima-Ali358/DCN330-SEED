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


انسخي الأوامر:
```
cd "C:\Users\Administrator\Downloads\Sysmon"

Invoke-WebRequest -Uri "https://raw.githubusercontent.com/SwiftOnSecurity/sysmon-config/master/sysmonconfig-export.xml" -OutFile ".\sysmonconfig-export.xml"

Test-Path ".\sysmonconfig-export.xml"
```

المفروض آخر أمر يعطيك:
True


Step 2 — Apply Configuration
```
.\Sysmon64.exe -c .\sysmonconfig-export.xml
```

إذا ظهر Configuration updated أو رسالة نجاح مشابهة، كملي.
Screenshot 1 — Updating Sysmon Configuration

صوري نتيجة الأمر بعد نجاح التحديث.

إذا ظهر Error أو Configuration validation failed، وقفي وأرسلي لي الخطأ.
Step 3 — Verify Sysmon Configuration
```
.\Sysmon64.exe -c
```

ثم:
```
Get-Service Sysmon64
```

المفروض تشوفين Running للخدمة، ومعلومات الـConfiguration الحالي.
Screenshot 2 — Verifying Sysmon Configuration

صوري نتيجة الأمرين.
Step 4 — Open Windows Event Viewer
```
eventvwr.msc
```

من القائمة اليسار افتحي:

Applications and Services Logs > Microsoft > Windows > Sysmon > Operational

المفروض تشوفين قائمة Events.
Screenshot 3 — Sysmon Operational Logs

صوري Event Viewer مع ظهور الأحداث.
Step 5 — Generate Process Creation Event
الحين بنولد حدث بسيط عشان نتأكد إن Sysmon يسجل تشغيل البرامج.

في PowerShell:
```
Start-Process notepad.exe
```

بعدها:
```
Get-WinEvent -FilterHashtable @{
    LogName = 'Microsoft-Windows-Sysmon/Operational'
    Id = 1
} -MaxEvents 20 | Format-List TimeCreated, Id, Message

```
ابحثي عن notepad.exe في النتائج.

Event ID 1 = Process Creation
Screenshot 4 — Process Creation (Event ID 1)

صوري الحدث الذي يظهر فيه notepad.exe.
Step 6 — Generate File Creation Event
بننشئ ملف داخل مجلد مؤقت على DC01:
```
New-Item -Path "C:\SysmonTest" -ItemType Directory -Force
```

ثم:
```
Set-Content -Path "C:\SysmonTest\test.txt" -Value "Sysmon Lab 2 Test"
```

افحصي Event ID 11:
```
Get-WinEvent -FilterHashtable @{
    LogName = 'Microsoft-Windows-Sysmon/Operational'
    Id = 11
} -MaxEvents 30 | Where-Object {
    $_.Message -like '*SysmonTest*'
} | Format-List TimeCreated, Id, Message
```

Event ID 11 = File Create

إذا ما ظهر الحدث، مو بالضرورة فيه مشكلة؛ ممكن ملف الـConfiguration يستبعد هذا النوع من الملفات أو المسارات.
Screenshot 5 — File Creation (Event ID 11)

صوري الحدث إذا ظهر.
Step 7 — Generate Registry Event
بنسوي Registry Key تجريبي، بدون تعديل إعدادات النظام الحساسة.
```
New-Item -Path "HKCU:\Software\SysmonLab2" -Force
```

ثم:
```
New-ItemProperty -Path "HKCU:\Software\SysmonLab2" -Name "LabTest" -Value "SPR320" -PropertyType String -Force
```

افحصي Event IDs الخاصة بالـRegistry:
```
Get-WinEvent -FilterHashtable @{
    LogName = 'Microsoft-Windows-Sysmon/Operational'
    Id = 12,13,14
} -MaxEvents 50 | Where-Object {
    $_.Message -like '*SysmonLab2*'
} | Format-List TimeCreated, Id, Message
```

```
Event IDs 12–14 = Registry Changes
```
إذا ما ظهر شيء، ممكن قواعد SwiftOnSecurity ما تسجل هذا المسار التجريبي.

Screenshot 6 — Registry Activity

صوري الحدث إذا ظهر.
Step 8 — Check DNS and Network Events
نفذي:
```
Resolve-DnsName example.com
```

ثم افحصي Event ID 22:
```
Get-WinEvent -FilterHashtable @{
    LogName = 'Microsoft-Windows-Sysmon/Operational'
    Id = 22
} -MaxEvents 20 | Format-List TimeCreated, Id, Message
```

Event ID 22 = DNS Query

بعدها افحصي Network Connections:
```
Get-WinEvent -FilterHashtable @{
    LogName = 'Microsoft-Windows-Sysmon/Operational'
    Id = 3
} -MaxEvents 20 | Format-List TimeCreated, Id, Message
```

Event ID 3 = Network Connection

قد لا يظهر Event ID 3 إذا كان معطلاً أو مستبعداً في الـConfiguration. كذلك نجاح DNS Query يعتمد على اتصال DNS المتاح في بيئة المختبر.
Screenshot 7 — DNS and Network Events

صوري النتائج إذا ظهرت.
Step 9 — Final Verification
آخر شيء نفذي

1) أهم تعديل: إصدار الـSchema (Step 2)

ملف SwiftOnSecurity قديم نسبياً (Schema 4.50 تقريباً). مع Sysmon 15.22 غالباً بيشتغل، لكن إذا رفضه أو ظهرت تحذيرات، استخدمي Olaf Hartong. وهو مطلوب منك أصلاً في اللاب (“Olaf Hartong configuration”). ملاحظة مهمة: الـREADME يقول إن الإصدارات المبنية جاهزة لـSysmon 15.21 وما دون، وإصدارك 15.22، فاختاري الملف 15.21 (هو الافتراضي sysmonconfig.xml). وبما أن الـSchema مستقل عن إصدار الـbinary، المفروض يشتغل.

الأفضل تطبقين الاثنين وتقارنين بينهم في التقرير، لأن الدكتور قال “You should be familiar with both”.

الرابط من صفحة الـReleases (تحتاجين تتأكدين من الاسم الفعلي للملف هناك):

powershell
# افتحي https://github.com/olafhartong/sysmon-modular/releases
# نزلي sysmonconfig.xml (الـBalanced، النسخة 15.21)

ثم:

powershell
```.\Sysmon64.exe -c .\sysmonconfig.xml```
2) تحقق قبل التطبيق (Step 1)

أضيفي هذا بعد Test-Path للتأكد أن الملف XML سليم وما هو صفحة خطأ HTML:

powershell
```(Get-Item .\sysmonconfig-export.xml).Length```
```Get-Content .\sysmonconfig-export.xml -TotalCount 5```

المفروض يبدأ بـ <Sysmon schemaversion=.

3) Event ID 3 غالباً ما بيظهر مع SwiftOnSecurity

الـConfiguration حقه يسجل الشبكة بشكل انتقائي (بورتات وبرامج معينة). فاختبار Resolve-DnsName ما بيولد Event 3 بالضرورة. الأفضل تولدينه بشكل مضمون:

powershell
```Test-NetConnection -ComputerName 192.168.56.11 -Port 445```

أو من DC01 افتحي اتصال إلى CLIENT01. وإذا ما ظهر، هذا نفسه يصير ملاحظة في التقرير (سببه الـConfiguration).

4) ناقص: اختبار John Strand (مطلوب في Part 2)

الاختبار الأساسي بالمهمة هو لاب Sysmon من IntroLabs (Part 2). خطواتك الحالية تغطي اختبارات Google AI بس. وللعلم هذا اللاب يحتاج Metasploit/Kali، فنفذيه لاحقاً بعد ما تخلصين الإعداد على الأجهزة الثلاثة.

5) ناقص: اختبارات الـPersistence

مطلوبة صراحة في اللاب، وما هي في خطواتك:

powershell
# Run Key (Event 13)
```New-ItemProperty -Path "HKCU:\Software\Microsoft\Windows\CurrentVersion\Run" -Name "FakeMalwarePersistence" -Value "C:\Windows\System32\calc.exe" -PropertyType String```

# Scheduled Task (Event 1 + 11)
```Register-ScheduledTask -TaskName "Suspicious_Update_Task" -Trigger (New-ScheduledTaskTrigger -AtLogOn) -Action (New-ScheduledTaskAction -Execute "notepad.exe") -Description "Simulated Persistence"```

وبعد ما تصورين، نظفي:
```
powershell
Remove-ItemProperty -Path "HKCU:\Software\Microsoft\Windows\CurrentVersion\Run" -Name "FakeMalwarePersistence"
Unregister-ScheduledTask -TaskName "Suspicious_Update_Task" -Confirm:$false
```
6) ناقص: Event 5 و Event 8 و Event 22 (ملف الدكتور)
Event 5 (Process Terminate): Stop-Process -Name notepad. لكن SwiftOnSecurity غالباً ما يسجله، والأرجح ما يظهر.
Event 8: ملف اللاب نفسه يقول يحتاج أدوات خاصة، فاعتبريه اختياري.
Event 22: ملاحظة الدكتور أن الاستعلام ما يحتاج ينجح. جربي``` nslookup super-suspicious-cnc-server.com بالإضافة لـexample.com.```
7) ناقص: تصدير CSV وفحص Event Viewer العادي

اللاب يتطلب سكربت التصدير إلى CSV وتجربة الـRegistry/Task parsers. أيضاً “Windows Event Viewer showing Security Logs” بـScreenshot 1 في خطتك، لكن ما كتبتي خطوة لها في الرد. أضيفيها (Windows Logs > Security).

8) تنبيه عن Sysmon + DC01

على الـDomain Controller غالباً بيطلع ضجيج كثير. بعد التطبيق راقبي حجم الـLog (Properties على Operational) وزيدي الحجم الأقصى لو احتجتي.

ملخص
الخطوة	الحالة
تحميل وتطبيق Configuration	صحيحة + أضيفي فحص XML
Event 1, 11, 12-14, 22	صحيحة
Event 3	تحتاج أمر مضمون
Olaf Hartong	ناقصة (مطلوبة)
Persistence	ناقصة (مطلوبة)
CSV export	ناقصة (مطلوبة)
John Strand lab	ناقصة (لاحقاً)

نفذي Step 1 و 2 وأرسلي لي النتيجة (أو الخطأ) لو ظهر شي.:
