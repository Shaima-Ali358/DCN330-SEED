بعطيك الأوامر دفعة واحدة، ومعها أوامر الـTroubleshooting، عشان إذا واجهتي مشكلة ما تحتاجين تنتظرين ردي بعد كل أمر.

**مهم:** أوامر الـTroubleshooting تنفذينها فقط إذا واجهتي المشكلة المذكورة، مو كلها بشكل عشوائي.

# **Part 1 — Environment Verification**

## **1. Windows 11 — PowerShell (Administrator)**

نفذي الأوامر التالية:

```
Get-ComputerInfo | Select-Object WindowsProductName, WindowsVersion, OsBuildNumber

hostname

whoami

ipconfig /all

Get-NetAdapter

Get-NetIPAddress -AddressFamily IPv4

Get-Service -Name "Sysmon*" -ErrorAction SilentlyContinue

Get-Service -Name AppIDSvc

Get-LocalUser
```

هذه الأوامر بتوضح لنا إصدار الويندوز، والـIP، والمستخدمين الموجودين، وحالة Sysmon وAppLocker.

📷 صوري النتائج للتحقق فقط، مو شرط نحطها بالتقرير.

## **2. Kali Linux — Terminal**

```
hostname
whoami
ip -br a
ip route
```

للتحقق من الأدوات الموجودة:

```
which msfconsole
which msfvenom
which python3
which git
which curl
which php
```

ولمعرفة المساحة المتوفرة:

```
df -h
```

## **3. Test Network Connectivity**

من Kali:

```
ping -c 4 WINDOWS_IP
```

من Windows PowerShell:

```
ping KALI_IP
```

استبدلي `WINDOWS_IP` و`KALI_IP` بالعناوين الفعلية للأجهزة على شبكة `SPR320-LAB`، وليس عناوين NAT.

### **Troubleshooting — Network**

إذا الاتصال ما اشتغل، نفذي على Windows:

```
Get-NetAdapter
Get-NetIPConfiguration
Get-NetConnectionProfile
Get-NetFirewallProfile
route print
arp -a
```

وعلى Kali:

```
ip link
ip -br a
ip route
ip neigh
```

عدم نجاح الـPing ما يعني بالضرورة أن الشبكة ما تشتغل، ممكن Windows Firewall يمنع ICMP.

# **Part 2 — Sysmon Installation and Configuration**

**جميع الأوامر التالية على Windows 11 باستخدام PowerShell as Administrator.**

## **1. Download Sysmon**

إذا Sysmon مو مثبت عندك، نفذي:

```
New-Item -ItemType Directory -Path C:\Tools\Sysmon -Force

Set-Location C:\Tools\Sysmon

Invoke-WebRequest -Uri "https://download.sysinternals.com/files/Sysmon.zip" -OutFile Sysmon.zip

Expand-Archive -Path Sysmon.zip -DestinationPath . -Force

Get-ChildItem
```

## **2. Download SwiftOnSecurity Configuration**

هذا ملف الـConfiguration المستخدم في شرح الدكتور.

```
Set-Location C:\Tools\Sysmon

Invoke-WebRequest -Uri "https://raw.githubusercontent.com/SwiftOnSecurity/sysmon-config/master/sysmonconfig-export.xml" -OutFile sysmonconfig.xml
```

للتحقق من الملف:

```
Get-Item .\sysmonconfig.xml

Get-Content .\sysmonconfig.xml -TotalCount 40
```

📸 **Required Screenshot #2**

صوري الملفات الموجودة داخل `C:\Tools\Sysmon` ومحتوى الـConfiguration.

**عنوان الصورة:**\
`Sysmon Files and SwiftOnSecurity Configuration`

## **3. Install Sysmon**

إذا Sysmon غير مثبت:

```
.\Sysmon64.exe -accepteula -i .\sysmonconfig.xml
```

أما إذا كان مثبتًا من اللاب السابق، استخدمي تحديث الـConfiguration بدل إعادة التثبيت:

```
.\Sysmon64.exe -c .\sysmonconfig.xml
```

