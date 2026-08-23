# cyber-security
Just help me to remember and note some practice

Assets
Before we protect or attack the company we want, we have to understand each machines purpose, what they do, how they work.

Computers
The common operating systems include windows, linux, macos（sounds like something+s).

  Windows:
    The os published by microsoft, the benefit is the company always patch to the system, but the patch sometimes great, sometimes sh**. They try to promot more ad, add more functions to edge and tool bar. Which cause the more source usage.
    However, it holds the largest market share in personal computer operating systems. Most software design to run on the windows.
  
  Linux:
    penguin, contribute by the community, while a zero day exist, linux allow to patch it faster. My first impression to the linux is the command line, and no game (QAQ). It can run some exe files by wine, but it is slow and not useful.
  
  Macos:
    I never used but the community always say it is Best for Creative Types
  
  ChromeOS:
    Open files, watch video, read the email or website, that is my feeling with bought the chromebook. It does not support a lots of things, but sells $500au. It have a linux built-in, just like a vm, again have a lots of limitation, but $500au. I feel stuck while I run the app by the built-in linux, but $500au.

Servers:
  The machine receive the request and provide services. Most have no screen, no mouse, control by the client. Some of them have GUI, some not. 

Routers:
  Help to communicate in different network. Rare to see the person set the end point as the router.

Switchs:
  Help to communicate in the same network. Learn the machine address code from hosts and delivery the message.

IOT:
  Internet of things, while a thing connect to the network, it can call IOT. However, the things connect to network brings the convinient, but also more attack surface.

Basic protection
Use the antivirus app is the first chose for most people, it don't need any knowledge or skills. The hardest is to find out the system use and architecture. Here use the Avast, tencent computer manager, AVG, 360(because they are free to use or provide a period of free to use). Traditional antivirus mainly relies on signature-based detection, while modern endpoint protection also uses heuristic, behavioral, and machine-learning techniques.

Signature base
Generate the hash and compare to the libary, relias on the hash did sumbited to the liabary. The hash would change even just a single bit in the app change.

Behavioral base
While a app do something malicious, it will detect and ask you to isolate.

For the antivirus above, they all allow to do the full drive or partial drive scan, but they had not explain what method they use. Here I create a schedule task that edit the firewall to a file periodlc. 
vbs file run by wscript.exe, use wscript to run powershell command can prevent the powershell windows flash even use the hidden widows
```
Set WshShell = CreateObject("WScript.Shell")

WshShell.Run "powershell.exe -NoProfile -NonInteractive -ExecutionPolicy Bypass -File ""C:\Scripts\Netrule.ps1""", 0, False
```
```
$Folder = "<path to the file>"
$LogFile = "path to the log"

try {

    if (-not (Test-Path $Folder)) {
        "ERROR: Folder does not exist: $Folder" | Out-File $LogFile -Append
        exit 1
    }


    Get-ChildItem $Folder -Filter *.exe -File -Recurse | ForEach-Object {

        $RuleName = "Block Network - " + $_.FullName


        if (-not (Get-NetFirewallRule -DisplayName $RuleName -ErrorAction SilentlyContinue)) {

            New-NetFirewallRule `
                -DisplayName $RuleName `
                -Direction Outbound `
                -Program $_.FullName `
                -Action Block `
                -Profile Any `
                -ErrorAction Stop | Out-Null


            "$(Get-Date -Format 'yyyy-MM-dd HH:mm:ss') - CREATED: $($_.FullName)" |
                Out-File $LogFile -Append
        }
    }


    Get-NetFirewallRule |
    Where-Object DisplayName -like "Block Network -*" |
    ForEach-Object {

        $Rule = $_
        $App = $Rule | Get-NetFirewallApplicationFilter

        if ($App.Program -and -not (Test-Path $App.Program)) {


            $Rule | Remove-NetFirewallRule
        }
    }
}
catch {

    "$(Get-Date -Format 'yyyy-MM-dd HH:mm:ss') - ERROR: $($_.Exception.Message)" |
        Out-File $LogFile -Append
}
```
The command can be maliciou if it disable the firewall rule. However, 360 only mark the vbs file as trojan.
![Network Diagram](images/"屏幕截图 2026-08-23 150832.png")

