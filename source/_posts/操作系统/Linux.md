---
title: Linux
date: 2017-01-01 09:00:00
tags: [操作系统]
---

# 开机流程

1. BIOS：开机主动执行的固件，会识别用户设定的用于启动系统的存储设备。
2. MBR：用于启动系统的存储设备的第一个扇区（或 LBA）内的主要启动记录区块，内含开机管理程序。
3. 开机管理程序（boot loader）：用于读取并执行系统核心文件。
    1. 提供选单：用户可以选择不同的开机项目。
    2. 载入核心文件：直接指向开机程序的地址分段来启动操作系统。
    3. 转交其他 loader：将开机管理功能转交给其他 loader 负责。每个分区的启动扇区里都可以放入一个 boot loader，该 loader 只认识本分区的系统核心文件和其他分区的 loader。
4. 核心文件：启动操作系统的各种功能

一些发行版使用一个表来管理开机自启的进程，该表通常位于 /etc/inittab 文件中。另一些发行版则采用 /etc/init.d/ 目录，将开机时启动或停止某个应用的脚本放到该目录下，这些脚本通过 /etc/rcX.d/ 目录下的符号链接（链接到 /etc/init.d/ 目录中的启动脚本）启动，其中 X 代表运行级（run level）。

# 环境变量

`$ export VAR="test"` 临时变量，重启系统后失效。

/etc/environment 是整个系统的环境，/etc/profile 是所有用户的环境。

login shell：通过终端凭借用户名和密码登录的动作。
non-login shell：在图形界面启动一个 terminal，或者执行 /bin/bash，/usr/bin/bash。

|文件           |非交互+登录|交互+登录|交互+非登录|非交互+非登录|
|:-------------:|:---------:|:-------:|:---------:|:-----------:|
|/etc/profile   |加载       |加载     |           |             |
|/etc/bashrc    |加载       |加载     |           |             |
|~/.bash_profile|加载       |加载     |           |             |
|~/.bashrc      |加载       |加载     |加载       |             |
|BASH_ENV       |           |         |           |加载         |

可以修改 shell 来模拟交互登录式的动作：

```sh
#!/path/to/my_sh -ilex
#e 一旦出错就退出当前 shell
#i 交互式
#l 登录式
#x 显示所执行的每条命令
```

login shell 读取文件的顺序是：

1. /etc/profile
2. 读到任意一个就忽略后面的
    1. ~/.bash_profile
    2. ~/.bash_login
    3. ~/.profile

non-login shell 每次启动 shell 都会读取：

1. ~/.bashrc

---
# 挂载/卸载

`df -h` 查看挂载情况
`umount {file_system}` 卸载，终端中挂载消失，UI 中挂载仍在
`mount {file_system}` 挂载 
`eject` 弹出，UI 中挂载也消失，且无法直接再用`mount`命令挂载回来

---
# 软件安装

## JDK

1. 安装
    1. 选择要安装 jdk 的位置，如 /usr/ 目录下，新建文件夹 java(`sudo mkdir /usr/java`)
    2. 将文件 jdk-7u40-linux-i586.tar.gz 移动到 /usr/java
    3. 解压：`sudo tar -zxvf jdk-7u40-linux-i586.tar.gz`
    4. 删除 jdk-7u40-linux-i586.tar.gz
2. 配置
    1. 打开 /home/{user}/.bashrc，加入：`source /etc/profile`
    2. 打开 /etc/profile（`sudo vim /etc/profile`）
    在最后面添加如下内容，这样使得每次进入该用户的 bash 时都会使添加的环境变量生效：
```
JAVA_HOME=/usr/java/jdk1.7.0_40
PATH=$JAVA_HOME/bin:$PATH
export JAVA_HOME PATH
```

验证是否安装成功：`java -version`

## Chrome

1. `sudo wget http://www.linuxidc.com/files/repo/google-chrome.list -P /etc/apt/sources.list.d/`
将下载源加入系统的源列表
2. `wget -q -O - https://dl.google.com/linux/linux_signing_key.pub  | sudo apt-key add -`
导入谷歌软件的公钥，用于后续步骤对下载软件进行认证
3. `sudo apt-get update`
对当前系统可用的更新列表进行更新
4. `sudo apt-get install google-chrome-stable`
安装 chrome 稳定版
5. `/usr/bin/google-chrome-stable`
  启动 chrome

## mysql

1. 下载 RPM包

   `wget https://dev.mysql.com/get/mysql57-community-release-el7-9.noarch.rpm`

