## 13.shell编程四剑客

### 13.1.sed：替换

```shell
#替换文件中的字符
#使用vi打开文件替换：
:%s/oldstring/newstring/g
:%s/oldstring/newstring/
```

```shell
#不打开文件，使用sed修改：
$ sed 's/oldstring/newstring/g' filename       #预修改，不会真的去修改
$ sed -i 's/oldstring/newstring/g' filename    #加参数-i，修改文件
$ sed 's/^/& /g' test.txt          #给test.txt文件中的每行的开头加上空格
$ sed 's/$/& id/g' test.txt        #给每行的结尾加上 空格id
$ sed '/wgk/a ###' test.txt        #给wgk行之后加上一行，输入###
$ sed '/wgk/i ###' test.txt        #给wgk行之前加上一行，输入###
$ sed '/wgk/p' test.txt 		  #print包含wgk的行
$ sed -n '1p' test.txt             #print第1行
$ sed -n '1,5p' test.txt 		  #print第1-5行
#查看文件，把空格改成换行，去掉空行，排序由大到小，取第一行和最后一行
$ cat number.txt |sed 's/ /\n/g' |grep -v "^$" |sort -nr | sed -n '1p;$p'
```

### 13.2 grep：行筛选

```shell
$ cat number.txt |grep “45”     #打印包含45的行
$ cat number.txt |grep -v “45”  #打印不包含45的行
$ grep “11$”               		#匹配以11结尾的行
$ grep “^10”               		#匹配以10开头的行
$ cat test.txt |grep -E “([0-9]{1,3}.){3}[0-9]{1,3}”     #匹配ip地址
$ egrep “11|10” test.txt         #匹配11或者10的行，egrep即grep -E
```



### 13.3 awk：列筛选

```shell
this is my file              #test.txt
####################################
$ cat test.txt |awk '{print $1}'            #this
$ cat test.txt |awk '{print $NF}'          #取最后一列
#etc/passwd文件，把 : 变成空格，取第一列的元素
$ cat /etc/passwd |sed 's/:/ /g' |awk '{print $1}'       
$ cat /etc/passwd |awk -F: '{print $1}'          #以 : 分割，第1列
$ ifconfig ens33 |grep “netmask” |awk '{print $2}'   #获取ip
$ df -h |grep "/$" |awk '{print $5}' |sed 's/%/ /g'      #获取根路径文件利用率
$ cat test.txt|awk '{print "id: "$2}'                 #给筛选出的内容前添加”id: ”
```

### 13.4 find：查找文件/目录

```shell
$ find path -name "*.txt"                    #寻找path路径下，.txt文件
$ find . -maxdepth 1 -name “test.txt”        #寻找当前目录下，第一级中的，名为test.txt文件
$ find . -maxdepth 1 -type f -name “*.txt”   #寻找类型为file，名字为 .txt 的文件
$ find . -maxdepth 1 -type f -name “*.txt” -mtime +30      #寻找30天前的文件
$ find . -maxdepth 1 -type f -name “*.txt” -mtime -1       #寻找1天内的文件
$ find . -maxdepth 1 -type f -name “*.txt” -mtime -1 -exec rm -rf {} \;   #delete找到的文件
$ find . -maxdepth 1 -name “*.txt” -mtime -1 -exec cp {} /tmp/ \;         #copy找到的文件到/tmp/
$ find . -maxdepth 1 -name “*.txt” |xargs rm -rf {} \;     #delete找到的文件，xargs不能copy
$ find . -maxdepth 1 -size +50M -type f                    #查找大于50M的文件
$ find . -maxdepth 1 -size +50M -type f -exec cp {} /download/ \; 
#把大于50M的文件copy到/download/中
```



## 14.备份

完整备份：每周日进行完整备份

增量备份：其余每天为增量备份

```shell
#使用tar命令来备份；
#全备份：
$ tar -g /tmp/snapshot -czvf /tmp/2019_full_system_data.tar.gz /data/sh/
#增量备份：
$ tar -g /tmp/snapshot -czvf /tmp/2019_add01_system_data.tar.gz /data/sh/
```

```shell
#查询日期
$ date
$ echo `date +%d`
$ echo `date +%u`
```

```shell
#自动备份脚本
#!/bin/sh
#Automatic Backup Linux System Files
#Author nyfblack
#Define Variable
SOURCE_DIR=(
	$*
)
TARGET_DIR=/data/backup/
YEAR=`date +%y`
MONTH=`date +%m`
DAY=`date +%d`
WEEK=`date +%u`
A_NAME=`date +%H%M`
FILES=${A_NAME}_system_backup.tgz
CODE=$?
if [ -z "$*" ];then
	echo -e "\033[32mUSAGE:\nPlease Enter Your Backup Files or Directories\n------------------------------------\n\nUsage:{ $0 /boot /etc}\033[0m"
	exit
fi
#Determine whether the Target Directory Exists
if [ ! -d $TARGET_DIR/$YEAR/$MONTH/$DAT ];then
	mkdir -p $TARGET_DIR/$YEAR/$MONTH/$DAT
	echo -e "\033[32mThe $TARGET_DIR Created Successfully! \033[0m"
fi
#EXEC Full_Backup Function Command
Full_Backup()
{
    if [ "$WEEK" -eq "7" ];then
    	rm -rf $TARGET_DIR/snapshot
    	cd $TARGET_DIR/$YEAR/$MONTH/$DAY ;tar -g $TARGET_DIR/snapshot -czvf $FILES ${SOURCE_DIR[@]} 
    	[ "$CODE" == "0" ]&&echo -e "---------------------------\n\033[32mThese Full_Backup System Files Backup Successfully ! 033[0m"
    fi
}
#Perform incremental BACKUP Function Command
Add_Backup()
{
    if [ $WEEK -NE "7" ];then
    	cd $TARGET_DIR/$YEAR/$MONTH/$DAY ;tar -g $TARGET_DIR/snapshot -czvf $A_NAME$FILES ${SOURCE_DIR[@]} 
    	[ "$CODE" == "0" ]&&echo -e "---------------------------\n\033[32mThese ADD_Backup System Files $TARGET_DIR/$YEAR/$MONTH/$DAY/${YEAR}_$A_NAME$FILES Backup Successfully ! 033[0m"
    fi
}
sleep 3
Full_Backup;Add_Backup
```



