# Week1

## Day1

### 1.`pwd`

pwd 在 Bash 里是 shell builtin（内建命令），直接由 Bash 处理，同时磁盘上也存在 /bin/pwd，是一个独立的外部命令

两者的区别是：内建版 pwd 直接读取 Shell 内存中维护的 $PWD变量，速度更快；外部版 /bin/pwd 需要从磁盘加载执行，走独立的系统调用。

```
/home/momo123456
```

### 2.`ls -la ~`

`~` = `/home/momo123456`**的快捷写法**

`ls` = `**list**`(列出)

`-l` = 用长格式显示（权限、大小、时间）

`-a` = 显示所有文件（包括隐藏文件，以 . 开头的）

```
total 40
drwxr-x--- 5 momo123456 mo 4096 May 17 06:50 .
drwxr-xr-x 3 root       root       4096 May  3 06:54 ..
-rw------- 1 momo123456 mom 1463 May 17 06:44 .bash_history
-rw-r--r-- 1 momo123456 mo  220 Feb 13 12:16 .bash_logout
-rw-r--r-- 1 momo123456 mo 3869 May 12 15:53 .bashrc
drwx------ 2 momo123456 mo 4096 May  3 10:44 .cache
-rw------- 1 momo123456 mo   37 May 17 06:49 .lesshst
drwxrwxr-x 3 momo123456 mo 4096 May 11 07:09 .local
-rw-r--r-- 1 momo123456 mo  807 Feb 13 12:16 .profile
drwx------ 2 momo123456 mo 4096 May  3 10:26 .ssh  
```

| 文件/目录       | 权限               | 作用                 |
| --------------- | ------------------ | -------------------- |
| `.bashrc`       | `-rw-r--r-- (644)` | 每次开终端的配置文件 |
| `.profile`      | `-rw-r--r-- (644)` | 登录时执行的配置     |
| `.bash_history` | `-rw------- (600)` | 你敲过的所有命令记录 |
| `.ssh`          | `drwx------ (700)` | SSH 密钥目录         |
| .`cache`        | `drwx------ (700)` | 缓存目录             |
| `.local`        | `drwxrwxr-x (775)` | 用户本地程序数据     |

**几个值得注意的点**：

  第一，你的 .ssh/ 目录权限是 700（只有你自己能进），.bash_history 是 600（只有你能看）—— 这说明你这台 Ubuntu
  的默认安全基线还不错。很多人装完系统没注意过这些。

  第二，. 和 .. 是特殊目录——. 代表当前目录（你的家目录），.. 代表上级目录（/home）。注意到 .. 的所有者是 root
  吗？普通用户的家目录归 root 管，这说明 /home 是 root 创建的，你只是使用者。

### 3.`cd`

是shell内建命令，可以直接更改shell的工作目录

```
cd /etc \\进入了etc目录
cd - \\退回到上一级目录
cd ~ \\回到家目录
```

### 4.`man hier`

`/etc` —— 系统配置文件（你刚才去过的）

`/var` —— 变动数据（日志、数据库等）

`/tmp` —— 临时文件

`/home` —— 用户家目录

`/` ——根目录

`/bin` —— 里面放着需要单用户模式或者对系统开机或者修复有关的可执行程序，但是在较新的Ubuntu中是指向 `/usr/bin` 的一个软链接（symlink）

### 5.`Shell`

你敲命令，他帮你转达给系统内核去干活。（本身也是文件）

**区分内建命令和外部命令**

```
命令：type cd

      cd is a shell builtin
命令：type ls

     ls is aliased to 'ls --color=auto'
```

**内建命令**

代码直接写在 `Bash` 这个程序里，不需要去硬盘上找。所以无论你在哪个目录，`cd` 都能用——它不依赖任何外部文件。反过来想：如果 `/bin/bash` 被人替换了，那所有内建命令都不可信——这也是安全上的一个关注点。（是**shell亲自执行**）

**外部命令**