2. 安装 RPM 包

   `yum localinstall mysql57-community-release-el7-9.noarch`

3. 安装 mysql

   `yum install mysql-community-server`

4. 查看自动生成的随机密码

   `grep 'temporary password' /var/log/mysqld.log`

5. 启动、查看 mysql service

   `service mysqld start`

   `service mysqld status`

6. 安全相关设置

   `mysql_secure_installation`

7. 登录成功后修改密码强度限制（测试环境）

   1. `set global validate_password_policy=0;`
   2. `set global validate_password_length=4;`
   3. `set password=password('root')` 测试环境换个简单的密码

## erlang

**源码编译**

1. 下源码。从官网或 github 下：
    - `wget http://www.erlang.org/download/otp_src_20.3.tar.gz`
    - `git clone https://github.com/erlang/otp.git`
2. 解压 `tar -zxvf ~/Downloads/otp_src_20.3.tar.gz`
3. 配置
    - 如果是从 github 上下载的源码，要先生成配置脚本 `./otp_build autoconf`
    - 配置中会检查必需的工具。
        - GNU `make`
        - Compiler -- GNU C Compiler, gcc or the C compiler frontend for LLVM, clang.
        - Perl 5
        - GNU `m4`
        - `ncurses`, `termcap`, or `termlib` -- The development headers and libraries are needed, often known as ncurses-devel. Use --without-termcap to build without any of these libraries. Note that in this case only the old shell (without any line editing) can be used.
        - `sed`
    - `cd otp_src_20.3`
    - 配置自定义的安装目录 `./configure --prefix=/opt/erlang`
4. 编译 `make`
5. 安装 `make install`
6. 设置环境变量 `vi /etc/profile`（`whereis erl` 查看安装目录）
```shell
ERL_HOME=/usr/local/erlang
PATH=$ERL_HOME/bin:$PATH
export ERL_HOME PATH
```

**yum 安装**

1. 安装 epel `yum -y install http://dl.fedoraproject.org/pub/epel/7/x86_64/Packages/e/epel-release-7-11.noarch.rpm`
2. 更新 erlang 仓库 `﻿yum -y install  http://packages.erlang-solutions.com/erlang-solutions-1.0-1.noarch.rpm`
3. 安装 erlang `yum install erlang`

## rabbitmq

1. 下包。`wget https://dl.bintray.com/rabbitmq/all/rabbitmq-server/3.7.4/rabbitmq-server-3.7.4-1.el7.noarch.rpm`
2. 可能要安装 socat：`yum -i socat`
3. 安装。`rpm -i rabbitmq-server-3.7.4-1.el7.noarch.rpm`
4. 如果不是用这种包管理的方式安装的话可能要手动创建几个文件夹
    - `mkdir -p /var/log/rabbitmq`
    - `mkdir -p /var/log/rabbitmq/mnesia/rabbit`
5. 启动服务 `sbin/rabbitmq-server`

# 包管理

## apt-get

`apt-get purge` `apt-get remove --purge`

删除软件包和配置文件，保留依赖包。

`apt-get remove`

删除软件包，保留依赖包和配置文件。

`apt-get autoremove`

删除不需要的依赖包。

`apt-get autoclean`

APT 的底层包是 dpkg，dpkg 会把 .deb 文件放在 /var/cache/apt/archives 中。

## yum

`yum search {keyword}`

根据关键字搜索软件包

`yum list`

列出可安装包

`yum list [updates|installed|{name}|extras]`

列出可更新包|已安装包|指定包|所有已安装但不在 Yum Repository 中的包

`yum info`

列出所有软件包信息

`yum info [updates|installed|{name}|extras]`

列出可更新包|已安装包|指定包|所有已安装但不在 Yum Repository 中的包的信息

`yum provides {name}`

列出软件包提供哪些文件

 

`yum clean packages`

清除缓存目录(/var/cache/yum)下的软件包

`yum clean headers`

清除缓存目录(/var/cache/yum)下的 headers

`yum clean oldheaders`

清除缓存目录(/var/cache/yum)下旧的 headers

`yum clean, yum clean all` (= `yum clean packages`;` yum clean oldheaders`)

清除缓存目录(/var/cache/yum)下的软件包及旧的headers



`yum update` 升级系统

`yum install {name}` 安装指定软件包

`yum update {name}` 升级指定软件包

`yum remove {name}` 卸载指定软件

`yum grouplist` 查看系统中已经安装的和可用的软件组，可用的可以安装

