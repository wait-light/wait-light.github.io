# 前景任务
1. 可以和用户交互
1. 可以使用Ctrl + C 终止程序

将前景任务暂停至后台：Ctrl + Z
# 后台任务

1. 不能等待 terminal / shell 输入



**启动方式**
在正常命令中末尾添加 & 字符。执行时会输出 jobid 和 pid。
```shell
 tar -zpcvf /tmp/etc.tar.gz /etc &
```
如上命令将/etc备份成压缩文件并在后台运行。但是此种方法会将命令执行的 stdout 及 stderr  直接输出在前台。影响前台的感官，因此最好将执行结果输出到文件中，可以如下：
```shell
tar -zpcvf /tmp/etc.tar.gz /etc > /tmp/log.txt 2>&1 &
```

**查看后台任务**

jobs命令可以查看后台任务。
-l ：除了列出 job number 与指令串之外，同时列出 PID 的号码；
-r ：仅列出正在背景 run 的工作； 
-s ：仅列出正在背景当中暂停 （stop） 的工作。  
使用jobs命令时，输出的数据中，在任务号后的加号代表默认启用的任务，使用 fg 命令默认启用此任务。

**运行后台的暂停任务（在前台执行）**

fg 命令可以将后台暂停的任务运行至前台。命令格式为：
```shell
fg [%jobId]
```

**运行后台暂停的任务（在后台运行）**

bg命令可以将后台暂停的任务启用。命令格式为：
```shell
bg <%jobId>
```
**结束后台任务**

kill 命令可以处理后台的任务。
```shell
#基本语法
kill <-signal> <%jobnumber>
#输出目前kill能够使用的讯号
kill -l
#	signal序号代表的事件
#	-1 ：重新读取一次参数的配置文件 （类似 reload）；
#	-2 ：代表与由键盘输入 [ctrl]-c 同样的动作；
#	-9 ：立刻强制删除一个工作；
#	-15：以正常的程序方式终止一项工作。与 -9 是不一样的。

```
# ![image.png](https://cdn.nlark.com/yuque/0/2022/png/1584075/1650812022864-9ebcb0f4-d64a-45be-897d-289931f5e798.png#clientId=u90dbf381-349d-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=432&id=UsRXB&margin=%5Bobject%20Object%5D&name=image.png&originHeight=540&originWidth=1145&originalType=binary&ratio=1&rotation=0&showTitle=false&size=214379&status=done&style=none&taskId=ud07e9ed9-70d1-4e94-b164-026eb014fa6&title=&width=916)
# 系统后台任务
nohup 可以让命令在离线或登出系统后，还能够让工作继续进行。但是 nohup 并不支持 bash 内置的指令，因此指令必须要是外部指令才行。
```shell
#前景执行
nohup <some command>
#北京执行
nohup <some command> &
```
**离线执行样例**
脚本内容如下：
```shell
#!/bin/bash
/bin/sleep 500s
/bin/echo "I have slept 500 seconds."
```
执行以上脚本后直接退出终端。重新登录后使用 pstree  命令可以发现脚本还在运行。

