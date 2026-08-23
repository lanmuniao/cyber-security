# 网络安全
放生一些野生知识

## 内容

- [设备](#设备)
- [基础保护](#基础保护)
- [SIEM](#SIEM) to do
- [钓鱼](#钓鱼) to do
- [恶意软件](#恶意软件) to do
- [安全配置](#安全配置) to do
# 设备
在进行攻击或者保护前，我们需要知道一台设备是干什么的，才能实行对应的手段

### **电脑**
闻斗死，鳞你死，迈哦s，常见的电脑系统，好了你可以进行攻击防御了。

-**Windows**:  
    微软发布维护的系统，个人电脑装的最多的系统，性能本最好的驾驶者，更是广告和屎山的集合体 :hankey:  
    但是，我还是喜欢你啊，windows小姐。告诉linux，我今晚不回去了。 
  
-**Linux**:  
    此企鹅非彼企鹅 :penguin:, 维护更新由社区完成。优点是系统简单，缺点也是系统简单。（喜欢我的指令吗，喜欢我的板砖吗）
  
-**Macos**:  
    I never used but the community say it is Best for Creative Types.  
  
-**ChromeOS**:  
    Open files, watch video, read the email or website, that is my feeling with bought the chromebook. It does not support a lots of things, but sells $500au. It have a linux built-in, just like a vm, again have a lots of limitation, but $500au. I feel stuck while I run the app by the built-in linux, but $500au.  

-**Servers**:  
  The machine receive the request and provide services. Most have no screen, no mouse, control by the client. Some of them have GUI, some not. 

-**Routers**:  
  Help to communicate in different network. Rare to see the person set the end point as the router.

-**Switchs**:  
  Help to communicate in the same network. Learn the machine address code from hosts and delivery the message.

-**IOT**:  
  Internet of things, while a thing connect to the network, it can call IOT. However, the things connect to network brings the convinient, but also more attack surface.

# **Basic protection**
Use the antivirus app is the first chose for most people, it don't need any knowledge or skills. The hardest is to find out the system use and architecture. Here I use the Avast, tencent computer manager, AVG, 360(because they are free to use or provide a period of free to use). Traditional antivirus mainly relies on signature-based detection, while modern endpoint protection also uses heuristic, behavioral, and machine-learning techniques.

**Signature base**  
Generate the hash and compare to the libary, relias on the hash did sumbited to the liabary. The hash would change even just a single bit in the app change.

**Behavioral base**  
Detect the action of a process take, like open a pdf that is safe, but will warning user while a doc execute the powershell(danger process, classify as maliciou marco)

The antivirus above, they all allow to do the full drive or partial drive scan, but they had not explain what method they use. Here I create a schedule task that edit the firewall to a directory periodlc. 
RunNetrule.vbs file run by wscript.exe, use wscript to run powershell command can prevent the powershell windows flash even use the hidden widows
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
The command can be maliciou if it disable the firewall rule. However, only 360 mark the vbs file as trojan. I think it classify the powershell command as safe, but the vbs execute the powershell which is unwanted process. 
![Network Diagram](images/360case1.png)
<picture>
 <img alt="the case used 360" src="images/360case1.png">
</picture>

The antivirus apps marked a few files that is PUP or backdoor, which are the same. Um, after I delete the file, the program can not run. ***I delete those antivirus and execute it again, it allow to run. Maybe one of them kill the process and had not notice me.*** Additional, 360 disable execution all the program in usb, it is the function I had not seen in other antivirus, also, don't grab an unknow usb on street and insert to the pc.(not the promotion to 360, only describe the advantage, and I want to say, 360 tried to install more app without you known it, act more like a trojan than a trojan)  
To the using, choose one of the antivirus you trust, the more does not mean better. (They fights in your pc)

For security, antivirus can act as the guard and the asistant, use for easy manage with the suspecious or malicious file. However, we can not 100% trust or rely it, it may not detect the new malware, new technologies, new vulnerability.