**shell开子进程执行**。从安全的角度看外部命令在子进程里：
    能干的：删你的文件、读你的数据、连网络……（文件系统权限管这个）
    不能干的：改你的当前目录、改你的环境变量、改你的 alias、换你的 shell（这些属于父进程的内部状态，子进程碰不到）

  所以进程隔离是一层"硬防护"——内核机制保证的，不是靠外部程序自觉。就算一
  个恶意程序故意想改你的工作目录，也做不到，保护`Shell`自身的内部状态（当前目录、环境变量、`alias` 这些）   

但这个边界不负责文件安全。如果一个恶意外部程序被你执行了（比如你下载了一个带后门的脚本），它虽然改不了你的 Shell 状态，但可以 `rm -rf  ~`、可以读取`.ssh/`、可以往外发数据。这些要靠权限来防      

**安全角度**

假设系统被入侵，攻击者替换了 `/bin/ls、/bin/ps、/bin/netstat`（这种手法叫 `rootkit`）。这时候你敲的 `ls`其实是假的——它不会显示攻击者的木马进程和文件。

  但你还可以用内建命令排查：

```
  echo /proc/[0-9]*           # 列出所有进程（echo 是内建，走不了假）
  cd /proc                     # 进 proc 文件系统看进程（cd 是内建）
  type ls                     # 检查 ls 到底指到哪去了
```

不过内建命令也不是绝对可靠——如果攻击者替换了 **/bin/bash**本身，那就全完了。到那个地步只能 Live USB 挂载磁盘来检查。

不信任当前 Shell 时用 `/bin/cat`绝对路径"就是这个道理——在应急响应里，每个命令的可信度是有等级的。                                                                                                        

**别名（alias）**

比如`ls` `ls` 实际执行的是 `ls --color=auto`，加了个 `--color=auto` 参数。好处是文件名根据类型用不同颜色显示，方便辨认。
安全角度：如果有人恶意改了你的别名，比如 `alias sudo='rm -rf /'`……后果自己想想。查看所有别名：`alias`

**外部命令**

比如 `ls` 本体（没有别名的情况下）

存在 `/bin/ls` 这个文件里，`Shell` 去磁盘上找到它再执行。

### 6.$ 和 # 的区别

  提示符最后一位：

`mo@ubuntu:~$` —— $ 表示普通用户

`root@ubuntu:~#` —— # 表示 root 用户

**安全运维第一课：没事不用 root。**



# Day2

### 实验：

#### 1.创建目录

`mkdir` = make directory（创建目录）

`-p` = 如果没有父目录，自动创建（parent）

```
命令：mkdir -p ~/week1-lab/subdir
命令：ls -la ~/week1-lab/

     total 12
     drwxrwxr-x 3 mo mo 4096 May 17 09:01 .
     drwxr-x--- 6 mo mo 4096 May 17 09:02 ..
     drwxrwxr-x 2 mo mo 4096 May 17 09:01 subdir
```

#### 2.创建文件

`touch` = 创建一个空文件（如果文件已存在，就更新它的时间戳）

`echo "Hello Linux" > hello.txt` = 把文字写入文件

接下来我们来看看结果：

```
命令:ls -la ~/week-lab/

    total 16
    drwxrwxr-x 3 mo mo 4096 May 17 09:16 .
    drwxr-x--- 6 mo mo 4096 May 17 09:17 ..
    -rw-rw-r-- 1 mo mo  12 May 17 09:16 hello.txt
    drwxrwxr-x 2 mo mo 4096 May 17 09:01 subdir
    -rw-rw-r-- 1 mo mo    0 May 17 09:07 test.txt
命令：cat ~/week1-lab/hello.txt

     Hello Linux
命令：echo "Second line" >> ~/week1-lab/hello.txt

     Hello Linux
     Second line
命令：echo "Hello" > ~/week1-lab/hello.txt

     Hello
     
\\一个>会覆盖内容，>> 是追加
```

#### 3.备份文件/移动文件

