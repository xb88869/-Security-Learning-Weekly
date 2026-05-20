# 第2周：用户管理、进程与文本三剑客入门

> **目标**：理解多用户隔离，学会看进程，迈出日志处理第一步。
> **周期**：5月18日（周一）— 5月23日（周六） → 5月24日（周天）发博客

---

## 每日任务清单

### Day 1（周一）| 用户管理基础

**目标**：学会创建用户、理解账户存在哪。

**动手清单：**
- [ ] `sudo useradd -m testuser` 创建新用户
- [ ] `sudo passwd testuser` 设置密码
- [ ] `su - testuser` 切换过去，观察提示符变化
- [ ] `exit` 退回原用户
- [ ] `cat /etc/passwd | grep testuser` 看用户记录（用户名:x:UID:GID:描述:家目录:Shell）
- [ ] `cat /etc/group | grep testuser` 看组信息
- [ ] `groups testuser` 查看用户所属组
- [ ] `id testuser` 查看用户 UID/GID

**理解要点：**
- `/etc/passwd` 每个字段的含义（用户名、密码占位符、UID、GID、描述、家目录、Shell）
- `/etc/shadow` 存哈希密码（只有 root 能读）
- UID 的含义：0=root，1-999=系统账户，1000+=普通用户
- `useradd` vs `adduser` 区别

**记录产出：**
- 写下 testuser 的 UID 和 GID
- 记录 `/etc/passwd` 中 testuser 那一行的完整内容

---

### Day 2（周二）| sudoers 与权限隔离

**目标**：理解 sudo 不是万能钥匙，学会查看和限制 sudo 权限。

**动手清单：**
- [ ] `su - testuser` 切换到 testuser
- [ ] `sudo ls /root` **观察被拒绝** —— testuser 不在 sudoers 中
- [ ] `exit` 退回原用户
- [ ] `cat /etc/sudoers` 看看 sudo 配置文件（或 `visudo`）
- [ ] 观察 `sudo -l` 查看当前用户有哪些 sudo 权限
- [ ] 可选拓展：`sudo usermod -aG sudo testuser` 将 testuser 加入 sudo 组后再试

**理解要点：**
- sudoers 机制：`/etc/sudoers` 决定了谁能用什么身份执行什么命令
- `root ALL=(ALL) ALL` 的含义：root 可以在任何主机上以任何用户身份执行任何命令
- 普通用户不在 sudoers 中 = 囚徒，即使他已经在系统里
- `visudo` 语法检查机制防止你把自己锁在外面

**安全思考：**
> 攻击者拿到一个低权限用户后，第一件事就是 `sudo -l` 试探。如果发现某个命令可以免密 sudo，提权的大门就打开了。这也是为什么审计 sudoers 配置是基线检查的必查项。

**记录产出：**
- 记下 `sudo -l` 的输出
- 思考一句话解释"为什么普通用户不能 sudo ls /root"

---

### Day 3（周三）| 进程管理入门

**目标**：学会看进程，理解 PID/PPID/STAT。

**动手清单：**
- [ ] `ps aux` 浏览所有进程
- [ ] 观察各列含义：USER、PID、%CPU、%MEM、VSZ、RSS、STAT、START、TIME、COMMAND
- [ ] `ps aux | head -5` 看前 5 行（含表头）
- [ ] 选一个进程（比如 bash），`ls -l /proc/<PID>/exe` 看可执行文件路径
- [ ] `ls -l /proc/<PID>/fd/` 看该进程打开的文件描述符
- [ ] `top` 启动交互式进程查看器（按 `q` 退出）
- [ ] `pstree` 看进程树形结构

**理解要点：**

| 列 | 含义 | 安全价值 |
|----|------|---------|
| PID | 进程 ID | 识别后门进程 |
| PPID | 父进程 ID | 追溯进程启动链 |
| STAT | 进程状态 | R=运行，S=睡眠，Z=僵尸，T=停止 |
| VSZ/RSS | 虚拟/物理内存 | 异常内存占用可能是挖矿进程 |
| COMMAND | 命令名 | 注意方括号括起来的 [kworker] 是内核线程，不是后门 |

- `/proc/<PID>/exe` 是一个符号链接，指向进程的可执行文件——这是应急响应里极常用的技巧
- **安全视角**：`ps aux` 看到一个 `/tmp/.X11-unix/x` 在监听端口？立刻拉警报

**记录产出：**
- 记下你的 bash 进程的 PID，查看其 exe 路径和打开的文件
- 运行 `pstree` 截下输出

---

### Day 4（周四）| 休息日——轻量回顾

- [ ] 不看笔记，口述 `/etc/passwd` 每一列的含义
- [ ] 口述 PID 和 PPID 的区别
- [ ] 思考：攻击者拿到低权限用户后，第一个试探的命令是什么？

