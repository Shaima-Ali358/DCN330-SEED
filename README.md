تمام، شفت الصور كلها، **الاتصال بين Kali وWindows 11 ناجح 100%**. ما نحتاج نضيع وقت أكثر على الـNetwork Troubleshooting.

## **Part 1 — Network Verification**

Windows 11 IP

**192.168.56.11**

Kali Linux IP

**192.168.56.15**

Ping

**4/4 Successful**

Packet Loss

**0%**

ARP Resolution

**Successful**

لاحظت أيضًا أن Windows عنده NAT بعنوان `10.0.3.15`، وهذا مناسب مؤقتًا لتحميل الأدوات. لكن الـPublic Firewall ظاهر أنه متوقف، فالأفضل نرجع نشغله ونفصل NAT بعد التحميل وقبل الاختبارات.

### **وش الصور اللي نحتفظ فيها للتقرير؟**

**الصورة الأولى — احتفظي فيها**

فيها الـPing الناجح وعنوان Kali والـNetwork Route.

عنوانها: `Network Connectivity Between Kali and Windows 11`

**الصورة الثانية — اختيارية**

توضح ARP Table وعناوين الأجهزة.

**الصور الثالثة والرابعة والخامسة**

ما نحتاج نحطها في التقرير حاليًا، كانت للتحقق وحل المشاكل.

# **Part 2 — Sysmon Installation and Configuration**

الحين نبدأ Sysmon مباشرة. كل الأوامر التالية على **Windows 11 — PowerShell as Administrator**.

## **Step 1 — Check Existing Installation**

```
Get-Service -Name "Sysmon*" -ErrorAction SilentlyContinue
Get-Service AppIDSvc
```

إذا ما ظهر Sysmon، يعني غالبًا أنه غير مثبت كخدمة.

## **Step 2 — Download Sysmon and Configuration**

نفذي هذه الأوامر دفعة واحدة:

```
New-Item -ItemType Directory -Path C:\Tools\Sysmon -Force

Set-Location C:\Tools\Sysmon

Invoke-WebRequest -Uri "https://download.sysinternals.com/files/Sysmon.zip" -OutFile Sysmon.zip

Expand-Archive -Path Sysmon.zip -DestinationPath . -Force

Invoke-WebRequest -Uri "https://raw.githubusercontent.com/SwiftOnSecurity/sysmon-config/master/sysmonconfig-export.xml" -OutFile sysmonconfig.xml

Get-ChildItem
```

بعدها:

```
Get-Content .\sysmonconfig.xml -TotalCount 40
```

📸 **Required Screenshot:** صوري الملفات الموجودة ومحتوى ملف الـConfiguration.

**عنوان الصورة:**\
`Sysmon and SwiftOnSecurity Configuration Files`

## **Step 3 — Install Sysmon**

إذا Sysmon غير مثبت:

```
.\Sysmon64.exe -accepteula -i .\sysmonconfig.xml
```

أما إذا كان مثبتًا مسبقًا:

```
.\Sysmon64.exe -c .\sysmonconfig.xml
```

📸 **Required Screenshot:** صوري نتيجة التثبيت أو تحديث الإعدادات.

**عنوان الصورة:**\
`Sysmon Installation and Configuration`

## **Step 4 — Verify Installation**

```
Get-Service -Name "Sysmon*"

.\Sysmon64.exe -c

Get-WinEvent -ListLog "*Sysmon*"
```

ثم نفذي:

```
eventvwr.msc
```

روحي للمسار:

```
Applications and Services Logs
 └── Microsoft
     └── Windows
         └── Sysmon
             └── Operational
```

📸 **Required Screenshot:** صوري الأحداث الموجودة داخل Operational.

**عنوان الصورة:**\
`Sysmon Operational Event Logs`

## **Troubleshooting — إذا واجهتي مشكلة**

إذا التحميل ما اشتغل:

```
Test-NetConnection download.sysinternals.com -Port 443
Resolve-DnsName download.sysinternals.com
```

إذا ظهر خطأ أن الملف غير موجود:

```
Set-Location C:\Tools\Sysmon
Get-ChildItem
```

إذا ظهر خطأ متعلق بالـXML أو Schema:

```
.\Sysmon64.exe -s
```

إذا Sysmon مثبت لكن الخدمة متوقفة:

```
Get-Service -Name "Sysmon*" | Start-Service
```

إذا ما ظهرت أحداث في Event Viewer:

```
Start-Process notepad.exe

Get-WinEvent -LogName "Microsoft-Windows-Sysmon/Operational" -MaxEvents 5 | Format-List TimeCreated,Id,Message
```

**الحين ما تحتاجين تنفذين أي شيء إضافي على Kali.** خلصنا التحقق من الشبكة، ونعرف عناوين الأجهزة، فما راح نعيد هذه الخطوات.

نفذي إعداد Sysmon كامل وأرسلي الصور دفعة واحدة. باقي ما تأكدنا من حالة تثبيت Sysmon أو سياسات AppLocker من الصور الحالية، لذلك لا تفعلين قواعد AppLocker للحين.
