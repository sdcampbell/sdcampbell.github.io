---
published: true
comments: true
---
Kickoff call: During the kickoff call, ask if any user input in the thick client app can be viewed by backend web interfaces, or if there is a web client that complements the thick client. If yes, ask if they want to add it to be tested. This should be scoped to another assessment, as we don't want to give away free work. Additionally, there is a possible attack vector if user input in the thick client is not sanitized and can trigger attacks such as XSS in the web client.

Run Wireshark before opening the application. After the application is put through its paces, check the Wireshark capture for sensitive data in unencrypted communication.

Test for DLL Hijacks. Run Sysinternals Procmon/Procmon64 and add include filters for: "process name is <processname>", "result is 'NAME NOT FOUND'", "Path ends with '.dll'". If any are found, see if you can generate a dll that opens calc.exe using Metasploit's msfvenom.

Test for code signing using either Sysinternals Sigcheck, or PESecurity.

PESecurity can also test for DEP, ASLR, and Control Flow Guard enabled on exe and dll files.

PESecurity:

Display for screenshot PoC:
  
```
powershell.exe -exec bypass
Import-Module .\Get-PESecurity.psm1
Get-PESecurity -directory <path without trailing \> -recursive | ?{($_.DEP -like “FALSE”) -or ($_.ASLR -like “FALSE”) -or ($_.Authenticode -like “FALSE”) -or ($_.ControlFlowGuard -like “FALSE”)} | Select-Object -Property FileName,DEP,ASLR,Authenticode,ControlFlowGuard | Format-Table
```

To export to CSV for reporting:

```
Get-PESecurity -directory <path without trailing \> -recursive | ?{($_.DEP -like “FALSE”) -or ($_.ASLR -like “FALSE”) -or ($_.Authenticode -like “FALSE”) -or ($_.ControlFlowGuard -like “FALSE”)} | Select-Object -Property FileName,DEP,ASLR,Authenticode,ControlFlowGuard | Export-Csv -Path <path> -NoTypeInformation
```
