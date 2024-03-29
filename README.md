# FakePort
Port Virtual Open

功能：一个没有开放的端口，在外面看来是开放的，但是在本地看还是没有开放（侦听）的。

设计思路：
1. 来一个SYN，回一个ACK。注释：这个SYN是没有侦听端口的TCP。
2. 发现回RST，把这个RST干掉（即阻断）。排除：有侦听端口的TCP的RST。
3. 因为第一条，所以这个可以兼容SYN扫描和全连接的扫描。
4. 应为第二条，所以可以兼容微软自带的防火墙（关闭防火墙会回复RST）。
5. 支持IPv6，暂不考虑UDP。
6. 尽量兼容第三方的防火墙，网闸，IPS，流量清洗等软硬件。
7. 关于回复ACK，最先想到的是WSK的RAW，可是测试不行，后改为协议驱动。
8. 其实也可以利用npcap在应用层实现，但是要识别是不是侦听的SYN，要阻断RST还得加入Divert等类似驱动。

测试步骤：
1. 安装协议驱动（ndisprot）。
2. 启动协议驱动（ndisprot）。
3. 安装本驱动（fakeport）。
4. 启动本驱动（fakeport）。
5. 在外面测试本机的情况。

测试命令：
1. nmap -sS -p0-65535 IPv4
2. nmap -sS -p0-65535 -6 IPv6	注释：Only ethernet devices can be used for raw scans on Windows
3. nmap -sT -p0-65535 IPv4
4. nmap -sT -p0-65535 -6 IPv6	注释：Only ethernet devices can be used for raw scans on Windows
5. telnet ip port
6. masscan -p1-65535 IPv4 --wait=1200

测试建议：
1. 为了更好的控制效果，可以添加--scan-delay=1选项，即--scan-delay=1000ms。  
   这个是每隔指定的时间发包。效果好，但是慢。
2. 建议重定向到文件。控制台的缓存行最大为9999，要现实65535，肯定会有冲刷。
3. nmap -sS在收到ACK后会返回RST。加上驱动回复ACK慢，所以要等一个测试（ACK回复）完毕后再测试下一个。
4. --host-timeout=30m也是一个不错的选项。
   这个需要预测对方多长时间能把包回复完毕。
5. 端口0是保留的，不建议对他进行扫描。
6. nmap和masscan都不支持IPv6，或者说支持有限，telnet是支持IPv6的。
7. 因为驱动回包慢，一定要等驱动回包完毕，再测试。特别是nmap -sS在连接成功后，还会来个RST。

测试效果：见test目录下的各个测试文件。

兼容微软杀毒自带的防火墙。  
开启，不会返回RST，  
关闭，会返回RST。  

用telnet链接到本机，在两端都可以看到（netstat或者tcpview.exe）链接。  
这个链接，亦真亦假，在操作系统看来是一个假的，但是对于TCP协议来说是真的。  
这个端口，亦真亦假，在外面看来是真的，在本地看来是假的。  
所以，看到一个链接，你会如何想呢？  
这个链接是真的吗？对方的端口是在侦听吗？  

这对于欺骗，干扰对方的攻击难度有所增加。  
你可以怀疑，但是确实能连接，扫描结果所证实。  
你可以进一步证实功能，因为没有进一步的模拟，这个建议用真实的软件。  
这对于如何确定为真（实的端口），让攻击者付出一定工作量。  

有待改进的地方：
1. 获取有默认网关的网卡。
2. 改进发包的速度。
   如：改用ExNotifyCallback和协议驱动进行驱动间的通讯。
3. ACK包里有硬编码。

本工程缘于2020年做的一个废弃的功能。

2023/12/18