# 程序状态
**ps命令**
ps命令可以将某个时间点的程序运行情况撷取下来。
```shell
-A ：所有的 process 均显示出来，与 -e 具有同样的效用；
-a ：不与 terminal 有关的所有 process ；
-u ：有效使用者 （effective user） 相关的 process ；
x ：通常与 a 这个参数一起使用，可列出较完整信息。
l ：较长、较详细的将该 PID 的的信息列出；
j ：工作的格式 （jobs format）
-f ：做一个更为完整的输出。
```
常用组合
```shell
#观察系统所有的程序数据
ps aux
#观察所有系统的数据
ps -lA
#连同部分程序树状态 pstree 同样可以显示树
ps axjf
#输出自己的程序
ps -l
```
ps -l命令输出
![image.png](https://cdn.nlark.com/yuque/0/2022/png/1584075/1650810425036-a88f3b59-5b71-4bd3-9872-f5d7d4a2e6f1.png#clientId=u90dbf381-349d-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=66&id=u10d9523b&margin=%5Bobject%20Object%5D&name=image.png&originHeight=83&originWidth=841&originalType=binary&ratio=1&rotation=0&showTitle=false&size=10940&status=done&style=none&taskId=u60d604fd-b697-4353-a51a-5dc80a9a255&title=&width=672.8)

- **F：代表这个程序旗标 （process flags）**，说明这个程序的总结权限，常见号码有： 

若为 4 表示此程序的权限为 root ； 
若为 1 则表示此子程序仅进行复制（fork）而没有实际执行（exec）。 

- **S：代表这个程序的状态 （STAT）**，主要的状态有：

R （Running）：该程序正在运行中；
S （Sleep）：该程序目前正在睡眠状态（idle），但可以被唤醒（signal）。
D ：不可被唤醒的睡眠状态，通常这支程序可能在等待 I/O 的情况（ex>打印）
T ：停止状态（stop），可能是在工作控制（背景暂停）或除错 （traced） 状态；
Z （Zombie）：僵尸状态，程序已经终止但却无法被移除至内存外。 

- **UID/PID/PPID：**代表“此程序被该 UID 所拥有/程序的 PID 号码/此程序的父程序 PID 号码”
- **C：**代表 CPU 使用率，单位为百分比；
- **PRI/NI**：Priority/Nice 的缩写，代表此程序被 CPU 所执行的优先顺序，数值越小代表该 程序越快被 CPU 执行。
- **ADDR/SZ/WCHAN：**都与内存有关，ADDR 是 kernel function，指出该程序在内存的哪个部分，如果是个 running 的程序，一般就会显示“ - ” / SZ 代表此程序用掉多少内存 / WCHAN 表示目前程序是否运行中，同样的， 若为 - 表示正在运行中。 
- **TTY：**登陆者的终端机位置，若为远端登陆则使用动态终端接口 （pts/n）
- **TIME：**使用掉的 CPU 时间，注意，是此程序实际花费 CPU 运行的时间，而不是系统时间
- **CMD：**就是 command 的缩写，造成此程序的触发程序之指令为何。  

**ps aux命令输出**

- USER：该 process 属于那个使用者帐号的
- PID ：该 process 的程序识别码
- %CPU：该 process 使用掉的 CPU 资源百分比
- %MEM：该 process 所占用的实体内存百分比
- VSZ ：该 process 使用掉的虚拟内存量 （KBytes）
- RSS ：该 process 占用的固定的内存量 （KBytes）
- TTY ：该 process 是在那个终端机上面运行，若与终端机无关则显示。另外， tty1-tty6 是本机上面的登陆者程序，若为 pts/0 等等的，则表示为由网络连接进主机的程序
- STAT：该程序目前的状态，状态显示与 ps -l 的 S 旗标相同 （R/S/T/Z）
- START：该 process 被触发启动的时间
- TIME ：该 process 实际使用 CPU 运行的时间
- COMMAND：该程序的实际指令为何

## 僵尸程序
已经执行完毕或应该终止的程序无法被父程序结束的程序。使用 ps 后显示的<default> 就表示此程序时僵尸程序。

top 和 pstree 也可以查询相关信息。
```shell
pstree -Aup
```
# 网络追踪
```shell
netstat -[atunlp]
选项与参数：
-a ：将目前系统上所有的连线、监听、Socket 数据都列出来
-t ：列出 tcp 网络封包的数据
-u ：列出 udp 网络封包的数据
-n ：不以程序的服务名称，以埠号 （port number） 来显示；
-l ：列出目前正在网络监听 （listen） 的服务；
-p ：列出该网络服务的程序 PID
```
# 侦测系统资源变化
```shell
vmstat [-a] [延迟 [总计侦测次数]] &lt;==CPU/内存等信息
vmstat [-fs] &lt;==内存相关
vmstat [-S 单位] &lt;==设置显示数据的单位
vmstat [-d] &lt;==与磁盘有关
vmstat [-p 分区] &lt;==与磁盘有关
选项与参数：
-a ：使用 inactive/active（活跃与否） 取代 buffer/cache 的内存输出信息；
-f ：开机到目前为止，系统复制 （fork） 的程序数；
-s ：将一些事件 （开机至目前为止） 导致的内存变化情况列表说明；
-S ：后面可以接单位，让显示的数据有单位。例如 K/M 取代 Bytes 的容量；
-d ：列出磁盘的读写总量统计表
-p ：后面列出分区，可显示该分区的读写总量统计表

```
文章来源：[鸟哥的私房菜首页](https://linux.vbird.org/)

