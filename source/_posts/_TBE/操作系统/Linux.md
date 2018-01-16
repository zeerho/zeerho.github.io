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

---
#命令

##常用

- cat

> *concatenate*
> 显示文件内容。

`cat {file1} [{file2}]...`
显示一个或多个文件的内容。对于多个文件，会将内容拼接起来显示。

---
- ls

> 显示指定目录的内容，缺省参数为当前目录。

`ls -l`
显示详细的列表。
`ls -F`
显示文件类型信息。

---
- cp

> 复制文件。

`cp {file1} {file2}`
将 file1 复制到 file2。
`cp {file1} ... {file2} {dir}`
将文件复制到指定目录。

---
- mv

> 重命名或移动文件。

`mv {file1} {file2}`
将 file1 重命名为 file2。
`mv {file1} ... {file2} {dir}`
将文件移动到指定目录。

---
- touch

> 创建文件。若文件已存在则更新时间戳。

`touch {file}`

---
- rm

> 删除文件。

`rm {file}`

---
- echo

> 将它的参数显示到标准输出。

`echo {variable}`

---
- cd 

> 设置当前工作目录。当前工作目录是指进程和 shell 当前所在的工作目录。

`cd {dir}`

---
- mkdir

> 创建新目录。

`mkdir {dir}`

---
- rmdir

> 删除目录，只能删除空目录。

`rmdir {dir}`
删除指定目录。
`rm -rf {dir}`
删除指定目录及其中所有内容。

---
- grep

> 显示文件和输入流中和参数匹配的行。可识别正则表达式。

`grep -[i][v] {keyword} {file}`
-i 表示不区分大小写，-v 表示反转匹配，即显示所有不匹配的行。

---
- less

> 分屏显示文件内容。空格查看下一屏，b 键查看上一屏，q 键退出。
`less`命令是`more`命令的增强版，在一些没有`less`命令的 Unix 系统和嵌入式系统中可以使用`more`命令。

---
- pwd

> 输出当前的工作目录名。

---
- diff

> 查看两个文件之间的不同。

`diff {file1} {file2}`

---
- file

> 查看文件的格式信息。

`file {file}`

---
- find

> 查找文件。可以使用模式匹配参数，但必须给模式匹配参数加引号，因为 shell 会在运行命令前展开通配符。

`find {dir} -name {file} -print`

---
- locate

> 在系统创建的文件索引中查找文件。该索引由系统周期性地进行更新。

---
- head 和 tail

> 显示文件的前 10 行或后 10 行内容。

`head|tail [-{n}|+{n}][-f] {file}`
-n 设置显示的行数，+n 设置显示第 n 行开始的内容，-f 设置不停地读取最新内容。

---
- sort

> 将文件内的所有行按照字母顺序快速排序。-n 按照数字顺序排序那些以数字开始的行，-r 反向排序。

---
- export

> 将某个 shell 变量设置为环境变量。

`export {var}`

---
- man
- {command} --help|-h
- info {command}

> 获取在线帮助。

`man [-k] {command}`
获取指定命令的帮助信息，可指定关键词进行查找。

---
- {command} > {file}
- {command} >> {file}
- {command} | {command}
- {command} 2> {file}
- {command} > {file} 2>&1
- {command} < {file}

> `>`将执行结果重定向至文件（缺省为终端屏幕），如果文件不存在，shell 会创建新文件；如果文件存在，shell 会先清空文件内容。bash 中可以通过设置参数`set -C`来防止清空。
`>>`将执行结果重定向至文件末尾。
`|`将执行结果作为后一个命令的输入。
`2>`重定向标准错误输出。2 是由 shell 修改的流 ID，1 是标准输出，2 是标准错误输出。
`>&`将标准输出和标准错误输出重定向到同一个地方。
`<`将文件内容重定向为命令的标准输入。

---
- du

> 查看文件和目录的磁盘使用空间。

`du -sh ./*`
显示当前目录下所有文件和目录的磁盘使用情况。

##进程
- ps

> 列出所有正在运行的进程。
PID：进程 ID。
TTY：进程所在的终端设备。
STAT：进程在内存中的状态。完整状态列表参阅帮助手册。
TIME：进程占用 CPU 的总时长。
COMMAND：命令名，进程有可能将其由初始值改为其他。

`ps x`
显示当前用户运行的所有进程。
`ps ax`
显示系统当前运行的所有进程，包括其他用户的进程。
`ps u`
显示更详细的进程信息。
`ps w`
显示命令的全名，而非仅显示一行以内的内容。
`ps {PID}`
显示指定进程的信息，可用 $$ 表示当前 shell 的进程。

---
- kill

> 向进程发送一个信号。

`kill [-{SIG}] PID`
{SIG} 是具体的信号。缺省是 TERM。STOP 表示暂停，CONT 表示继续，INT 或 CTRL-C 表示中断，KILL 表示强行终止。也可使用数字代替信号名。

**KILL** 在内核层面立即终止进程，该信号不能被阻塞、处理或忽略。
**INT** 通知前台进程组终止进程。
**QUIT** 和 INT 类似, 但由 QUIT 字符(通常是Ctrl-\\)来控制。进程在因收到 QUIT 退出时会产生 core 文件, 在这个意义上类似于一个程序错误信号。
**TERM** 要求进程自己正常退出，该信号可以被阻塞或处理。

---
#特殊字符

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
#命令行按键

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
#配置

##安装配置文件
kickstart
/root/anaconda-ks.cfg

---
#问题

1. `FATAL:Could not read from Boot Medium! System Halted.`
解决：虚拟机设置-存储-删除多余的控制器
2. 参数风格
解决：
    1. 参数用一横的说明后面的参数是字符形式。
    2. 参数用两横的说明后面的参数是单词形式。
    3. 参数前有横的是 System V 风格。
    4. 参数前没有横的是 BSD 风格。
