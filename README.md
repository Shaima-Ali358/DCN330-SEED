    ```powershell
    Get-WinEvent -LogName "Microsoft-Windows-AppLocker/EXE and DLL" -MaxEvents 10 | Format-List TimeCreated, Id, Message

    Get-WinEvent -FilterHashtable @{LogName="Microsoft-Windows-Sysmon/Operational"; ID=1} -MaxEvents 5 | Format-List TimeCreated, Id, Message


    Get-WinEvent -LogName "Microsoft-Windows-AppLocker/EXE and DLL" -MaxEvents 10 | Format-List TimeCreated, Id, Message

    runas /user:labuser "C:\path\to\TrustMe.exe"

    New-Item -Path "HKCU:\Software\SPR320_Attack" -Force
    ```

    
 ```   
New-ItemProperty -Path "HKCU:\Software\SPR320_Attack" -Name "MaliciousStartup" -Value "C:\Temp\bad.exe" -PropertyType String -Force


Get-WinEvent -FilterHashtable @{LogName="Microsoft-Windows-Sysmon/Operational"; ID=12,13} -MaxEvents 5 | Format-List TimeCreated, Id, Message


Set-Content -Path C:\Temp\Evidence.txt -Value "Secret Data"
Remove-Item -Path C:\Temp\Evidence.txt -Force

Get-WinEvent -FilterHashtable @{LogName="Microsoft-Windows-Sysmon/Operational"; ID=26} -MaxEvents 5 | Format-List TimeCreated, Id, Message

Resolve-DnsName google.com

Get-WinEvent -FilterHashtable @{LogName="Microsoft-Windows-Sysmon/Operational"; ID=22} -MaxEvents 5 | Format-List TimeCreated, Id, Message

Get-WinEvent -LogName "Microsoft-Windows-AppLocker/EXE and DLL" -MaxEvents 5 | Format-List TimeCreated, Id, Message

Get-WinEvent -FilterHashtable @{LogName="Microsoft-Windows-Sysmon/Operational"; ID=1} -MaxEvents 5 | Format-List TimeCreated, Id, Message
    ```
