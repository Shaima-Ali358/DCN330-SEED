Part 1 — إصلاح AppLocker على CLIENT01
افتحي PowerShell بصلاحية Administrator.
Step 1 — احفظي نسخة من إعدادات AppLocker
نفذي:
New-Item -Path "C:\Tools" -ItemType Directory -Force

Get-AppLockerPolicy -Local -Xml |
Out-File "C:\Tools\AppLocker-Backup.xml" -Encoding UTF8


هذا يحفظ نسخة من القواعد المحلية قبل التعديل.
Step 2 — افحصي القواعد الثلاث
Get-AppLockerPolicy -Effective |
Select-Object -ExpandProperty RuleCollections |
Where-Object { $_.RuleCollectionType -eq 'Exe' } |
Select-Object -ExpandProperty Rules |
Format-List Name, Action, UserOrGroupSid


إذا ظهرت قواعد Allow فقط، لا تحذفينها. هذه غالبًا القواعد الافتراضية التي تسمح بتشغيل البرامج من Windows وProgram Files.
Step 3 — افحصي سجل الحظر
نفذي:
Get-WinEvent -FilterHashtable @{
    LogName = 'Microsoft-Windows-AppLocker/EXE and DLL'
    Id = 8003,8004
} -MaxEvents 15 -ErrorAction SilentlyContinue |
Format-List TimeCreated, Id, Message


- Event 8004 يعني أن AppLocker منع تشغيل برنامج.
- Event 8003 يعني أن البرنامج كان سيُمنع لو كانت القواعد في وضع Enforce، لكنه سُمح له بالعمل في وضع Audit.
إذا ما ظهرت نتائج، هذا لا يثبت أن AppLocker ليس السبب؛ ممكن يكون الحظر من سياسة ثانية.
Part 2 — تحويل AppLocker إلى Audit Only
بما إن الدكتور طالب AppLocker في اللاب، الأفضل نخلي القواعد موجودة، لكن نوقف منع البرامج مؤقتًا.

افتحي نافذة Local Security Policy باستخدام:
secpol.msc


ثم روحي إلى:

Application Control Policies → AppLocker → Properties

اضبطي التالي:
Rule collection	Setting
Executable rules	Audit only
Windows Installer rules	Audit only
Script rules	Audit only
Packaged app Rules	Audit only

اضغطي Apply ثم OK.

بعدها نفذي:
Get-AppLockerPolicy -Local |
Select-Object -ExpandProperty RuleCollections |
Select-Object RuleCollectionType, EnforcementMode


ملاحظة: لو بقيت بعض القواعد NotConfigured، هذا لا يثبت أنها معطلة. نحتاج نتأكد من الـEffective Policy ومن سجل الأحداث.
Part 3 — إصلاح Group Policy
على CLIENT01 نفذي:
Test-NetConnection 192.168.56.10 -Port 445


ثم:
Test-NetConnection 192.168.56.10 -Port 389


ثم:
Test-NetConnection 192.168.56.10 -Port 88


المفروض يظهر:
TcpTestSucceeded : True


إذا فشل أحدها، راجعي الاتصال أو الخدمات على DC01 قبل تحديث السياسات.

بعدها افحصي DNS:
Get-DnsClientServerAddress -AddressFamily IPv4


المفروض DNS الخاص بـCLIENT01 يشير إلى:
192.168.56.10


إذا كان مختلفًا، وكان اسم كرت الشبكة Ethernet، نفذي:
Set-DnsClientServerAddress -InterfaceAlias "Ethernet" -ServerAddresses 192.168.56.10


ثم:
ipconfig /flushdns


وجربي:
nslookup spr320.test


بعد التأكد من الاتصال وDNS:
w32tm /resync /rediscover
gpupdate /force


إذا استمر فشل Computer Policy، نفذي:
gpresult /h "C:\Tools\GPReport.html"


وافتحي التقرير لمعرفة السياسات المطبقة والأخطاء.
Part 4 — اختبار البرامج
جربي هذه الأوامر على CLIENT01:
Start-Process explorer.exe


Start-Process notepad.exe


Start-Process ms-settings:


وجربي Start Menu أيضًا.

إذا اشتغلت البرامج بدون رسالة الحظر، نقدر نكمل اللاب.

إذا استمر الحظر، افحصي سجل AppLocker مرة ثانية، وكذلك Software Restriction Policies، بدل حذف القواعد أو تعطيل خدمات الحماية عشوائيًا.
Part 5 — إعادة تشغيل CLIENT01
بعد ما تتأكدين أن الإعدادات محفوظة:
Restart-Computer


بعد إعادة التشغيل، افحصي:
Get-AppLockerPolicy -Effective |
Select-Object -ExpandProperty RuleCollections |
Select-Object RuleCollectionType, EnforcementMode


وجربي فتح البرامج مرة ثانية.

مهم بالنسبة للـAssignment
