# 1.Linux

## 1.other

在电脑【不是虚拟机】上安装linux的时候需要先记录计算机的硬件情况，获取硬件的兼容性列表和驱动支持：

http://hardware.redhat.com/hcl/

如果硬件与linux系统版本不兼容，则不能安装；

- 关于软件版本的约定：版本格式r.x.y

r：release version

x：偶数：稳定版本；奇数：开发中版本

y：正式错误修补的次数

eg：2.0.38；2.2.16

- linux中先安装gcc：

  ```shell
  yum -y install gcc
  yum -y install gcc-c++
  ```

- 两个linux之间使用ssh免密传输时使用：

  <https://blog.csdn.net/xinshui151/article/details/79187563>




## 2.上传文件(ubuntu)

- windows端：

  下载winscp/[FileZilla](https://filezilla-project.org/download.php?type=client) 等软件，并安装

- linux端：

  ```shell
  #Linux远程连接需要开通sshd服务，监听22端口
  $ sudo apt-get update
  $ sudo apt-get install openssh-server
  ```



## 3.ubuntu添加root用户登录 

```shell
1. $ sudo gedit /usr/share/lightdm/lightdm.conf.d/50-ubuntu.conf
   #在编辑区添加：
   greeter-show-manual-login=true
   all-guest=false
2. $ sudo passwd root	#输入设置root密码
3. $ cd /etc/pam.d
   $ vim gdm-autologin
   #注释掉auth required pam_succeed_if.so user != root quiet_success这一行，保存
   $ vim gdm-password 
   #注释掉 auth required pam_succeed_if.so user != root quiet_success这一行，保存
4. $ vim /root/.profile
   #将文件末尾的mesg n || true这一行修改成tty -s&&mesg n || true， 保存
5. 重启系统，输入root用户名和密码，登录系统
```



## 4.redhat添加本地yum源

1. 先卸载系统上已经默认安装的yum，据说默认的yum 是需要注册的 

   ```shell
   $ rpm -aq|grep yum|xargs rpm -e --nodeps
   ```

2. 卸载完后挂载上系统镜像

   ```shell
   $ mount /dev/cdrom /mnt 
   ```

3. 将系统安装包拷贝到本地

   ```shell
   $ mkdir /redhat-iso     #创建一个目录存放系统安装包;可以copy到自己指定的目录
   $ cp -Rf /mnt/* /redhat-iso  
   ```

4. 安装yum 工具

   ```shell
   $ cd /redhat-iso/Packages/   #系统rpm 所在目录
   $ rpm -ivh yum-metadata-parser-1.1.2-16.el6.x86_64.rpm  -force  
   #此包需要强制安装正常安装会提示“no key”;提示了也没有关系；
   $ rpm -ivh yum-3.2.29-17.el6.noarch.rpm
   ```

5. 安装完成后需要配置本地源

   ```shell
   $ cd /etc/yum.repos.d/                        #进入yum 源配置目录
   $ cp rhel-source.repo rhel-source.repo.bak    #备份原配置文件
   $ vi rhel-source.repo                         #修改配置文件
   #####################修改为如下配置，或者添加一个新的配置文件，内容如下####################
   [rhel-source-beta]               #YUM源标签，与其他YUM源标签不同即可   
   name=local iso         			#YUM源名称，与其他YUM源名称不同即可                
   baseurl=file:///redhat-iso       #指定YUM源 #指定yum 源目录                      
   enabled=1                        #是否可用，0 不可用，1可用                              
   gpgcheck=0                       #是否进行数字签名检查， 1为检查，0为不检查。              
   gpgkey=file:///redhat-iso/RPM-GPG-KEY-redhat-beta,file:///redhat-iso/RPM-GPG-KEY-redhat-release                          #指定RED Hat 发行的数字签名公钥文件。
   ```


6. 保存修改完的配置文件，测试一下本地yum 是否正常

   ```shell
   $ yum clean all   #清除yum 缓存
   $ yum grouplist   #列出所有可用的安装组件
   ```

   

## 5.vim快捷键

| 说明           | 命令                                                    |
| -------------- | ------------------------------------------------------- |
| 复制+粘贴      | yy(复制) + p(粘贴)   ；   5yy 复制以下5行               |
| 删除           | dd  ；  5dd                                             |
| 命令行 /关键字 | enter查找       n查找下一个                             |
| 行号设置       | 命令行: set nu(设置)   :set nonu(取消)                  |
| 撤销           | u                                                       |
| 编辑           | 末行G 首行gg<br />定位到文档第20行：显示行号→20→shift+g |

  

## 6.一些常用命令

```shell
####Linux路由表
$ route                        #查看
$ ip route add 192.168.2.0/24 via 192.168.59.104 dev eth0
#通过本机的eth0网卡，查找（连接）192.168.59.104上的192.168.2.0/24网段
####进程查看
$ netstat -nptl|grep 12181   #找出进程
$ kill 16261	#kill掉
$ ps -aux | grep zookeeper

#cp命令使用的时候，会有覆盖提示；\cp覆盖的时候不会提示
$ cat -n filename   #查看的时候显示行号
$ cal   #显示日历
$ head [-n 5] filename   #查看文件前n行（默认为10行）
$ tail [-n 5] filename   #查看文件后n行（默认为10行）
$ tail -f filename       #实时追踪文档的所有更新
$ ln -s 连接              #建立软连接
$ find / -size +20M   	 # +n大于 -n小于 n等于
$ history [n]  			#显示后n条命令
$ !n             		#执行history中编号为n的命令
```

## 7.一些shell的命令

```shell
1.	$ /bin/bash -x for2Sum.sh		#查看脚本执行的过程
2.	$ /bin/bash -n filename.sh		#验证脚本有没有问题
3.	tar czf all.tgz *			    #打包目录中的所有文件
4.	tar xzf all.tgz -C ./test		#解压到，./test目录中
5.	find . -name "*.tgz"			#查找当前目录中，以 .tgz 结尾的文件
6.	find . -name "*.sh" | tail -2	 #查找当前目录中，以 .sh 结尾文件的后两条数据
7.	ssh-keygen	#两个linux之间使用ssh免密传输时使用
8.	scp -r test.sh glzj@127.0.0.1:/home/glzj  
	#远程cp文件，-r 表示可以是目录也可以是文件
9.	ssh -l glzj 127.0.0.1 'ls /home'	#远程主机执行命令
10.	read -p "Please input numbers:" input | read input
	#读取数据存入变量input；（2.7.2示例2 while逐行读取某个文件）
11.	echo a b c | awk '{print $2}'		#取第二个参数，用于read读值的时候
```




# 2.文件系统

## 1.目录详解

### 1.1. /bin目录

系统启动时需要执行的文件；或普通用户可能用的命令(可能在引导启动后)。这些命 令都是二进制文件的可执行程序(bin是binary的简称)，多是系统中重要的系统文件。

### 1.2. /sbin目录

/sbin目录类似/bin ，也用于存储二进制文件。因为其中的大部分文件多是系统管理员使用的基本的系统程序，所以虽然普通用户必要且允许时可以使用，但一般不给普通用户使用。

### 1.3. /root目录

/root目录是超级用户的主目录。启动linux时使用的一些核心文件。Ex：操作系统内核、引导程序Grub等；

### 1.4. /lib目录

#标准程序设计库，又叫动态链接共享库，作用类似windows里的.dll文件

/lib目录是根文件系统上的程序所需的共享库，存放了根文件系统程序运行所需的共享文件。 这些文件包含了可被许多程序共享的代码，以避免每个程序都包含有相同的子程序的副本。

- /lib/modules目录

/lib/modules目录包含系统核心可加载各种模块，尤其是那些在恢复损坏的系统时重 新引导系统所需的模块(例如网络和文件系统驱动)。

### 1.5. /tmp目录

公用的临时文件储存点

### 1.6. /boot目录

/boot目录存放引导加载器(bootstrap loader)使用的文件，如lilo，核心映像也经常放在这里，而不是放在根目录中。但是如果有许多核心映像，这个目录就可能变得很大，这时使用单独的 文件系统会更好一些。还有一点要注意的是，要确保核心映像必须在ide硬盘的前1024柱面内。

### 1.7. /mnt目录

零时用于挂载文件系统的地方。一般情况下该目录是空的，将要挂载分区时在这个目录下建立挂载目录，并挂载

### 1.8. /home目录

其他文件系统的安装点。

目录树可以分为小的部分，每个部分可以在自己的磁盘或分区上。主要部分是根、/usr 、/var 和 /home 文件系统。每个部分有不同的目的。

- 根文件系统

  每台机器都有根文件系统，它包含系统引导和使其他文件系统得以mount所必要的文件，根文件系统应该有单用户状态所必须的足够的内容。还应该包括修复损坏 系统、恢复备份等的工具。

- /home 包含用户家目录，即系统上的所有实际数据。


### 1.9 /lost+found

这个目录平时是空着的，系统非正常关机而留下“无家可归”的文件就在这里； 



## 2./etc文件系统详解

/etc目录包含各种系统配置文件，下面说明其中的一些。许多网络配置文件也在/etc中。

- /etc/rc或/etc/rc.d或/etc/rc?.d：启动、或改变运行级时运行的脚本或脚本的目录。
- /etc/passwd：用户数据库，其中的域给出了用户名、真实姓名、用户起始目 录、加密口令和用户的其他信息。
- /etc/fdprm：软盘参数表，用以说明不同的软盘格式。可用setfdprm进行设置。更多的信息见setfdprm的帮助页。
- /etc/fstab：指定启动时需要自动安装的文件系统列表。也包括用swapon -a启用的swap区的信息。
- /etc/group：类似/etc/passwd ，但说明的不是用户信息而是组的信息。包括组的各种数据。
- /etc/inittab：init 的配置文件。运行级别的配置；#init 3 切换到3级别(零时的)
- /etc/issue：包括用户在登录提示符前的输出信息。通常包括系统的一段短说明 或欢迎信息。具体内容由系统管理员确定。
- /etc/magic：“file”的配置文件。包含不同文件格式的说 明，“file”基于它猜测文件类型。
- /etc/motd：motd是message of the day的缩写，用户成功登录后自动输出。内容由系统管理员确定。

常用于通告信息，如计划关机时间的警告等。

- /etc/mtab：当前安装的文件系统列表。由脚本(scritp)初始化，并由 mount命令自动更新。当需要一个当前安装的文件系统的列表时使用(例如df命令)。
- /etc/shadow：在安装了shadow口令软件的系统上的口令文件。口令文件将/etc/passwd文件中的加密口令移动到/etc/shadow中，而后者只对超级用户(root)可读。这使破译口令更困难，以此增加系统的安全性。
- /etc/login.defs：login命令的配置文件。
- /etc/printcap：类似/etc/termcap ，但针对打印机。语法不同。
- /etc/profile 、/etc/csh.login、/etc/csh.cshrc：登 录或启动时bourne或cshells执行的文件。这允许系统管理员为所有用户建立全局缺省环境。
- /etc/securetty：确认安全终端，即哪个终端允许超级用户(root) 登录。一般只列出虚拟控制台，这样就不可能(至少很困难)通过调制解调器(modem)或网络闯入系统并得到超级用户特权。
- /etc/shells：列出可以使用的shell。chsh命令允许用户在本文件指定范围内改变登录的shell。提供一台机器ftp服务的服务进程ftpd检查用户shell是否列在/etc/shells文件中，如果不是，将不允许该用户登录。
- /etc/termcap：终端性能数据库。说明不同的终端用什么“转义序列”控制。写程序时不直接输出转义序列(这样只能工作于特定品牌的终端)，而是从/etc/termcap中查找要做的工作的正确序列。这样，多数的程序可以在多数终端上运行。

 

## 3./dev文件系统详解

/dev目录包括所有设备的设备文件。设备文件用特定的约定命名，这在设备列表中说明。设备文件在安装时由系统产生，以后可以用/dev/makedev描述。        /dev/makedev.local 是系统管理员为本地设备文件(或连接)写的描述文稿(即如一些非标准设备驱动不是标准makedev 的一部分)。下面简要介绍/dev下 一些常用文件。

- /dev/console：系统控制台，也就是直接和系统连接的监视器。
- /dev/hd：ide硬盘驱动程序接口。如：/dev/hda指的是第一个硬盘，had1则是指/dev/hda的第一个分区。如系统中有其他的硬盘，则依次为/dev /hdb、/dev/hdc、. . . . . .；如有多个分区则依次为hda1、hda2 . . . . . .
- /dev/sd：scsi磁盘驱动程序接口。如系统有scsi硬盘，就不会访问/dev/had， 而会访问/dev/sda。
- /dev/fd：软驱设备驱动程序。如：/dev/fd0指 系统的第一个软盘，也就是通常所说的a盘，/dev/fd1指第二个软盘，. . . . . .而/dev/fd1 h1440则表示访问驱动器1中的4.5高密盘。
- /dev/st：scsi磁带驱动器驱动程序。
- /dev/tty：提供虚拟控制台支持。如：/dev/tty1指 的是系统的第一个虚拟控制台，/dev/tty2则是系统的第二个虚拟控制台。
- /dev/pty：提供远程登陆伪终端支持。在进行telnet登录时就要用到/dev/pty设备。
- /dev/ttys：计算机串行接口，对于dos来说就是“com1”口。
- /dev/cua：计算机串行接口，与调制解调器一起使用的设备。
- /dev/null：“黑洞”，所有写入该设备的信息都将消失。例如：当想要将屏幕上的输出信息隐藏起来时，只要将输出信息输入到/dev/null中即可。

 

## 4./usr文件系统详解

/usr是个很重要的目录，通常这一文件系统很大，因为所有程序安装在这里。/usr里的所有文件一般来自linux发行版；本地安装的程序和其他东西在/usr/local下，因为这样可以在升级新版系统或新发行版时无须重新安装全部程序。/usr目录下的许多内容是可选的，但这些功能会使用户使用系统更加有效。/usr可容纳许多大型的软件包和它们的配置文件。下面列出一些重要的目录(一些不太重要的目录被省略了)。

- /usr/x11r6：包含x window系统的所有可执行程序、配置文件和支持文件。为简化x的开发和安装，x的文件没有集成到系统中。x window系统是一个功能强大的图形环境，提供了大量的图形工具程序。用户如果对microsoft windows比较熟悉的话，就不会对x window系统感到束手无策了。
- /usr/x386：类似/usr/x11r6，但是是专门给x 11 release 5的。
- /usr/bin：集中了几乎所有用户命令，是系统的软件库。另有些命令在/bin或/usr/local/bin中。
- /usr/sbin：包括了根文件系统不必要的系统管理命令，例如多数服务程序。
- /usr/man、/usr/info、/usr/doc：这些目录包含所有手册页、 gnu信息文档和各种其他文档文件。每个联机手册的“节”都有两个子目录。例如：/usr/man/man1中包含联机手册第一节的源码(没有格式化的原始文件)，/usr/man/cat1包含第一节已格式化的内容。联机手册分为以下九节：内部命令、系统调用、库函数、设备、文件格式、游戏、宏软件包、 系统管理和核心程序。
- /usr/include：包含了c语言的头文件，这些文件多以.h结尾，用来描述c 语言程序中用到的数据结构、子过程和常量。为了保持一致性，这实际上应该放在/usr/lib下，但习惯上一直沿用了这 个名字。
- /usr/lib：包含了程序或子系统的不变的数据文件，包括一些site – wide配置文件。名字lib来源于库(library); 编程的原始库也存在/usr/lib 里。当编译程序时，程序便会和其中的库进行连接。也有许多程序把配置文件存入其中。
- /usr/local：本地安装的软件和其他文件放在这里。这与/usr很相似。用户 可能会在这发现一些比较大的软件包，如tex、emacs等。

## 5./var文件系统详解

/var包含系统一般运行时要改变的数据。通常这些数据所在的目录的大小是要经常变化或扩充的。原来/var目录中有些内容是在/usr中的，但为了保持/usr目录的相对稳定，就把那些需要经常改变的目录放到/var中了。每个系统是特定的， 即不通过网络与其他计算机共享。下面列出一些重要的目录(一些不太重要的目录省略了)。

- /var/catman：包括了格式化过的帮助(man)页。帮助页的源文件一般存在 /usr/man/catman中；有些man页可能有预格式化的版本，存在/usr/man/cat中。而其他的man页在第一次看时都需要格式化，格 式化完的版本存在/var/man中，这样其他人再看相同的页时就无须等待格式化了。(/var/catman经常被 清除，就像清除临时目录一样。)
- /var/lib：存放系统正常运行时要改变的文件。
- /var/local：存放/usr/local中安装的程序的可变数据(即系统管理员安装的程序)。注意，如果必要，即使本地安装的程序也会使用其他/var目录，例如/var/lock 。
- /var/lock：锁定文件。许多程序遵循在/var/lock中 产生一个锁定文件的约定，以用来支持他们正在使用某个特定的设备或文件。其他程序注意到这个锁定文件时，就不会再使用这个设备或文件。
- /var/log：各种程序的日志(log)文件，尤其是login (/var/log/wtmplog纪 录所有到系统的登录和注销) 和syslog (/var/log/messages 纪录存储所有核心和系统程序信息)。/var/log 里的文件经常不确定地增长，应该定期清除。
- /var/run：保存在下一次系统引导前有效的关于系统的信息文件。例如，/var/run/utmp包 含当前登录的用户的信息。
- /var/spool：放置“假脱机(spool)”程序的目录，如mail、 news、打印队列和其他队列工作的目录。每个不同的spool在/var/spool下有自己的子目录，例如，用户的邮箱就存放在/var/spool/mail 中。
- /var/tmp：比/tmp允许更大的或需要存在较长时间的临时文件。注意系统管理 员可能不允许/var/tmp有很旧的文件。

 

## 6./proc文件系统详解

/proc文件系统是一个伪的文件系统，就是说它是一个实际上不存在的目录，因而这是一个非常特殊的目录。它并不存在于某个磁盘上，而是由核心在内存中产生。这个目录用于提供关于系统的信息。下面说明一些最重要的文件和目录(/proc文件系统在proc man页中有更详细的说明)。

 虚拟的目录，是系统内存的映射。可以直接访问该目录来获取系统信息

- /proc/x：关于进程x的信息目录，这x是这一进程的标识号。每个进程在 /proc下有一个名为自己进程号的目录。
- /proc/cpuinfo：存放处理器(cpu)的信息，如cpu的类型、制造商、 型号和性能等。
- /proc/devices：当前运行的核心配置的设备驱动的列表。
- /proc/dma：显示当前使用的dma通道。
- /proc/filesystems：核心配置的文件系统信息。
- /proc/interrupts：显示被占用的中断信息和占用者的信息，以及被占用 的数量。
- /proc/ioports：当前使用的i/o端口。
- /proc/kcore：系统物理内存映像。与物理内存大小完全一样，然而实际上没有 占用这么多内存；它仅仅是在程序访问它时才被创建。(注意：除非你把它拷贝到什么地方，否则/proc下没有任何东西占用任何磁盘空间。)
- /proc/kmsg：核心输出的消息。也会被送到syslog。
- /proc/ksyms：核心符号表。
- /proc/loadavg：系统“平均负载”；3个没有意义的指示器指出系统当前 的工作量。
- /proc/meminfo：各种存储器使用信息，包括物理内存和交换分区 (swap)。
- /proc/modules：存放当前加载了哪些核心模块信息。
- /proc/net：网络协议状态信息。
- /proc/self：存放到查看/proc的 程序的进程目录的符号连接。当2个进程查看/proc时，这将会是不同的连接。这主要便于程序得到它自己的进程目录。
- /proc/stat：系统的不同状态，例如，系统启动后页面发生错误的次数。
- /proc/uptime：系统启动的时间长度。
- /proc/version：核心版本。

 

## 7.总结

- /usr/local

一般是你安装软件的目录，这个目录就相当于在windows下的programefiles这个目录 

这里主要存放那些手动安装的软件，即不是通过“新立得”或apt-get安装的软件。它和/usr目录具有相类似的目录结构。让软件包管理器来管理/usr目录，而把自定义的脚本(scripts)放到/usr/local目录下面。

- /opt

这个目录是一些大型软件的安装目录，或者是一些服务程序的安装目录

ex：刚才装的测试版firefox，就可以装到/opt/firefox_beta目录下，/opt/firefox_beta目录下面就包含了运行firefox所需要的所有文件、库、数据等等。要删除firefox的时候，你只需删除/opt/firefox_beta目录即可，非常简单。

- /var/log

各种程序的日志(log)文件，尤其是login (/var/log/wtmplog纪 录所有到系统的登录和注销) 和syslog (/var/log/messages 纪录存储所有核心和系统程序信息)。/var/log 里的文件经常不确定地增长，应该定期清除。



# 3.shell

## 1.第一个shell脚本

```shell
#!/bin/bash
echo "Hello World !"
```

\#! 是一个约定的标记，它告诉系统这个脚本需要什么解释器来执行，即使用哪一种 Shell。

echo 命令用于向窗口输出文本。

## 2.shell脚本的执行方式

- 作为可执行程序

  ```shell
  #将上面的代码保存为 test.sh，并 cd 到相应目录：
  $ chmod +x ./test.sh  #使脚本具有执行权限
  $ ./test.sh  #执行脚本
  ```

  **注意：**一定要写成 ./test.sh，而不是 test.sh，运行其它二进制的程序也一样，直接写 test.sh，linux 系统会去 PATH 里寻找有没有叫 test.sh 的，而只有 /bin, /sbin, /usr/bin，/usr/sbin 等在 PATH 里，你的当前目录通常不在 PATH 里，所以写成 test.sh 是会找不到命令的，要用 ./test.sh 告诉系统说，就在当前目录找。

- 作为解释器参数

  ```shell
  #这种运行方式是，直接运行解释器，其参数就是 shell 脚本的文件名，如： 
  $ /bin/sh test.sh
  $ /bin/php test.php
  #这种方式运行的脚本，不需要在第一行指定解释器信息，写了也没用。
  ```

## 3.shell中的变量

```shell
#变量名和等号之间不能有空格；可以不加$符号
your_name="runoob.com"

#可以用语句给变量赋值，如：
for file in $(ls /etc)
do
echo $file
done
#以上语句将 /etc 下目录的文件名循环出来。

#使用一个定义过的变量，只要在变量名前面加$符号即可
your_name="qinjx"
echo $your_name
echo ${your_name}
#花括号是可选的，是为了帮助解释器识别变量的边界
```

## 4.SHELL常见的系统变量

```shell
$0        #当前程序的名称
$n        #当前程序的第n个参数，n=1,2,...9
$*        #当前程序的所有参数（不包括程序本身）
$#        #当前程序的参数个数（不包括程序本身）
$？       #命令或程序执行完后的状态，一般返回0表示执行成功
##############################################################
echo "\033[32m====================\033[0m"
#echo "\033[32m====================\033[1m" #1->down all is green
echo "The $0 is $0"
echo "The $1 is $1"
echo "The $2 is $2"
echo "The $? is $?"
echo "The $* is $*"
echo "The $# is $#"
```

## 5.If条件语句

### 5.1语法

```shell
if condition1
then
    command1
elif condition2
    command2
else
    commandN
fi

#注意事项：
if (($NUM> 60))          #((计算)) / [[比较]] :可以是比较，也可以是计算
```

### 5.2逻辑运算符解析：

```shell
-f       判断文件是否存在 eg：if [ -f filename ]
-d       判断目录是否存在 eg：if [ -d dir ]
-eq      等于  应用于：整型比较
-ne      不等于  应用于：整型比较
-lt      小于   应用于：整型比较
-gt      大于     应用于：整型比较
-le      小于等于   应用于：整型比较
-ge      大于等于   应用于：整型比较
-a       and 都成立     逻辑表达式 -a 逻辑表达式
-o       or 单方成立    逻辑表达式 -o 逻辑表达式
-z       空字符串
```

### 5.3示例1：判断是否存在目录

```shell
#!/bin/bash
if [ -d ./test ];then
        echo "test dir is exist"
else
        echo "create test dir in myfile"
        mkdir ./test
fi
```

### 5.4示例2：判断成绩

```shell
#!/bin/bash
score=$1
if [ -z $score ];then
        echo "Usage:{$0 60|80};"
        exit
fi
if [[ $score -gt 85 ]];then
        echo "very good" 
elif [[ $score -gt 75 ]];then
        echo "good"
elif [[ $score -ge 60 ]];then
        echo "pass"
else
        echo "no pass"
fi
```

### 5.5示例3：备份mysql数据库

```shell
#!/bin/bash
#auto backup mysql db
#by authors nyf
#define var
BAK_DIR=/data/backup/date +%Y%m%d
MYSQLDB=test1
MYSQLUSER=backup
MYSQLPW=123456
MYSQLCMD=/usr/bin/mysqldump

#check is root user
if [ $UID -ne 0 ];then
        echo "Must to be use root for exec shell."
        exit
fi

#check dir exist
if [ ! -d $BAK_DIR ];then
        mkdir -p $BAK_DIR
        echo "\033[32mCreate $BAK_DIR successfully!\033[0m"
else
        echo "\033[32mThis $BAK_DIR is exist...\033[0m"
fi

#backup database commend
#mysqldump -ubackup -p123456 test1 > $BAK_DIR/test1.sql
$MYSQLCMD -u$MYSQLUSER -p$MYSQLPW -d $MYSQLDB > $BAK_DIR/$MYSQLDB.sql

#check pro is true
if [ $? -eq 0 ];then
        echo "\033[32mThe Mysql Backup $MYSQLDB Successfully !\033[0m"
else
        echo "\033[32mThe Mysql Backup $MYSQLDB Failed,Please check.\033[0m"
fi
```

```shell
#创建备份用用户，并赋予权限
mysql> grant all on test1.* to backup@'localhost' identified by "123456";
```



### 5.6示例4 一键安装LAMP

1. Apache服务器安装部署

```shell
#下载httpd-2.2.27.tar.gz版本，下载URL，解压，进入安装目录
configure;make;make install
```

2. Mysql服务器安装

```shell
#下载mysql-5.5.20.tar.bz2版本，下载URL，解压，进入安装目录
configure;make;make install
```

3. PHP服务器安装

```shell
#下载php-5.3.8.tar.bz2版本，下载URL，解压，进入安装目录
configure;make;make install
```

4. LAMP架构的整合和服务启动

```shell
/usr/local/apache2/bin/apachectl start
vi htdocs/index.php
<?php
phpinfo();
?>
```

---

```shell
#!/bin/bash
#Auto install LAMP
#print list
if [ -z $1 -o "$1" -eq "help" ];then
        echo -e "\033[32mPlease select install Menu:\033[0m"
        echo "1.install apache;"
        echo "2.install mysql;"
        echo "3.install php;"
        echo "4.config index.php and start LAMP;"
        echo -e "\033[34mUsage:{$0 1|2|...|help};\033[0m"
fi

#Http define path variable
H_FILES=httpd-2.2.27.tar.bz2
H_FILES_DIR=HTTPD-2.2.27
H_URL=http://mirrors.cnnic.cn/apache/httpd/
#Mysql define path variable
M_FILES=mysql-5.5.20.tar.gz
M_FILES_DIR=mysql-5.5.20
M_URL=http://down1.chinaunix.net/distfiles/
M_PREFIX=/usr/local/mysql/
#PHP define path variable
p_FILES=php-5.3.28.tar.bz2
p_FILES_DIR=php-5.3.28
p_URL=http://mirrors.sohu.com/php/
p_PREFIX=/usr/local/php5/
echo -e "\033[32m------------------------------------------\033[0m"
#install httpd web server
if [[ "$1" -eq "1" ]];then
        wget -c $H_URL/$H_FILES && tar -jxvf $H_FILES && cd $H_FILES_DIR &&./configure --prefix=$H_PREFIX
        if [ $? -eq 0 ];then
                make && make install
                echo -e "\n\033[32m-------------------------------\033[0m"
                echo -e "\033[35mThe $H_FILES_DIR Server Install Success\033[0m"
        else
                echo -e "\033[35mThe $H_FILES_DIR Server Install Failed\033[0m"
                exit 0
        fi
fi

#auto install Mysql
if [[ "$1" -eq "2" ]];then
     wget -c $M_URL/$M_FILES && tar -jxvf $M_FILES && cd $M_FILES_DIR &&./configure --prefix=$M_PREFIX
        if [ $? -eq 0 ];then
                make && make install
                echo -e "\n\033[32m-------------------------------\033[0m"
                echo -e "\033[35mThe $M_FILES_DIR Server Install Success\033[0m"
        else
                echo -e "\033[35mThe $M_FILES_DIR Server Install Failed\033[0m"
                exit 0
        fi
fi
#auto install php server
if [[ "$1" -eq "3" ]];then
        wget -c $P_URL/$M_FILES && tar -jxvf $P_FILES && cd $P_FILES_DIR &&./configure --prefix=$P_PREFIX
        if [ $? -eq 0 ];then
                make && make install
                echo -e "\n\033[32m-------------------------------\033[0m"
                echo -e "\033[35mThe $P_FILES_DIR Server Install Success\033[0m"
        else
                echo -e "\033[35mThe $P_FILES_DIR Server Install Failed\033[0m"
                exit 0
        fi
fi
if [[ "$1" -eq "4" ]];then
        sed -i '/DirectoryIndex/s/index.html/index.php index.html/g' $H_PREFIX/conf/httpd.conf
        $H_PREFIX/bin/apachectl restart
        echo "addType application/x-httpd-php .php" >> $H_PREFIX/conf/httpd.conf
        IP=ifconfig eth1|grep "Bcast"|awk '{print $2}'|cut -d: -f2
        echo "You can access http://$IP/"
        cat>$H_PREFIX/htdocs/index.php <<EOF
<?php
phpinfo();
?>
EOF
fi
```



## 6.for循环语句

### 6.1.  示例1 循环seq中的数

```shell
#!/bin/bash
for i in `seq 1 15`
do
       echo -e "\033[31mThe number is $i\033[0m"
done
```

### 6.2.  示例2  0-100求和

```shell
#!/bin/bash
j=0
#for [[ i=1;i<=100;i++ ]]          #false
#for i in `seq 1 100`              #true
for ((i=1;i<=100;i++))             #true
do
        j=`expr $i + $j`
done
echo $j
```

### 6.3.  示例3 找到相关文件，批量打包

```shell
#!/bin/bash
for i in `find . -name "*.sh"`
do
        tar czf sh.tgz $i
done
```

### 6.4.  示例4 远程主机批量传输文件

```shell
#!/bin/bash
for i in `seq 100 110`
do
   echo -e "\033[33mscp -r /home/glzj/myFile/ glzj@127.0.0.1:/home/glzj/Download\033[0m"
done
```

```shell
#!/bin/bash
FILE=$*
if [ -z $FILE ];then
        echo -e "\033[34mUsage:{$0 /home|/home/glzj/test.sh|help}\033[0m"
        exit
fi
for i in "127.0.0.1 192.168.80.129"
do
        scp -r $FILE glzj@$i:/home/glzj/test2
done
```

### 6.5.  示例5 远程主机批量执行命令

```shell
#!/bin/bash
for i in "127.0.0.1 192.168.80.129"
do
    ssh -l glzj $i "ls /home" >> log.txt
done
```

## 7. while循环语句

### 7.1.  示例1 循环打印数字

```shell
#!/bin/bash
i=0
#while [[ $i -lt 10 ]]
while (($i<10))
do
    echo -e "\033[35mThe number is $i\033[0m"
    ((i++))
done
```

### 7.2.  示例2 while逐行读取某个文件

```shell
#!/bin/bash
while read line 
do
	echo -e "\033[32m$line\033[0m"
done < ifDir.sh
```

```shell
#!/bin/bash
while read addr 
do
    echo -e "\033[32mscp -r test glzj@$addr:/home/glzj\033[0m"
done < addr.txt
```

```shell
#!/bin/bash
for i in `cat addr.txt`
do
	echo -e "\033[36m$i\033[0m"
done
```

```shell
#!/bin/bash
while read i
do
    IP=`echo $i | awk '{print $2}'`
    echo -e "\033[36m$IP\033[0m"
done < addr.txt
```

## 8. Until循环

until: 为条件满足的时候，退出循环；

 ```shell
#!/bin/bash
i=10
until [[ $i -lt 0 ]]
do
    echo "$i"
    ((i--))
done
 ```



## 9. case选择

```shell
#!/bin/bash
case $1 in
    apache)
    echo -e "\033[32mYou select apache\033[0m"
    ;;
    php)
    echo -e "\033[32mYou select php\033[0m"
    ;;
    *)
    echo -e "\033[32mUsage:{$0 apache|php|help}\033[0m"
    ;;
esac
```



## 10.Select选择语句

Select一般用于选择菜单的创建，可以配合PS3来做菜单的打印输出信息

 ```shell
#!/bin/bash
PS3="Please select you excu mune:"
select i in "php" "java" "golong"
do
   echo "You select $i"
done
#########################################
[root@localhost s]# sh sele.sh
1) php
2) java
3) golong
Please select you excu mune:
 ```

```shell
#!/bin/bash
PS3="Please select your exec mume"
select i in "apache" "php"
do
    case $i in
        apache)
        echo -e "\033[32mYou select apache\033[0m"
        ;;
        php)
        echo -e "\033[32mYou select php\033[0m"
        ;;
        *)
        echo -e "\033[32mUsage:{$0 apache|php|help}\033[0m"
        ;;
    esac
done
```



## 11.函数

```shell
[ function ] funname [()]{
    action;
    [return int;]
}
```

**说明：**

1、可以带function fun() 定义，也可以直接fun() 定义,不带任何参数。 

2、参数返回，可以显示加：return 返回，如果不加，将以最后一条命令运行结果，作为返回值。return后跟数值n(0-255)，只能是int类型的数，如命令运行没错，即为0。

### 11.1第一个函数

```shell
#!/bin/bash
myfunction(){
	echo "This my first function"
}
echo "function start"
myfunction	#调用函数
echo "function end"
```

### 11.2带return的函数

```shell
#!/bin/bash
function myfun(){
    echo "This function is sum tow numbers"
    read -p "first var a :" a
    read -p "secound var b :" b
    return $(($a+$b))
}
myfun
echo "a+b=$?"
#函数返回值在调用该函数后通过 $?来获得
```

### 11.3带参数的函数

```shell
#!/bin/bash
function myfunc(){
        echo "This is 1 para: $1"
        echo "This is 2 para: $2"
        echo "This is 10 para: $10"
        echo "This is 10 para: ${10}"
        echo "This is 11 para: ${11}"
        echo "All paras numbers: $#"
        echo "This is all para: $*"
}
myfunc 1 3 4 5 6 7 8 9 12 13 14 15 16
#注意，$10 不能获取第十个参数，获取第十个参数需要${10}。当n>=10时，需要使用${n}来获取参数；
```

### 11.4函数中的一些特殊符号

| 参数处理 | 说明                                                         |
| -------- | ------------------------------------------------------------ |
| $#       | 传递到脚本的参数个数                                         |
| $*       | 以一个单字符串显示所有向脚本传递的参数                       |
| $$       | 脚本运行的当前进程ID号                                       |
| $!       | 后台运行的最后一个进程的ID号                                 |
| $@       | 与$*相同，但是使用时加引号，并在引号中返回每个参数。         |
| $-       | 显示Shell使用的当前选项，与set命令功能相同。                 |
| $?       | 显示最后命令的退出状态。0表示没有错误，其他任何值表明有错误。 |

## 12.数组

Shell 数组用括号来表示，元素用"空格"符号分割开，语法格式如下：

```shell
array_name=(value0 value1 ... valuen)
```

```shell
#!/bin/bash
soft=(
    jdk.tar.gz
    tomcat.tar.gz
    nginx.tar.gz
)
echo "This soft total is ${#soft[@]}"   #元素总数
echo "This soft total is ${#soft[*]}"
echo "This soft all is ${soft[@]}"      #所有元素列表
echo "This soft all is ${soft[*]}"
echo "This soft first is ${soft[0]}"
tar -xzf ${soft[2]} ; cd nginx; ./configure ; make install          #一般的用法
echo "Update soft two is ${soft[@]/tomcat.tar.gz/tomcat.tgz}"
echo "Delete soft two is"
unset soft[1]
echo ${soft[*]}
```







