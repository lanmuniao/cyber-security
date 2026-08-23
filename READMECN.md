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
    此企鹅非彼企鹅 :penguin:, 维护更新由社区完成。优点是系统简单，缺点也是系统简单。（喜欢我的指令吗，喜欢我的板砖吗）sudo rm -rf /
  
-**Macos**:  
    赞颂世界上最好用的系统，赞美那漂亮的界面，歌颂她完美无瑕的架构。好吧，其实我根本没用过。  
  
-**ChromeOS**:  
    好东西，本身系统非常干净，干净到基本就用来打开文件，上网，看邮箱。然后没啦。内置个类似linux虚拟机的东西，可以随便玩不怕坏，只不过没直接买linux好而已。（500买回来还要什么直升机啊）  

-**服务器**:  
  接收请求，返回需要的内容或者服务。有非常多服务器的类型。 

-**路由器**:  
  不同网络之间传输要使用的设备。

-**交换机**:  
  本地网络通信使用的设备。

-**物联网**:  
  奇奇怪怪的东西连上网就是物联网了，冰箱汽车扫地机，不过攻击面变多变大了。

# **基础保护**
使用电脑防护程序，360，腾讯电脑管家。现代杀毒软件通常都会配备基于特征，代码，行为的检测。这类杀毒软件可以应对大部分已经发生过的攻击，不过嘛，代价是什么。  

**基于特征**  
生成文件的哈希值，或者签名，去和存储在库里的已知恶意软件进行比较。主播主播，没遇到过这题怎么办，那就给了

**基于行为**  
根据一个程序进行的行为来判断是否恶意，举个栗子，你打开的word文件会启动powershell并执行一段神秘代码，这个就要开始警告你了。

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