---

### Day 5（周五）| 管道与 grep 基础

**目标**：掌握 `|` 管道的使用，学会用 grep 搜索文本。

**动手清单：**
- [ ] `ps aux | grep ssh` 过滤出含 ssh 的进程
- [ ] `ps aux | grep -v grep` 去掉 grep 进程本身（经典技巧）
- [ ] `echo -e "apple\nbanana\napple\ncherry" | grep apple` 基础过滤
- [ ] `cat /etc/passwd | grep "/bin/bash"` 找出所有能登录的用户
- [ ] `cat /etc/passwd | grep -v "/bin/bash"` 找出所有不能登录的用户
- [ ] `cat /etc/passwd | grep -c "bash"` 统计行数

**理解要点：**

**管道 `|` 的本质**：把左边命令的 stdout（标准输出）接到右边命令的 stdin（标准输入）。这是 Linux"组合小工具"哲学的核心。

**grep 的正则入门：**

| 符号 | 含义 | 示例 |
|------|------|------|
| `^` | 行首 | `^root` 匹配以 root 开头的行 |
| `$` | 行尾 | `bash$` 匹配以 bash 结尾的行 |
| `.` | 任意单个字符 | `r..t` 匹配 root、rust 等 |
| `*` | 前一个字符重复 0 次或多次 | `ro*t` 匹配 rt、rot、root |
| `[0-9]` | 数字范围 | `[0-9]\+` 匹配连续数字 |
| `\` | 转义 | `\.` 匹配字面句点 |

**常见选项：**
- `-i` 忽略大小写
- `-v` 反向匹配
- `-c` 计数
- `-n` 显示行号
- `-r` 递归搜索目录

**记录产出：**
- 写一条命令：找出 `/etc` 目录下所有包含 "root" 的配置文件
- 写一条命令：统计 `/etc/passwd` 中有多少个用户使用 bash

---

### Day 6（周六）| 日志分析实战——auth.log

**目标**：学会在日志中定位安全事件，动手用 last/lastb 读登录记录。

**动手清单：**
- [ ] `ls -la /var/log/` 浏览日志目录，认识常见日志文件
- [ ] `sudo last` 读 `/var/log/wtmp`——所有登录记录
- [ ] `sudo lastb` 读 `/var/log/btmp`——失败登录记录
- [ ] `sudo cat /var/log/auth.log | grep "Failed password"` 找失败登录
- [ ] `sudo cat /var/log/auth.log | grep "Failed password" | wc -l` 统计次数
- [ ] `sudo cat /var/log/auth.log | grep "Failed password" | awk '{print $(NF-3)}' | sort | uniq -c | sort -nr` 统计各 IP 尝试次数
- [ ] 自己 SSH 登录一次，然后 `sudo tail -f /var/log/auth.log` 实时观察

**理解要点：**
- `/var/log/auth.log`：认证日志，SSH 登录、sudo 都记在这
- `/var/log/syslog`：系统综合日志
- `/var/log/btmp`：失败登录记录（二进制，用 `lastb` 读）
- `/var/log/wtmp`：所有登录记录（二进制，用 `last` 读）
- `/var/log/dpkg.log`：软件包安装日志

> **安全价值**：暴力破解（SSH 爆破）的第一发现点就是 auth.log。脚本化的攻击会在短时间内产生大量连续 "Failed password" 记录。这也是你第6周写自动封禁脚本的数据来源。

**记录产出：**
- 找到你的 auth.log 中最新的 5 条 "Failed password" 记录（如果没有就自己输错几次密码制造一条）
- 运行 `sudo lastb` 截下输出
- 写一条命令统计 auth.log 中失败登录的来源 IP 数量

---

### Day 7（周日）| 复习 + 写博客

**目标**：串联所有知识点，输出本周博客。

**知识串联：**
- [ ] 用户管理：`useradd` → `/etc/passwd` → `su` → `sudo` 的完整链路
- [ ] 进程管理：`ps aux` → `/proc/PID/exe` → 识别异常进程
- [ ] 管道与 grep：`|` → `grep` → 日志过滤
- [ ] 日志分析：`last`/`lastb` → `auth.log` → 定位 Failed password

**安全思维演练：**
> 假设你是蓝队，发现 auth.log 中 1 分钟内出现 50 条来自同一 IP 的 "Failed password for root" 记录。你会如何响应？用这周学到的命令，列出你的排查步骤。

**博客写作：**
- [ ] 使用下方的博客模板，完成本周博客
- [ ] 推送到 GitHub 仓库

---

## 每周产出

**本周核心产出（Github 博客发布）：**

1. 一篇技术博客 —— 用户管理、进程与日志分析入门
2. 能用 `ps aux` + `/proc/PID/exe` 识别异常进程
3. 能在 `auth.log` 中找出所有失败登录记录并统计来源 IP
4. 已掌握 `last`/`lastb` 读取登录记录

**博客发布节奏：**
- 每周四发布上一篇的博客
- 本周博客于 **5月29日（周四）** 发布

---

## GitHub 博客模板（可直接使用）

```markdown
---
title: "第2周：用户管理、进程与日志分析——从创建用户到发现 SSH 爆破"
date: 2026-05-29
description: "学习多用户隔离、进程查看和日志分析，亲手在 auth.log 中发现 SSH 暴力破解痕迹。"
tags: [Linux, 安全, 用户管理, 进程, 日志]
---