` yum grooupinstall {name}` 安装上一个命令显示的可用的软件组中的一个

` yum grooupupdate {name}` 更新指定软件组的软件包

`yum grooupremove {name}` 卸载指定软件组中的软件包

`yum deplist {name}` 查询指定软件包的依赖关系

` yum list yum\* ` 列出所有以yum开头的软件包

`yum localinstall {name}` 从硬盘安装rpm包并使用yum解决依赖

# 命令

## 常用

**cat**

concatenate 显示文件内容。

`cat {file1} [{file2}]...`
显示一个或多个文件的内容。对于多个文件，会将内容拼接起来显示。

---
**ls**

显示指定目录的内容，缺省参数为当前目录。

`-a` `--all` 显示 dot 文件。
`-A` `--almost-all` 显示 dot 文件，除了 . 和 ..。
`--block-size=[K|M|G|T|P|E|Z|Y]` 定义显示文件大小的单位。
`-c` 结合 `-lt`：显示最近修改时间，并按其排序；结合 `-l`：显示最近修改时间，按文件名排序；否则按最近修改时间排序。
`-d` `--directory` 显示文件夹本身，而不是它的内容。
`-F` 显示文件类型信息。
`-hide={pattern}` 不显示匹配的文件。
`-i` `--inode` 显示文件的 inode 号。
`-I` `--ignore={pattern}` 不显示匹配的文件。
`-l` 显示详细的列表。
`-L` 当显示符号链接的文件信息时，显示符号链接指向的对象而非链接本身的信息。
`-n` `--numeric-uid-gid` 类似 `-l`，但显示用户和用户组的 ID。
`-o` 类似 `-l`，但不显示组信息。
`-r` 反向排序。
`-R` `--recursive` 递归显示子目录。
`-S` 按文件大小排序。
`-t` 按修改时间排序，由新到旧。
`-u` 结合 `-lt`：显示最近打开时间并按其排序；结合 `-l`：显示最近打开时间，并按文件名排序；否则，按最近打开时间排序。
`-U` 不排序。
`-w` `--width={cols}` 定义屏幕宽度。
`-X` 按扩展名的字母顺序排序。

---
**cp**

复制文件。

`cp {file1} {file2}`
将 file1 复制到 file2。
`cp {file1} ... {file2} {dir}`
将文件复制到指定目录。

---
**mv**

重命名或移动文件。

`mv {file1} {file2}`
将 file1 重命名为 file2。
`mv {file1} ... {file2} {dir}`
将文件移动到指定目录。

---
**touch**

> 创建文件。若文件已存在则更新时间戳。

`touch {file}`

---
**rm**

> 删除文件。

`rm {file}`

---
**echo**

将它的参数显示到标准输出。

`echo {variable}`

---
**cd** 

> 设置当前工作目录。当前工作目录是指进程和 shell 当前所在的工作目录。

`cd {dir}`

---
**mkdir**

> 创建新目录。

`mkdir {dir}`

---
**rmdir**

> 删除目录，只能删除空目录。

`rmdir {dir}`
删除指定目录。
`rm -rf {dir}`
删除指定目录及其中所有内容。

---
**grep**

> 显示文件和输入流中和参数匹配的行。可识别正则表达式。

`grep -[i][v] {keyword} {file}`
-i 表示不区分大小写，-v 表示反转匹配，即显示所有不匹配的行。

---
**less**

> 分屏显示文件内容。空格查看下一屏，b 键查看上一屏，q 键退出。
`less`命令是`more`命令的增强版，在一些没有`less`命令的 Unix 系统和嵌入式系统中可以使用`more`命令。

---
**pwd**

> 输出当前的工作目录名。

---
**diff**

> 查看两个文件之间的不同。

`diff {file1} {file2}`

---
**file**

> 查看文件的格式信息。

`file {file}`

---
**find**

> 查找文件。可以使用模式匹配参数，但必须给模式匹配参数加引号，因为 shell 会在运行命令前展开通配符。

`find {dir} -name {file} -print`

---
**locate**

> 在系统创建的文件索引中查找文件。该索引由系统周期性地进行更新。

---
**head** 和 **tail**

> 显示文件的前 10 行或后 10 行内容。

`head|tail [-{n}|+{n}][-f] {file}`
-n 设置显示的行数，+n 设置显示第 n 行开始的内容，-f 设置不停地读取最新内容。

---
**sort**

> 将文件内的所有行按照字母顺序快速排序。-n 按照数字顺序排序那些以数字开始的行，-r 反向排序。

