    ```powershell
    Get-WinEvent -LogName "Microsoft-Windows-AppLocker/EXE and DLL" -MaxEvents 10 | Format-List TimeCreated, Id, Message

    Get-WinEvent -FilterHashtable @{LogName="Microsoft-Windows-Sysmon/Operational"; ID=1} -MaxEvents 5 | Format-List TimeCreated, Id, Message
    ```