## 前言

第1周我们搞懂了文件权限：谁是文件的主人、谁能读谁能写、SUID 为什么危险。这周把视角从"静态的文件"转向"动态的系统"——**用户从哪来、进程怎么跑、攻击来了怎么在日志里发现**。

---

## 一、用户管理：系统里的"人"存在哪？

### 三个核心文件

| 文件 | 内容 | 权限 | 说明 |
|------|------|------|------|
| `/etc/passwd` | 用户账户信息 | 644 | 所有人可读，存 UID、家目录、Shell |
| `/etc/shadow` | 密码哈希 | 640/600 | 只有 root 能读 |
| `/etc/group` | 组信息 | 644 | 所有人可读 |

### 创建用户的完整链路

```bash
sudo useradd -m testuser   # 创建用户，-m 同时创建家目录
sudo passwd testuser        # 设置密码
su - testuser               # 切换过去
```

执行完这三条，系统发生了什么？

1. `/etc/passwd` 新增一行：`testuser:x:1001:1001::/home/testuser:/bin/sh`
2. `/etc/shadow` 新增一行，存着密码的哈希值
3. `/home/testuser/` 目录被创建，从 `/etc/skel/` 复制了默认配置文件

### UID 的分水岭

| UID 范围 | 类型 | 例子 |
|----------|------|------|
| 0 | root | 拥有全部权力 |
| 1-999 | 系统账户 | www-data、mysql 等，不能登录 |
| 1000+ | 普通用户 | testuser，你的日常账户 |

> **安全要点**：`/usr/sbin/nologin` 作为 Shell 的账户无法交互登录。如果 `/etc/passwd` 里一个系统账户（比如 www-data）的 Shell 被改成了 `/bin/bash`，那就是攻击者干的。

---

## 二、sudoers：权力可以给，但不能滥

```bash
# testuser 试图访问 root 的家目录
su - testuser
sudo ls /root

[sudo] password for testuser:
testuser is not in the sudoers file.  This incident will be reported.
```

这句 **"This incident will be reported"** 不是吓你的——它确实被记录到了 `/var/log/auth.log` 里。

### sudoers 文件解读

```bash
# /etc/sudoers 核心语法
root    ALL=(ALL:ALL) ALL
```

- `root`：谁
- `ALL`：在哪些主机上
- `(ALL:ALL)`：以哪个用户:组的身份
- `ALL`：可以执行什么命令

翻译成人话：**root 可以在任何机器上以任何人的身份干任何事**。

**审计命令**：`sudo -l` —— 查看当前用户有哪些 sudo 权限。攻击者拿到低权限用户后，第一条试探就是这个。

---

## 三、进程管理：系统里每个"正在运行的程序"

### ps aux 每一列的含义

```bash
ps aux
USER         PID %CPU %MEM    VSZ   RSS STAT START   TIME COMMAND
root           1  0.0  0.4 102108 11356 ?    Ss   06:42   0:03 /sbin/init
momo1234   12345  0.0  0.1  11456  3456 pts/0 Ss   09:15   0:00 -bash
```

| 列 | 含义 | 安全价值 |
|----|------|---------|
| USER | 进程所属用户 | 异常用户跑的进程可疑 |
| PID | 进程 ID | |
| PPID | 父进程 ID | 追溯谁启动了它 |
| %CPU | CPU 占用率 | 挖矿进程 CPU 会飙到 90%+ |
| STAT | 状态 | R=运行, S=睡眠, Z=僵尸 |
| COMMAND | 命令名 | 看有没有异常路径 |

### 应急响应的关键技巧

```bash
# 查看进程的可执行文件路径
ls -l /proc/<PID>/exe

# 查看进程打开的文件
ls -l /proc/<PID>/fd/

# 查看进程树
pstree
```

> **为什么这很重要？** 攻击者经常把恶意进程改名伪装成系统进程（比如把木马改名叫 `[kworker]` 或 `systemd`）。`/proc/PID/exe` 会暴露真正的可执行文件路径——如果 `[kworker]` 的 exe 指向 `/tmp/.backdoor`，那它就不是内核线程。

