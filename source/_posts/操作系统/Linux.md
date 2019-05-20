---
title: Linux
date: 2017-01-01 09:00:00
tags: [操作系统]
---

# 系统安装

## centos7

常用分区计划：

|挂载点 |大小    |设备类型          |文件系统|
|:------|:-------|:-----------------|:-------|
|`/boot`|1024M   |standard Partition|xfs     |
|`swap` |内存容量|LVM               |swap    |
|`/`    |剩余    |LVM               |xfs     |

安装前强制使用 GPT 分区表。

1. 光标移至 `Install CentOS 7`
2. 按 Tab 键，在下方输入参数
3. `vmlinuz initrd=initrd.img inst.stage2=hd:LABEL=CentOS\x207\x20x86_64 rd.live.check quiet inst.gpt` 关键是 `inst.gpt`。

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

**login shell 读取配置的顺序**

1. /etc/profile
  - -> /etc/inputrc
  - -> /etc/profile.d/*.sh -> /etc/sysconfig/i18n
2. 读到任意一个就忽略后面的（注意：调用链括号部分跟上面有重复）
  1. ~/.bash_profile -> ~/.bashrc -> /etc/bashrc (-> /etc/profile.d/*.sh -> /etc/sysconfig/i18n)
  2. ~/.bash_login
  3. ~/.profile

**non-login shell 读取配置的顺序**

1. ~/.bashrc -> /etc/bashrc -> /etc/profile.d/*.sh -> /etc/sysconfig/i18n

**总结：**两者的交点是 ~/.bashrc（用户级）和 /etc/bashrc（系统级）。

## /etc/profile 内容解析

```sh
 # 根据 UID 决定 PATH 变量要不要含有 sbin 的系统指令目录
pathmunge () {
    case ":${PATH}:" in
        *:"$1":*)
            ;;
        *)
            if [ "$2" = "after" ] ; then
                PATH=$PATH:$1
            else
                PATH=$1:$PATH
            fi
    esac
}


 # 根据用户的账号设置此变量内容
if [ -x /usr/bin/id ]; then
    if [ -z "$EUID" ]; then
        # ksh workaround
        EUID=`/usr/bin/id -u`
        UID=`/usr/bin/id -ru`
    fi
    USER="`/usr/bin/id -un`"
    LOGNAME=$USER
    MAIL="/var/spool/mail/$USER"
fi

 # Path manipulation
 # 调用前面的 `pathmunge` 函数
if [ "$EUID" = "0" ]; then
    pathmunge /usr/sbin
    pathmunge /usr/local/sbin
else
    pathmunge /usr/local/sbin after
    pathmunge /usr/sbin after
fi

HOSTNAME=`/usr/bin/hostname 2>/dev/null`

 # 命令历史纪录相关
HISTSIZE=1000
if [ "$HISTCONTROL" = "ignorespace" ] ; then
    export HISTCONTROL=ignoreboth
else
    export HISTCONTROL=ignoredups
fi

export PATH USER LOGNAME MAIL HOSTNAME HISTSIZE HISTCONTROL

 # By default, we want umask to get set. This sets it for login shell
 # Current threshold for system reserved uid/gids is 200
 # You could check uidgid reservation validity in
 # /usr/share/doc/setup-*/uidgid file
if [ $UID -gt 199 ] && [ "`/usr/bin/id -gn`" = "`/usr/bin/id -un`" ]; then
    umask 002
else
    umask 022
fi

 # 调用其他配置