```
命令：cp ~/week1-lab/hello.txt ~/week1-lab/hello.bak
     mv ~/week1-lab/hello.txt ~/week1-lab/subdir/
     
     ls -la ~/week1-lab/
     total 16
     drwxrwxr-x 3 mo mo 4096 May 17 11:41 .
     drwxr-x--- 7 mo mo 4096 May 17 11:43 ..
     -rw-rw-r-- 1 m mo    6 May 17 11:40 hello.bak
     drwxrwxr-x 2 mo mo 4096 May 17 11:41 subdir
     -rw-rw-r-- 1 mo mo    0 May 17 11:16 test.txt
     
     ls -la ~/week1-lab/subdir/
     total 12
     drwxrwxr-x 2 mo mo 4096 May 17 11:41 .
     drwxrwxr-x 3 mo mo 4096 May 17 11:41 ..
     -rw-rw-r-- 1 mo mo    6 May 17 11:35 hello.txt 
```

#### 4.删除文件

```
rm ~/week1-lab/test.txt
```

```
现在你的目录结构应该是这样的：

  ~/week1-lab/
  ├── hello.bak      # 备份文件
  ├── subdir/
  │   └── hello.txt  # 被移过来的原文件
  └── test.txt       # 已删除
```

#### 5.`man`的使用

 `/` 键：搜索

`q` ：退出

<img src="C:\Users\angle\Pictures\Screenshots\屏幕截图 2026-05-18 143909.png" alt="屏幕截图 2026-05-18 143909"  />

在覆盖已有文件时会先询问你确认，防止手误覆盖重要文件

#### 6.设置别名 `alias rm='rm -i'`（`=`左右不能有空格）

```
alias rm='rm -i'
#会提示: rm: remove regular empty file '~/test_delete'?
# 输入 y 确认删除，n 取消
```

注意：这个别名只在当前终端会话有效。要永久生效，需要加到 ~/.bashrc 里。

```
rm -r 文件夹名
#可以删除非空文件夹
```



## Day3

### 1.看 .ssh 目录权限

```
ls -la ~/.ssh/
total 8
drwx------ 2 mo mo 4096 May  3 10:26 .
drwxr-x--- 5 mo mo 4096 May 18 07:02 ..
-rw------- 1 mo mo    0 May  3 10:26 authorized_keys
```

```
stat ~/.ssh/authorized_keys
  File: /home/mo/.ssh/authorized_keys
  size: 0         	Blocks: 0          IO Block: 4096   regular empty file
Device: 252,0	Inode: 1048583     Links: 1
Access: (0600/-rw-------)  Uid: ( 1000/momo123456)   Gid: ( 1000/momo123456)
Access: 2026-05-03 10:26:25.891764907 +0000
Modify: 2026-05-03 10:26:25.890772606 +0000
Change: 2026-05-03 10:26:25.891405327 +0000
 Birth: 2026-05-03 10:26:25.890772606 +0000

```

### 2.创建测试文件练手

```
touch ~/week1-lab/test
ls -l ~/week1-lab/test

chmod 644 ~/week1-lab/test
ls -l ~/week1-lab/test

-rw-r--r-- 1 mo mo 0 May 18 07:20 /home/mo/week1-lab/test

chmod 700 ~/week1-lab/test
ls -l ~/week1-lab/test

-rwx------ 1 mo mo 0 May 18 07:20 /home/mo/week1-lab/test

chmod 777 ~/week1-lab/test
ls -l ~/week1-lab/test
  
-rwxrwxrwx 1 mo mo 0 May 18 07:20 /home/mo/week1-lab/test

```

### 3.符号法加执行权限

```
默认权限

-rw-rw-r-- 1 mo mo 0 May 18 07:20 /home/mo/week1-lab/test

chmod u+x ~/week1-lab/test
ls -l ~/week1-lab/test 

-rwxrw-r-- 1 mo mo 0 May 18 07:20 /home/mo/week1-lab/test
```

