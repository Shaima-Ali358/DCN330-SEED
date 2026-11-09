```
Get-ChildItem "C:\Users\Administrator\Downloads\sysmon-modular-master" -Recurse -Filter "*.xml" |
Select-Object -First 20 FullName

Get-ChildItem "C:\Users\Administrator\Downloads\sysmon-modular-master" -Recurse -Filter "*.xml" |
Where-Object {
    $_.Name -match "sysmonconfig|default|config"
} |
Select-Object FullName



Get-ChildItem "C:\Users\Administrator\Downloads\sysmon-modular-master" -Recurse -Filter "*.xml" |
ForEach-Object {
    try {
        [xml]$xml = Get-Content $_.FullName -Raw
        if ($xml.DocumentElement.LocalName -eq "Sysmon") {
            $_.FullName
        }
    } catch {}
}


cd "C:\Users\Administrator\Downloads\Sysmon"

.\Sysmon64.exe -c "C:\PATH\TO\sysmonconfig.xml"

.\Sysmon64.exe -c


Start-Process notepad.exe

Get-WinEvent -FilterHashtable @{
    LogName = 'Microsoft-Windows-Sysmon/Operational'
    Id = 1
} -MaxEvents 100 |
Where-Object { $_.Message -match 'notepad.exe' } |
Select-Object -First 1 |
Format-List TimeCreated, Id, Message

```