for i in /etc/profile.d/*.sh /etc/profile.d/sh.local ; do
    if [ -r "$i" ]; then
        if [ "${-#*i}" != "$-" ]; then
            . "$i"
        else
            . "$i" >/dev/null
        fi
    fi
done

unset i
unset -f pathmunge
```

# 挂载/卸载

`df -h` 查看挂载情况
`umount {file_system}` 卸载，终端中挂载消失，UI 中挂载仍在
`mount {file_system}` 挂载
`eject` 弹出，UI 中挂载也消失，且无法直接再用`mount`命令挂载回来

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
2. 更新 erlang 仓库 `yum -y install  http://packages.erlang-solutions.com/erlang-solutions-1.0-1.noarch.rpm`
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
`yum grooupinstall {name}` 安装上一个命令显示的可用的软件组中的一个
`yum grooupupdate {name}` 更新指定软件组的软件包
`yum grooupremove {name}` 卸载指定软件组中的软件包
`yum deplist {name}` 查询指定软件包的依赖关系
`yum list yum\* ` 列出所有以yum开头的软件包
`yum localinstall {name}` 从硬盘安装rpm包并使用yum解决依赖

## 配置阿里云源

**ubuntu**

源文件是 `/etc/apt/sources.list`，记录了 apt 所用的包仓库位置。类似的还有 `/etc/apt/sources.list.d/` 下的各种 `.list` 文件。

1. 备份。`sudo cp /etc/apt/sources.list /etc/apt/sources.list.bak`
2. 查看系统版本信息 `lsb_release -c`。冒号后面是系统代号，后面要用。
3. 编辑 `sources.list` 文件，将其中内容全删后粘贴以下内容。其中的 `bionic` 是系统代号，要根据实际情况替换。
```
deb http://mirrors.aliyun.com/ubuntu/ bionic main restricted universe multiverse

deb-src http://mirrors.aliyun.com/ubuntu/ bionic main restricted universe multiverse

deb http://mirrors.aliyun.com/ubuntu/ bionic-security main restricted universe multiverse

deb-src http://mirrors.aliyun.com/ubuntu/ bionic-security main restricted universe multiverse

deb http://mirrors.aliyun.com/ubuntu/ bionic-updates main restricted universe multiverse

deb-src http://mirrors.aliyun.com/ubuntu/ bionic-updates main restricted universe multiverse

deb http://mirrors.aliyun.com/ubuntu/ bionic-backports main restricted universe multiverse

deb-src http://mirrors.aliyun.com/ubuntu/ bionic-backports main restricted universe multiverse

deb http://mirrors.aliyun.com/ubuntu/ bionic-proposed main restricted universe multiverse

deb-src http://mirrors.aliyun.com/ubuntu/ bionic-proposed main restricted universe multiverse
```
4. 更新软件列表 `sudo apt update`

`sources.list` 文件内容是以下格式。后面的 component 是对软件包的分类。Ubuntu 下是 main、restricted、universe、multiverse。

```
deb http://{host}/{debian-distribution} component1 component2 component3
deb-src http://{host}/{debian-distribution} component1 component2 component3
```

**centos**

1、用wget下载repo文件

`wget  http://mirrors.aliyun.com/repo/Centos-7.repo`

如果wget命令不生效，说明还没有安装wget工具，输入`yum -y install wget` 回车进行安装。

2、移至源目录

`mv ./Centos-7.repo /etc/yum.repos.d/`

3、备份系统原来的repo文件

`mv CentOs-Base.repo CentOs-Base.repo.bak`

4、替换系统原理的repo文件

`mv Centos-7.repo CentOs-Base.repo`

5、执行yum源更新命令

`yum clean all`
`yum makecache`
~~`yum update`~~

# 用户/用户组管理

管理用户和用户组的命令基本都是对 `/etc/passwd`、`/etc/group` 和 `/etc/shadow` 进行操作。

## 用户管理

**新增用户**

`useradd [-D] [opts] {newUser}`

- `-D` 变更预设值。
- `-c {comment}` 备注，会保存在 `/etc/passwd` 的备注栏中。
- `-d {loginDir}` 用户登录时的初始目录。
- `-e {effectiveTime}` 账号的有效期限。
- `-f {effectiveDays}` 密码过期后多少天关闭账号。
- `-g {mainGroup}` 主用户组，缺省与用户同名。
- `-G {supplementaryGroup}` 附加组。
- `-m|-M` 自动建立/不建立用户的登录目录。
- `-n` 取消建立与用户同名的组。
- `-r` 建立系统账号。
- `-s {shell}` 用户登录后使用的 shell。
- `-u {uid}` 指定用户 ID。

**关于 `adduser` 和 `useradd`**

Ubuntu 中 `useradd` 是一个二进制程序，`adduser` 是一个脚本，是对前者的封装；CentOS 中，`adduser` 是另一个的符号链接。 **一般使用 `useradd`，它在不同发行版中都一样。**

**修改用户信息**
`usermod [-LU] [opts] {user}`

- `-L|-U` 锁定密码/解锁密码。
- `-c {comment}` 备注，会保存在 `/etc/passwd` 的备注栏中。
- `-d {loginDir}` 用户登录时的初始目录。
- `-e {effectiveTime}` 账号的有效期限。
- `-f {effectiveDays}` 密码过期后多少天关闭账号。
- `-g {mainGroup}` 主用户组，缺省与用户同名。
- `-G {supplementaryGroup}` 附加组，会覆盖已有的附加组。搭配 `-a` 可以新增而不覆盖。
- `l` 修改用户名。
- `-s {shell}` 用户登录后使用的 shell。
- `-u {uid}` 指定用户 ID。

**删除用户**

`userdel [-r] {user}`

- `-r` 连同用户目录一起删除。

**修改密码**

`passwd [opts] {user}`

- `-l` 锁定用户，禁止登录。
- `-u` 接触锁定。
- `--stdin` 允许通过标准输入修改密码，比如 `echo "abc" | passwd --stdin tom`。
- `-d` 可用空密码登录。
- `-e` 强制该用户下次登录时修改密码。
- `-S` 显示密码是否被锁定，以及密码所采用的加密算法名称。

## 用户组管理

**查看用户所属的组**

`groups [user]` 若不指定用户则为当前用户。

**创建用户组**

`groupadd [-r] {group}`

- `[-r]` 创建系统用户组，即 GID 小于 500。

**修改用户组**

- `groupmod -n {new} {old}` 修改用户组名，不会改变 GID。
- `groupmod -g {newGID} {group}` 修改 GID。

**删除用户组**

`groupdel {group}`

若该用户组是某用户的主用户组，则必须先删除该用户，在删除该用户组。

## 用户和用户组之间的操作

**添加用户到用户组/从用户组中删除用户**

- `gpasswd -a {user} -g {group}` 将指定用户添加到指定组。
- `gpasswd -d {user} -g {group}` 从指定组中删除指定用户。

**修改**

`groupmems [option] [action]`

- option
  - `-g` 更改为指定组。
- action
  - `-a` 指定用户加入组。
  - `-d` 从组中删除用户。
  - `-p` 从组中删除所有成员。
  - `-l` 显示组成员列表。

# sudo

- `sudo -V` 显示版本号
- `sudo -h` 帮助
- `sudo -l` 显示当前执行者的权限
- `sudo -v` 重新验证用户并顺延验证失效时间
- `sudo -k` 强制在下次执行 `sudo` 时验证密码
- `sudo [-b] [-u {username}|{#uid}] {cmd}`
  - `-b` 后台执行此命令
  - `-u {username}|{#uid}` 用指定用户执行命令。缺省为 root。

赋予 sudo 特权实际上是修改 `/etc/sudoers`。该文件只能用 root 用户通过 `visudo` 来编辑。

`/etc/sudoers` 中包含两类内容：别名定义、授权规则。

**别名定义**

`{aliasType} {name1} = {val1}, {val2}...[: {name2} = {val3}, {val4}...]`

- `{aliasType}` 有以下几种
  - `Host_Alias` 主机别名
  - `User_Alias` 用户别名，可以是用户或用户组，若是用户组，前面要加 `%`
  - `Runas_Alias` runas 别名，即 `sudo` 允许切换至的用户
  - `Cmnd_Alias` 命令别名

**授权规则**

`{user}|{group} {host}=[({runAs})] [NOPASSWD:] [!]{cmd1}[, {cmd2}...]`

- `{user}|{group}` 授权用户或用户组，用户组前面要加 `%`
- `{host}` 授权规则对应的主机
- `({runAs})` 切换到哪些用户/组。缺省为 `(root:root)`。若为 `(ALL)`，即 `(ALL:ALL)`，则可切换到任意用户/组。
- `NOPASSWD:` 不需要验证密码。
- `[!]{cmd1}[, {cmd2}...]` 允许执行的命令。`!` 表示禁止执行。为了避免执行到同名命令导致安全问题，应写命令的全路径，执行时同理。可以使用通配符。

```
 # 井号为注释

 # 默认有这么一行，可以当作参照
root  ALL=(ALL)  ALL

 # 给指定用户添加 sudo 权限
tom  ALL=(ALL)  ALL

 # 给指定用户组的所有用户添加 sudo 权限
%example  ALL=(ALL)  ALL

 # 使用 `sudo` 时不需要输入密码，一般不这样做，不太安全
ben  ALL=(ALL)  NOPASSWD: ALL

 # 指定可以通过 `sudo` 执行的命令的白名单。
 # 多个命令之间用 `, ` 分隔。
 # 命令必须写全路径，执行时也一样。为的是避免其他路径的同名命令被执行。
 # `!` 表示禁止执行。
%example  ALL=/sbin/mount /mnt/example, !/sbin/umount /mnt/example
```

# 命令

## 常用

### cat

concatenate 显示文件内容。

`cat {file1} [{file2}]...`
显示一个或多个文件的内容。对于多个文件，会将内容拼接起来显示。

### ls

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

### cp

复制文件。

`cp {file1} {file2}`
将 file1 复制到 file2。
`cp {file1} ... {file2} {dir}`
将文件复制到指定目录。

### mv

重命名或移动文件。

`mv {file1} {file2}`
将 file1 重命名为 file2。
`mv {file1} ... {file2} {dir}`
将文件移动到指定目录。

### touch

> 创建文件。若文件已存在则更新时间戳。

`touch {file}`

### rm

> 删除文件。

`rm {file}`

### echo

将它的参数显示到标准输出。

`echo {variable}`


### cd

> 设置当前工作目录。当前工作目录是指进程和 shell 当前所在的工作目录。

`cd {dir}`


### mkdir

> 创建新目录。

`mkdir {dir}`


### rmdir

> 删除目录，只能删除空目录。

`rmdir {dir}`
删除指定目录。
`rm -rf {dir}`
删除指定目录及其中所有内容。


### grep

> 显示文件和输入流中和参数匹配的行。可识别正则表达式。

`grep -[i][v] {keyword} {file}`
-i 表示不区分大小写，-v 表示反转匹配，即显示所有不匹配的行。


### less

分屏显示文件内容。

`less` 命令是 `more` 命令的增强版，在一些没有 `less` 命令的 Unix 系统和嵌入式系统中可以使用 `more` 命令。

大部分移动命令同 vim，包括标记跳转。

`less [opt] {file}...`

- `-b {bufferSize}` 缓冲区大小。
- `-e` 文件显示结束后自动离开。
- `-f` 强制打开特殊文件，如外围设备代号、目录和二进制文件。
- `-g` 只标识最后搜索的关键词。
- `-i` 搜索忽略大小写。
- `-m` 显示类似 `more` 的百分比。
- `-N` 显示行号。
- `-o {file}` 将 less 输出内容保存到指定文件。
- `-Q` 不使用警告音。
- `-s` 将连续空行显示为一行。
- `-S` 舍弃行的超长部分。
- `-x {n}` 将 `{tab}` 显示为 n 个空格。

- `/{str}` `?{str}` 向下/上搜索字符串。
- `n` `N` 重复/反向重复上一次搜索。
- `y` `<enter>` 上/下滚一行。
- `u` `d` 上/下滚半页。
- `b` `<space>` 上/下滚一页。
- `<pageup>` `<pagedown>` 向上/下滚一页。
- `:n` `:p` 下/上一个文件。
- `Q` 推出 `less`。
- `h` 帮助。

### pwd

> 输出当前的工作目录名。


### diff

> 查看两个文件之间的不同。

`diff {file1} {file2}`


### file

> 查看文件的格式信息。

`file {file}`


### find

> 查找文件。可以使用模式匹配参数，但必须给模式匹配参数加引号，因为 shell 会在运行命令前展开通配符。

`find {dir} -name {file} -print`


### locate

> 在系统创建的文件索引中查找文件。该索引由系统周期性地进行更新。


**head** 和 **tail**

> 显示文件的前 10 行或后 10 行内容。

`head|tail [-{n}|+{n}][-f] {file}`
-n 设置显示的行数，+n 设置显示第 n 行开始的内容，-f 设置不停地读取最新内容。


### sort

> 将文件内的所有行按照字母顺序快速排序。-n 按照数字顺序排序那些以数字开始的行，-r 反向排序。


### export

> 将某个 shell 变量设置为环境变量。

`export {var}`


### man

### {command} --help|-h

### info {command}

> 获取在线帮助。

`man [-k] {command}`
获取指定命令的帮助信息，可指定关键词进行查找。


### {command} > {file}

### {command} >> {file}

### {command} | {command}

### {command} 2> {file}

### {command} > {file} 2>&1

### {command} < {file}

> `>`将执行结果重定向至文件（缺省为终端屏幕），如果文件不存在，shell 会创建新文件；如果文件存在，shell 会先清空文件内容。bash 中可以通过设置参数`set -C`来防止清空。
`>>`将执行结果重定向至文件末尾。
`|`将执行结果作为后一个命令的输入。
`2>`重定向标准错误输出。2 是由 shell 修改的流 ID，1 是标准输出，2 是标准错误输出。
`>&`将标准输出和标准错误输出重定向到同一个地方。
`<`将文件内容重定向为命令的标准输入。


### du

> 查看文件和目录的磁盘使用空间。

`du -sh ./*`
显示当前目录下所有文件和目录的磁盘使用情况。

### chmod

`chmod [ops] {xyz} {path}`

- `-R` 对目录内部递归执行。
- `{xyz}` 依次对应拥有者、拥有者所在组、其他用户。rwx 641。

### chown

`chown [ops] {userOrGroup} {path}`

- `{userOrGroup}` 用户名或组名。
- `-c` 若拥有者确实发生更改才显示更改动作。
- `-f` 若拥有者无法更改，不显示错误信息。
- `-h` 更改链接本身，而不是它指向的路径。
- `-v` 显示详情。

## 进程

**ps**

列出所有正在运行的进程。

PID：进程 ID。
TTY：进程所在的终端设备。
STAT：进程在内存中的状态。完整状态列表参阅帮助手册。
TIME：进程占用 CPU 的总时长。
COMMAND：命令名，进程有可能将其由初始值改为其他。

- `ps x` 显示当前用户运行的所有进程。
- `ps ax` 显示系统当前运行的所有进程，包括其他用户的进程。
- `ps u` 显示更详细的进程信息。
- `ps w` 显示命令的全名，而非仅显示一行以内的内容。
- `ps {PID}` 显示指定进程的信息，可用 $$ 表示当前 shell 的进程。


**kill**

向进程发送一个信号。

`kill [-{SIG}] {PID}`
{SIG} 是具体的信号。缺省是 `TERM`。`STOP` 表示暂停，`CONT` 表示继续，`INT` 或 `CTRL-C` 表示中断，`KILL` 表示强行终止。也可使用数字代替信号名。

- `KILL` 在内核层面立即终止进程，该信号不能被阻塞、处理或忽略。
- `INT` 通知前台进程组终止进程。
- `QUIT` 和 INT 类似, 但由 QUIT 字符(通常是 `Ctrl-\`)来控制。进程在因收到 QUIT 退出时会产生 core 文件, 在这个意义上类似于一个程序错误信号。
- `TERM` 要求进程自己正常退出，该信号可以被阻塞或处理。


# 特殊字符

|字符|名称    |用途                         |
|:---|:-------|:----------------------------|
|\*  |星号    |正则表达式，通用字符         |
|.   |句点    |当前目录，文件/主机名的分隔符|
|!   |感叹号  |逻辑非运算符，命令历史       |
|\|  |管道    |命令管道                     |
|/   |斜线    |命令分隔符，搜索命令         |
|\   |反斜线  |常量，宏（非目录）           |
|$   |美元符号|变量符号，行尾               |
|'   |单引号  |字符串常量                   |
|`   |反引号  |命令替换                     |
|"   |双引号  |半字符串常量                 |
|^   |脱字符  |逻辑非运算，行头             |
|~   |波浪字符|逻辑非运算符，目录快捷方式   |
|#   |井号    |注释，预处理，替换           |
|[]  |方括号  |范围                         |
|{}  |大括号  |声明块，范围                 |
|_   |下划线  |空格的简单替代               |

控制键 CTRL 通常用 ^ 来表示。

# 命令行按键

|按键  |操作                        |
|:-----|:---------------------------|
|CTRL-B|左移光标                    |
|CTRL-F|右移光标                    |
|CTRL-P|查看上一条命令（或上移光标）|
|CTRL-N|查看下一条命令（或下移光标）|
|CTRL-A|移动光标至行首              |
|CTRL-E|移动光标至行尾              |
|CTRL-W|删除前一个词                |
|CTRL-U|删除从光标至行首的内容      |
|CTRL-K|删除从光标至行尾的内容      |
|CTRL-Y|粘贴已删除的文本            |


# 配置

## 安装配置文件

kickstart
/root/anaconda-ks.cfg

## yum 代理

`sudo vi /etc/yum.conf`

加入
proxy=http://{host}:{port}
proxy_username=abc
proxy_password=123

## 全局代理

1. `sudo vi /etc/profile`
2. 加上

```
http_proxy=http://{user}:{pwd}@{host}:{port}
export http_proxy
```

## 添加到 dock 栏

1. 创建 `.desktop` 文件。
```
 #!/usr/bin/env xdg-open
 [Desktop Entry]
 Version=1.0
 Type=Application
 # 若是脚本且想看到命令行则改为 true
 Terminal=false
 # 必须，填启动文件路径，后面可加启动参数
 Exec=command to run here
 # 必须，用于搜索和显示
 Name=visible name here
 Comment=comment here
 Icon=icon path here
```
2. 添加到桌面。把上述文件移至桌面。
3. 添加到 dock 栏和 app 搜索。把上述文件移至 `~/.local/share/applications/`。

# 问题

## `FATAL:Could not read from Boot Medium! System Halted.`

解决：虚拟机设置-存储-删除多余的控制器

## 参数风格

1. 参数用一横的说明后面的参数是字符形式。
2. 参数用两横的说明后面的参数是单词形式。
3. 参数前有横的是 System V 风格。
4. 参数前没有横的是 BSD 风格。

## Failed to open \EFI\BOOT\grubx64.efi - Not Found

1. 挂载 centos 的 iso，进入 rescue 模式。
2. `cd /mnt/sysimage/boot/efi/EFI`
3. `cp centos/grubx64.efi ../boot/`
4. `reboot`

## vdi 迁移

`VBoxManage internalcommands sethduuid H:\virtualSys\ubuntu.vdi`