### 4.核心实验：目录去掉 x 权限会怎样

```
chmod 644 ~/week1-lab
ls -ld ~/week1-lab       # 目录变成了 drw-r--r--，没有 x
cd ~/week1-lab           # 试试能不能进去？
不能进去，会报错
ls -l ~/week1-lab        # 能看到文件列表吗？
不能
```

 `ls`（不加参数）：只读目录内容本身。

  目录本质上是一个"文件名→inode 号"的映射表。有 r 权限就能读这张表，所以能看到文件名列表。

  `ls -l`：除了读文件名，还要对每个文件调 `stat()`。

  `stat()` 需要拿到文件的 inode 信息（权限、所有者、大小、时间戳）。而要到达文件的inode，内核必须穿透目录路径——这需要目录有 x 权限。

  没有 x 时，`ls -l` 的行为通常是：文件名能显示，但权限/所有者/大小那几列全是 ? 或报 Permission denied。

### 总结

| 权限 | 对文件       | 对目录         |
| ---- | ------------ | -------------- |
| r    | 查看内容     | 列出文件名(ls) |
| w    | 修改内容     | 创建/删除文件  |
| x    | 当作程序执行 | 进入目录(cd)   |



| 缩写 | 全称   | 含义                                 |
| ---- | ------ | ------------------------------------ |
| U    | User   | 文件所有者（创建文件的那个人）       |
| G    | Group  | 文件所属组（一个组里的所有用户）     |
| O    | Others | 其他人（既不是所有者，也不在所属组） |

举例：

用 ls -l 看：

  -rwx r-x ---
   │    │   └─ O (其他人)：无权限
   │    └──── G (所属组)：读 + 执行
   └──────── U (所有者)：读 + 写 + 执行

**数字法（八进制） 记住了就行：**

  r = 4
  w = 2
  x = 1

  把每一组的数字加起来：

  -rwx r-x ---
    │    │   └─ --- = 0+0+0 = 0
    │    └──── r-x = 4+0+1 = 5
    └──────── rwx = 4+2+1 = 7

  所以 -rwxr-x--- = 750

  常见权限速记：
  - 755 = rwxr-xr-x — 经典的程序/目录权限
  - 644 = rw-r--r-- — 经典的文件权限
  - 700 = rwx------ — 仅所有者能碰
  - 600 = rw------- — 仅所有者能读写

**安全视角**：家目录如果是 `777`，等于开门揖盗



## Day4

### 1.口述"一切皆文件"的意思

**Linux系统中所有的硬件，软件，进程，网络，目录，设备全部都可以统一当成文件来看，shell本质也是文件，除了shell的内建命令外的命令也是文件**

### 2.`.bashrc` 和 `.profile` 有什么区别？

#### 分别在什么时候执行？

**`.bashrc`在每次打开终端的时候执行，`.profile`在登陆的时候执行**

#### 哪个更容易被攻击者利用做持久化后门？

**攻击者修改了.bashrc，每次打开终端都会先执行攻击者的木马程序，.profile 其实更隐蔽 — 因为它只在登录 Shell 时执行一次，平时很难注意到。而 .bashrc每次开终端都跑，反而更容易被 ps aux 等工具发现异常进程。所以有经验的攻击者两个都会改。**

### 3.提问

你在 Day3 做了"目录去掉 x 权限"的实验——目录变成 644 后，`cd` 进不去，`ls` 也看不到文件列表。那如果目录是
`rw-r-----（640）`，你是所有者，你能列出文件名吗？能进目录吗？

****目录 640 能列文件名、不能 `cd`。 补充一个细节：`ls -l` 可能对部分文件报错或显示不全信息，因为 `stat`**
 每个文件需要 x 权限来解析路径。纯 `ls`（不加 -l）没问题。**

你说 `.bashrc` 每次开终端执行，`.profile` 登录时执行。那当你通过 SSH远程登录到服务器时，这两个文件都会被执行吗？还是只执行其中一个？