لا تنفذين الأمرين إلا حسب حالة التثبيت.

📸 **Required Screenshot #3**

صوري نتيجة التثبيت أو تحديث الـConfiguration.

**عنوان الصورة:**\
`Sysmon Installation and Configuration`

## **4. Verify Sysmon**

```
Get-Service -Name "Sysmon*"

.\Sysmon64.exe -c
```

ثم:

```
Get-WinEvent -ListLog "*Sysmon*"
```

ولعرض آخر الأحداث:

```
Get-WinEvent -LogName "Microsoft-Windows-Sysmon/Operational" -MaxEvents 10 | Format-Table TimeCreated, Id, Message -Wrap
```

## **5. Open Event Viewer**

```
eventvwr.msc
```

روحي للمسار التالي:

&#x20;Applications and Services Logs

&#x20;Microsoft

&#x20;Windows

&#x20;Sysmon

&#x20;**Operational**

📸 **Required Screenshot #4**

صوري Sysmon Operational Log وفيه الأحداث المسجلة.

**عنوان الصورة:**\
`Sysmon Operational Event Logs`

## **Troubleshooting — Sysmon**

| المشكلة                         | الأمر المناسب                                            |
| ------------------------------- | -------------------------------------------------------- |
| التأكد من وجود الملفات          | `Get-ChildItem C:\Tools\Sysmon`                          |
| التأكد من الخدمة                | `Get-Service -Name "Sysmon*"`                            |
| التحقق من الإنترنت              | `Test-NetConnection download.sysinternals.com -Port 443` |
| التحقق من DNS                   | `Resolve-DnsName download.sysinternals.com`              |
| التحقق من صلاحيات Administrator | `whoami /groups`                                         |
| معرفة إعدادات Sysmon الحالية    | `.\Sysmon64.exe -c`                                      |
| التحقق من XML                   | `[xml](Get-Content .\sysmonconfig.xml -Raw)`             |

إذا الخدمة موجودة لكنها متوقفة:

```
Get-Service -Name "Sysmon*" | Start-Service
```

إذا ظهر خطأ في توافق الـConfiguration:

```
.\Sysmon64.exe -s
```

هذا يعرض الـSchema المدعوم.

# **Part 3 — AppLocker Preparation**

حالياً نتحقق من جاهزية AppLocker، لكن **لا تفعلين قواعد الحظر قبل توثيق اختبار التنفيذ الأول**، لأن الدكتور يطلب مقارنة النتائج قبل وبعد.

## **1. Verify AppLocker**

على Windows PowerShell:

```
Get-Command *AppLocker*

Get-Service AppIDSvc

Get-AppLockerPolicy -Local

Get-AppLockerPolicy -Effective
```

لفتح Local Security Policy:

```
secpol.msc
```