---

## 四、管道与 grep：组合小工具的哲学

### 管道的本质

```bash
ps aux | grep ssh
```

`|` 把左边 `ps aux` 的输出（所有进程列表）直接喂给右边 `grep ssh`（只显示含 ssh 的行）。

这是 Linux"一个工具只做一件事，做好然后组合"的设计哲学。单个命令能力有限，但用 `|` 串起来威力巨大。

### grep 正则速查

```bash
grep "^root" /etc/passwd       # 行首匹配：以 root 开头
grep "bash$" /etc/passwd       # 行尾匹配：以 bash 结尾
grep "r..t" /etc/passwd        # 任意字符：r+t 之间有两个字符
grep "[0-9]\{3\}" file         # 连续 3 个数字
grep -v "nologin" /etc/passwd  # 反向匹配：不包含 nologin
grep -c "root" /etc/passwd     # 计数
```

---

## 五、日志分析——auth.log 实战

### 日志文件速览

| 文件 | 内容 | 读取方式 |
|------|------|---------|
| `/var/log/auth.log` | 认证日志（SSH、sudo） | `cat`/`grep` |
| `/var/log/syslog` | 系统综合日志 | `cat`/`grep` |
| `/var/log/wtmp` | 所有登录记录 | `last` |
| `/var/log/btmp` | 失败登录记录 | `lastb` |

### 在 auth.log 中找暴力破解

```bash
# 1. 找所有失败登录
grep "Failed password" /var/log/auth.log

# 2. 统计失败次数
grep "Failed password" /var/log/auth.log | wc -l

# 3. 统计各 IP 的尝试次数
grep "Failed password" /var/log/auth.log | \
    awk '{print $(NF-3)}' | \
    sort | uniq -c | sort -nr

# 4. 实时监控新登录
sudo tail -f /var/log/auth.log
```

这条 IP 统计命令解释：
1. `grep` 过滤出失败登录行
2. `awk '{print $(NF-3)}'` 提取倒数第 3 列（来源 IP）
3. `sort | uniq -c` 排序并统计每个 IP 出现次数
4. `sort -nr` 按次数从大到小排序

> **安全价值**：这条命令组合就是**暴力破解检测脚本**的核心。如果某个 IP 的失败次数超过阈值（比如 10 次/分钟），就可以判定为 SSH 爆破攻击。

---

## 六、蓝队思维演练

> **场景**：你早晨登录服务器，发现 auth.log 中 1 分钟内出现 50 条来自 192.168.1.100 的 "Failed password for root"。

**你的排查步骤**（用这周学的命令）：

```bash
# 1. 确认攻击来源和时间范围
grep "192.168.1.100" /var/log/auth.log

# 2. 统计总共尝试了多少次
grep "192.168.1.100" /var/log/auth.log | wc -l

# 3. 检查是否有一次成功了
grep "Accepted.*192.168.1.100" /var/log/auth.log

# 4. 封禁该 IP（iptables，第6周会细讲）
sudo iptables -A INPUT -s 192.168.1.100 -j DROP

# 5. 转存攻击日志备档
grep "192.168.1.100" /var/log/auth.log > attack-20260528-1921681100.log
```

---

## 七、这条命令链意味着什么？

本周学的东西有一条明显的递进链：

```
用户管理 (useradd) → 进程管理 (ps aux) → 文本过滤 (grep) → 日志分析 (auth.log) → 发现攻击
```

每一步都在为下一步提供数据。第6周你写 SSH 自动封禁脚本时，核心逻辑就是：

```
grep "Failed password" auth.log → awk 提取 IP → sort 统计次数 → 超过阈值 → iptables 封禁
```

这条链的起点，就是你本周学的 `grep + awk + sort + uniq` 组合。**每一个基础命令都是未来安全工具的零件**。

---

## 下周预告

第3周：**包管理、计划任务与 HTTP 明文**
- apt/dpkg 安装管理软件包
- crontab 设置定时任务
- 用 Python HTTP Server 起假登录页，抓包看 POST 明文

---

*欢迎 Star 和 Issue 讨论。这是 [我的安全学习周记系列] 的第2篇。*
```

---

## 发布指引

1. 在 GitHub 仓库（如 `Security-Learning-Weekly`）下建 `posts/` 目录
2. 把博客内容保存为 `2026-05-29-week2-users-processes-logs.md`
3. 每周四固定发布上一篇的博客
4. 第8周时可以整理一个总目录页

**每周博客写作节奏建议：**
- 周三晚：整理本周笔记，对照任务清单梳理知识点
- 周四上午：把模板中的内容补充为你自己的实际输出（命令截图、踩坑记录、思考感悟）
- 周四下午：推送 GitHub