**SSH 登录时两个都会执行 → 对。 但背后的机制值得理解：bash 本身不会自动执行 `.bashrc`（那是给交互式非登录 shell**
  **用的）。Ubuntu 之所以两个都跑了，是因为 `/etc/skel/.profile`（你家目录里那个）末尾有一段：**

```
  if [ -f "$HOME/.bashrc" ]; then
      . "$HOME/.bashrc"
  fi
```

  **是 .profile 主动调用了 .bashrc，不是 bash 替你做的。**

登录 Shell → 执行 .profile → .profile 主动 source .bashrc。如果攻击者删了 .profile里的这段调用，.bashrc 就不会跑了。

你说 authorized_keys 当前是空文件（size:0）。这意味着什么？——是好事还是坏事？如果有人想免密登录你的服务器，现在能做到吗？

**空文件说明还没有任何人配置过密钥认证——这不叫权限高，这叫"还没配"。好处是没人能通过密钥登录你，坏处是你自己也不能用密钥登录，只能输密码。而密码是可以被暴力破解的**

你说家目录的权限是 `drwxr-x---（750）`。注意最后三位是 ---，这意味着"其他人"对你家目录没有任何权限。那 Apache/Nginx
这类以 www-data 用户运行的 Web 服务器，能读取 `~/public_html/` 下的文件吗？

**www-data 不能访问 `~/public_html/` → 正确。 根因是"其他人"对你的家目录没有 x 权限，路径无法穿透。很多新手配 Web**
**时踩的坑就是这个——文件权限明明设了 644，但上级目录权限不够，导致 403。**

你 Day4 提到有经验的攻击者会同时改 `.bashrc` 和 `.profile`。那如果你怀疑自己的 `.bashrc` 被篡改了，在不信任当前 Shell
的情况下，你该用什么方式来安全地检查？

**方案一：用绝对路径调用命令——`/bin/cat ~/.bashrc`（避免 `alias` 劫持）**

**方案二：前面加反斜杠——`\cat ~/.bashrc`（临时绕过所有 `alias`）**

**方案三：从另一个用户 `su` 过来——`su - momo123456` 后 `/bin/cat ~/.bashrc`**

**方案四：最彻底的——挂载 live USB 查看磁盘文件**

要**删除目录里的文件**，你需要同时满足：
  - 对目录有 w（修改目录条目）— 你有
  - 对目录有 x（穿透路径，到达文件）



## Day5

### 1.确认黏滞位 (Sticky Bit)

```
ls -ld /tmp

drwxrwxrwt 12 root root 240 May 19 15:09 /tmp    \\看看权限位最后是不是 `t`。
```

黏滞位（Sticky Bit）是 Linux 特殊权限之一，数字表示为 1xxx（如 1777）。 作用设置了黏滞位的目录里，只有文件所有者才能删除自己的文件。

**为什么需要它**

  `/tmp` 目录的典型权限是 `drwxrwxrwt`。注意它允许所有人写（777），这意味着：

  - 任何用户都能在 `/tmp` 下创建文件
  - 如果没有黏滞位，任何用户也都能删除别人的文件——因为删除看的是目录的 w 权限，跟文件本身权限无关

  加了 t 之后：用户 A 在 `/tmp` 下创建了 A_temp_file，用户 B 无法删除它，即使 B 对 `/tmp` 有 w 权限。

  **怎么识别**

  `drwxrwxrwt`   ← 最后一位是 t（不是 x），说明 x + sticky 都有
  `drwxrwxrwT`   ← 最后一位是 T（大写），说明 sticky 有，但 x 没有（罕见）

  **怎么设置**  

```
chmod 1777 /some/dir    # 数字法
chmod +t /some/dir      # 符号法
```

  **安全视角**

  `/tmp` 是攻击者喜欢落脚的地方——所有人都能写，脚本 kiddie 常把恶意文件丢进去执行。黏滞位至少保证了"你的临时文件不会被别人
  删"，但如果攻击者用你的身份运行了进程，黏滞位就帮不上忙了。