## 15.防止恶意攻击服务器

```shell
#tail命令：查看文件syslog中的后100行
$ tail -n 100 /var/log/syslog 
```

```shell
#查找输入登录密码错误的ip
$ tail -n 100 /var/log/secure | grep "Failed password" |awk '{print $11}'|sort|uniq -c |sort -nr
################运行结果如下################
  7 192.168.33.14
$ tail -n 100 /var/log/secure | grep "Failed password" |awk '{print $11}'|sort|uniq -c |sort -nr |awk '{print $1}'
################运行结果如下################
  7
```

```shell
$ vi /etc/sysconfig/iptables
#########################################
-A INPUT -s 192.168.33.14 -j DROP
-A INPUT -m state --state NEW -m tcp -p tcp --dport 22 -j ACCEPT
-A RH-Firewall-1-INPUT -m state --state NEW -m tcp -p tcp --dport 23306 -j ACCEPT
#########################################
$ /etc/init.d/iptables restart
```

```shell
#!/bin/sh
#auto drop ssh failed IP address
#定义变量
SEC_FILE=/var/log/secure
#如下为截取secure文件中的恶意ip 远程登录22端口，大于4此密码错误，就写入防火墙，禁止此ip登录服务器22端口
IP_ADDR=`tail -n 1000 /var/log/secure | grep "Failed password"|egrep -o "([0-9]{1,3}\.{3}[0-9]{1,3})"|uniq -c |sort -nr|awk ' $1>=4 {print $2}'`
IPTABLE_CONF=/etc/sysconfig/iptables
echo
cat <<EOF
++++++++++++++++welcome to use ssh login drop failed ip+++++++++++++++++++++++
EOF

for i in `echo $IP_ADDR`
do
	#查看iptables配置文件是否含有提取的ip信息
	cat $IPTABLE_CONF |grep $1 >/dev/null
if [ $? -ne 0 ];then
	#判断iptables配置文件里面是否存在已拒绝的ip，如果不存在，就不再添加相应条目
	#sed a参数的意思是匹配之后加入行
	sed -i "/lo/a -A INPUT -s $1 -m state --state NEW -m tcp -p tcp --dport 22 -j DROP" $IPTABLE_CONF
else
	#如果存在，就打印信息
	echo "This is $1 is exist in iptables,please exit ....."
fi
done
#最后重启iptables生效
/etc/init.d/iptables restart
```



## 16.网站自动化部署

- 同步文件到多个服务器：

```shell
#!/bin/sh
#Auto Change Server Files
#SRC=/etc/
if [ ! -f ip.txt ];then
	echo "Please Create ip.txt Files,The ip.txt content as follows:"cat <<EOF
192.168.149.128
192.168.149.128
EOF
	exit
fi

if [ -z "$1" ];then
	echo -e "\033[31mUsage:$0 command ,example{Src_Files|Src_Dir Des_dir} \033[0m"
	exit
fi
count=`cat ip.txt |wc -l`
rm -rf ip.txt.swp
i=0
while ((i<$count))
do
    i=`expr $i+1`
    sed "${i}s/^/&${i} /g" ip.txt>>ip.txt.swp
    IP=`awk -v I="$i" '{if(I==$1)print $2}' ip.txt.swp`
    scp -r $1 root@${IP}:$2
    #rsync -ap --delete $1 root@${IP}:$2
done
```

- 批量远程服务器执行命令：

```shell
#!/bin/sh
#Auto Change Server Files
#SRC=/etc/
if [ ! -f ip.txt ];then
	echo "Please Create ip.txt Files,The ip.txt content as follows:"cat <<EOF
192.168.149.128
192.168.149.128
EOF
	exit
fi

if [ -z "$*" ];then
	echo -e "\033[31mUsage:$0 command ,example{rm /tmp/test.txt |mkdir /tmp/test.txt} \033[0m"
	exit
fi
count=`cat ip.txt |wc -l`
rm -rf ip.txt.swp
i=0
while ((i<$count))
do
	i=`expr $i+1`
	sed "${i}s/^/&${i} /g" ip.txt>>ip.txt.swp
	IP=`awk -v I="$i" '{if(I==$1)print $2}' ip.txt.swp`
	ssh -q -l root $IP "$*;echo -e '------------------\nThe $IP Exec command: $* success!';sleep2"
done
```

- Rsync+ssh自动部署