---
**export**

> 将某个 shell 变量设置为环境变量。

`export {var}`

---
**man**

**{command} --help|-h**

**info {command}**

> 获取在线帮助。

`man [-k] {command}`
获取指定命令的帮助信息，可指定关键词进行查找。

---
**{command} > {file}**

**{command} >> {file}**

**{command} | {command}**

**{command} 2> {file}**

**{command} > {file} 2>&1**

**{command} < {file}**

> `>`将执行结果重定向至文件（缺省为终端屏幕），如果文件不存在，shell 会创建新文件；如果文件存在，shell 会先清空文件内容。bash 中可以通过设置参数`set -C`来防止清空。
`>>`将执行结果重定向至文件末尾。
`|`将执行结果作为后一个命令的输入。
`2>`重定向标准错误输出。2 是由 shell 修改的流 ID，1 是标准输出，2 是标准错误输出。
`>&`将标准输出和标准错误输出重定向到同一个地方。
`<`将文件内容重定向为命令的标准输入。

---
**du**

> 查看文件和目录的磁盘使用空间。

`du -sh ./*`
显示当前目录下所有文件和目录的磁盘使用情况。

## 进程

**ps**

列出所有正在运行的进程。

PID：进程 ID。
TTY：进程所在的终端设备。
STAT：进程在内存中的状态。完整状态列表参阅帮助手册。
TIME：进程占用 CPU 的总时长。
COMMAND：命令名，进程有可能将其由初始值改为其他。

`ps x` 显示当前用户运行的所有进程。
`ps ax` 显示系统当前运行的所有进程，包括其他用户的进程。
`ps u` 显示更详细的进程信息。
`ps w` 显示命令的全名，而非仅显示一行以内的内容。
`ps {PID}` 显示指定进程的信息，可用 $$ 表示当前 shell 的进程。

---
**kill**

向进程发送一个信号。

`kill [-{SIG}] PID`
{SIG} 是具体的信号。缺省是 TERM。STOP 表示暂停，CONT 表示继续，INT 或 CTRL-C 表示中断，KILL 表示强行终止。也可使用数字代替信号名。

**KILL** 在内核层面立即终止进程，该信号不能被阻塞、处理或忽略。
**INT** 通知前台进程组终止进程。
**QUIT** 和 INT 类似, 但由 QUIT 字符(通常是Ctrl-\\)来控制。进程在因收到 QUIT 退出时会产生 core 文件, 在这个意义上类似于一个程序错误信号。
**TERM** 要求进程自己正常退出，该信号可以被阻塞或处理。

---
# 特殊字符

|字符|名称|用途|
|:--|:--|:--|
|*|星号|正则表达式，通用字符|
|.|句点|当前目录，文件/主机名的分隔符|
|!|感叹号|逻辑非运算符，命令历史|
|\||管道|命令管道|
|/|斜线|命令分隔符，搜索命令|
|\ |反斜线|常量，宏（非目录）|
|$|美元符号|变量符号，行尾|
|'|单引号|字符串常量|
|`|反引号|命令替换|
|"|双引号|半字符串常量|
|^|脱字符|逻辑非运算，行头|
|~|波浪字符|逻辑非运算符，目录快捷方式|
|#|井号|注释，预处理，替换|
|[]|方括号|范围|
|{}|大括号|声明块，范围|
|_|下划线|空格的简单替代|
控制键 CTRL 通常用 ^ 来表示。

---
# 命令行按键

|按键|操作|
|:--|:--|
|CTRL-B|左移光标|
|CTRL-F|右移光标|
|CTRL-P|查看上一条命令（或上移光标）|
|CTRL-N|查看下一条命令（或下移光标）|
|CTRL-A|移动光标至行首|
|CTRL-E|移动光标至行尾|
|CTRL-W|删除前一个词|
|CTRL-U|删除从光标至行首的内容|
|CTRL-K|删除从光标至行尾的内容|
|CTRL-Y|粘贴已删除的文本|

---
# 配置

## 安装配置文件
kickstart
/root/anaconda-ks.cfg

---
# 问题

1. `FATAL:Could not read from Boot Medium! System Halted.`
  解决：虚拟机设置-存储-删除多余的控制器
2. 参数风格
  解决：
      1. 参数用一横的说明后面的参数是字符形式。
      2. 参数用两横的说明后面的参数是单词形式。
      3. 参数前有横的是 System V 风格。
      4. 参数前没有横的是 BSD 风格。