### 2.确认 SUID

```
ls -l /usr/bin/passwd

-rwsr-xr-x 1 root root 93640 Feb  2 20:45 /usr/bin/passwd \\注意所有者执行位是 `s` 而不是 `x`
```

SUID（Set User ID）是 Linux 特殊权限中**最需要警惕**的一个，数字表示为 4xxx（如 4755）。

**作用**

执行文件时，以文件所有者的身份运行，而不是以执行者自己的身份。

最经典的例子：`passwd`

普通用户改密码需要写 `/etc/shadow`，但 `shadow` 的权限是 `-rw-r-----`，只有 **root** 能写。矛盾怎么解决？

  `/usr/bin/passwd` 的所有者是 `root`，并且设了 **SUID**。任何用户执行 passwd 时，这个进程暂时获得 `root` 身份，就能写 `shadow`
  了——写完就退出，提权瞬间结束。

**为什么是安全重灾区**

  攻击者拿到一个低权限 shell 后，第一件事通常是：

```
 find / -perm -4000 -type f 2>/dev/null
```

  列出系统所有 SUID 文件，然后逐个研究有没有可利用的漏洞。一旦某个 SUID程序存在缓冲区溢出、命令注入等漏洞，攻击者就能提权到 root。

 **怎么识别**

  `-rwsr-xr-x`   ← 所有者执行位是小写 s，SUID + x 都有
  `-rwSr-xr-x`   ← 所有者执行位是大写 S，SUID 有但 x 没有（异常情况）

 **怎么设置**

```
chmod 4755 /path/to/file    # 数字法
chmod u+s /path/to/file     # 符号法
```

**三个权限速记**

| 权限   | 数字 | 对文件           | 对目录               |
| ------ | ---- | ---------------- | -------------------- |
| SUID   | 4    | 以所有者身份执行 | 无意义               |
| SGID   | 2    | 以所属组身份执行 | 新建文件继承目录的组 |
| Sticky | 1    | 无意义           | 只能删自己的文件     |

### 3.审计所有 SUID 文件

```
find / -perm -4000 -type f 2>/dev/null

/usr/lib/openssh/ssh-keysign
/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/usr/lib/cargo/bin/sudo
/usr/lib/cargo/bin/su
/usr/bin/fusermount3
/usr/bin/passwd
/usr/bin/chsh
/usr/bin/sudo.ws
/usr/bin/chfn
/usr/bin/ntfs-3g
/usr/bin/newgrp
/usr/bin/su
/usr/bin/umount
/usr/bin/gpasswd
/usr/bin/mount
```

  这条命令列出系统里所有带 SUID 位的文件——每个都是潜在的提权入口。

注意列表里的 `/usr/lib/cargo/bin/sudo` 和 `/usr/lib/cargo/bin/su`——这是 Rust写的替代版本，不是系统默认的。了解一下就行，以后审计 SUID 时如果出现陌生路径（比如 `/tmp/.evil`），那才是红旗。

### 4.生成SSH密钥

```
ssh-keygen -t ed25519
```

完了之后执行 `ls -la ~/.ssh/` 确认生成了 `id_ed25519`（私钥）和 `id_ed25519.pub`（公钥）

### 5.把公钥部署到虚拟机

```
ssh-copy-id mo@IP
```

验证：

```
ssh mo@ip
```



## Day6

### 1.读配置文件，看有没有可疑别名

```
cat ~/.bashrc
```

### 2.读 .profile

```
cat ~/.profile
```

### 3.看最近 20 条命令记录

```
tail -20 ~/.bash_history
```

### 4.确认这三个文件的权限

```
ls -l ~/.bashrc ~/profile ~/.bash_history
```

### 5.创建测试用户

```
sudo useradd -m testuser && sudo passwd testuser
```

### 6.切到 testuser 并测试权限边界

```
su - testuser

sudo ls /root

被拒绝了，没有sudo权限
```