[How to restrict command line CMD from users that are not admins or to a specific list of users - Microsoft Q&A](https://images.openai.com/static-rsc-4/OjpsesxSTTcb32LoLcwD8PUtHN0q7GU1CE9DbRwlob5o6IG9BFjmYGIRFgXsmsEPuz7srlVThJ6q4gOUnhvkV3MkZW0pkzu9fBs95E_bsnzqb8IQ1QIo__G8qBgWJxgQHXgQ40Mvg51tiCg5TFu43vWKJkHnJ6-1xSNqyVmQcRM?purpose=inline)

روحي إلى:

Security Settings → Application Control Policies → AppLocker

تأكدي أن الخيارات موجودة، لكن لا تنشئين القواعد للحين.

## **2. AppLocker Troubleshooting Commands**

هذه الأوامر بنحتاجها لاحقًا عند تطبيق القواعد.

لتشغيل الخدمة:

```
Start-Service AppIDSvc
```

للتحقق منها:

```
Get-Service AppIDSvc
```

لتطبيق السياسات:

```
gpupdate /force
```

للتحقق من السياسات الفعلية:

```
Get-AppLockerPolicy -Effective -Xml
```

لعرض سجلات AppLocker:

```
Get-WinEvent -ListLog "*AppLocker*"
```

لعرض آخر أحداث Executable Rules:

```
Get-WinEvent -LogName "Microsoft-Windows-AppLocker/EXE and DLL" -MaxEvents 20 | Format-List TimeCreated, Id, Message
```

أهم Event IDs اللي بنبحث عنها:

| Event ID | Meaning                                    |
| -------- | ------------------------------------------ |
| 8002     | Application allowed                        |
| 8003     | Application would be blocked in audit mode |
| 8004     | Application blocked                        |

# **Part 4 — Additional Sysmon Tests**

هذه الأوامر بنستخدمها لاحقًا لمحاكاة أنشطة ومراقبتها باستخدام Sysmon.

## **Test 1 — Process Creation**

```
Start-Process notepad.exe
```

عرض Event ID 1:

```
Get-WinEvent -FilterHashtable @{LogName="Microsoft-Windows-Sysmon/Operational"; ID=1} -MaxEvents 10 | Format-List TimeCreated, Id, Message
```

## **Test 2 — File Creation**

```
New-Item -ItemType Directory -Path C:\Lab3 -Force

Set-Content -Path C:\Lab3\TestFile.txt -Value "SPR320 Sysmon File Creation Test"
```

عرض Event ID 11:

```
Get-WinEvent -FilterHashtable @{LogName="Microsoft-Windows-Sysmon/Operational"; ID=11} -MaxEvents 10 | Format-List TimeCreated, Id, Message
```

## **Test 3 — Registry Modification**

```
New-Item -Path "HKCU:\Software\SPR320Lab3" -Force

New-ItemProperty -Path "HKCU:\Software\SPR320Lab3" -Name "TestValue" -Value "Sysmon Registry Test" -PropertyType String -Force
```

عرض Registry Events:

```
Get-WinEvent -FilterHashtable @{LogName="Microsoft-Windows-Sysmon/Operational"; ID=12,13} -MaxEvents 10 | Format-List TimeCreated, Id, Message
```

## **Test 4 — Network Connection**

على Kali نشغل HTTP Server داخل مجلد تجريبي فارغ:

```
mkdir -p ~/lab3-http
cd ~/lab3-http
echo "SPR320 Network Test" > index.html
python3 -m http.server 8080 --bind KALI_IP
```

وعلى Windows:

```
curl.exe http://KALI_IP:8080
```

عرض Event ID 3:

```
Get-WinEvent -FilterHashtable @{LogName="Microsoft-Windows-Sysmon/Operational"; ID=3} -MaxEvents 10 | Format-List TimeCreated, Id, Message
```

**ملاحظة:** مو مضمون أن كل هذه الأنشطة تظهر مع SwiftOnSecurity Configuration، لأن الملف يستخدم Filters. إذا ما ظهر حدث معين، نراجع الـConfiguration ونعدله حسب الاختبار. هذه اختبارات محاكاة لأنشطة مرتبطة بالهجمات، وليست هجمات فعلية بحد ذاتها.

# **وش تنفذين الحين بالضبط؟**

**Current Progress**

1 / 9

Snapshot — تم حسب كلامك

Part 1 — Windows verification

Part 1 — Kali verification

Part 1 — Network connectivity

Part 2 — Download Sysmon

Part 2 — Download configuration

Part 2 — Install or update Sysmon

Part 2 — Verify Event Viewer

Part 3 — Verify AppLocker availability

نفذي Part 1 وPart 2 والتحقق الأولي من AppLocker فقط الآن. بعدها أرسلي النتائج دفعة واحدة.

**الوقت المتوقع: 15–25 دقيقة.**

بعدها نعطل NAT ونكمل الاختبارات على الشبكة الداخلية. ما أدرجت أوامر إنشاء الـBackdoor أو التقاط بيانات الدخول باستخدام Zphisher بسبب القيود اللي وضحتها لك سابقًا، وباقي إعداد قواعد AppLocker بنسويه بعد توثيق اختبار التنفيذ الأول حتى نحافظ على ترتيب اللاب المطلوب من الدكتور.
