# scp 命令使用方法（最实用、最常用版）

`scp` = **secure copy**，是 Linux / Mac 自带的**跨服务器安全传输文件命令**，基于 SSH，不用额外安装。

我给你整理**最常用、工作中100%会用到**的用法，直接复制就能用。

---
grep

## 一、基本语法

```bash
# 本地 → 远程
scp [选项] 本地文件 远程用户@远程IP:远程路径

# 远程 → 本地
scp [选项] 远程用户@远程IP:远程文件 本地路径
```

---

## 二、最常用命令（直接背这几个）

### 1. 本地文件 → 远程服务器

```bash
scp test.txt root@192.168.1.100:/home/
```

### 2. 远程文件 → 本地

```bash
scp root@192.168.1.100:/home/test.txt ./
```

### 3. 传输**整个文件夹**（必须加 -r）

```bash
# 本地文件夹 → 远程
scp -r myfolder root@192.168.1.100:/home/

# 远程文件夹 → 本地
scp -r root@192.168.1.100:/home/myfolder ./
```

---

## 三、常用选项（工作必备）

- `-r`：递归传输**文件夹**

- `-P 端口`：**SSH 不是22端口时必须加**
  
  ```bash
  scp -P 2222 test.txt root@192.168.1.100:/home/
  ```

- `-v`：显示详细传输过程（debug用）

- `-i 密钥文件`：使用密钥登录传输
  
  ```bash
  scp -i mykey.pem test.txt root@192.168.1.100:/home/
  ```

---

## 四、最容易踩的坑

1. **传文件夹必须加 -r**
2. **端口用大写 -P**，不是小写 -p
3. 远程路径必须是**绝对路径**（/home/xxx）
4. Windows 也能用（PowerShell / Git Bash 都支持 scp）

---

## 五、一句话速记

```
本地→远程：scp 文件 用户@IP:目标路径
远程→本地：scp 用户@IP:文件 本地路径
传文件夹：scp -r ...
非22端口：scp -P 端口 ...
```

---

### 总结

- `scp` 是 Linux/Mac 自带的**SSH 安全文件传输命令**
- 最常用：**本地 ↔ 服务器互传文件/文件夹**
- 必记参数：`-r`（文件夹）、`-P`（指定端口）

需要我给你写**日本IT现场最常用的 scp 批量传输/自动化脚本**吗？

# `netstat -ntlp` 完整详解（运维/Java后端 必用）

## 一、拆解参数

```bash
netstat -ntlp
```

- `-n`：**数字形式**显示IP、端口，不解析域名/服务名（更快）
- `-t`：只看 **TCP** 协议
- `-l`：只看 **监听中(Listening)** 的端口（对外提供服务的）
- `-p`：显示**进程PID、进程名**（谁占用了端口）

---

## 二、输出字段含义

```
tcp  0  0  0.0.0.0:8080  0.0.0.0:*  LISTEN  12345/java
```

1. `tcp`：协议类型
2. 接收队列/发送队列（一般都是0）
3. `0.0.0.0:8080`：**本机监听IP:端口**
   - `0.0.0.0`：本机所有网卡都监听
   - `127.0.0.1`：仅本机访问，外部连不上
4. `0.0.0.0:*`：外部连接地址（监听状态固定）
5. `LISTEN`：监听状态，等待连接
6. `12345/java`：**PID / 程序名**

---

## 三、日常高频用法

### 1. 查端口被谁占用（最常用）

```bash
# 查8080端口
netstat -ntlp | grep 8080
```

### 2. 查看所有TCP正在监听的服务

```bash
netstat -ntlp
```

### 3. 查看所有连接（包含已建立连接）

```bash
netstat -ntap
```

### 4. 查看UDP端口

```bash
netstat -nulp
```

---

## 四、补充（CentOS / Ubuntu 适配）

1. 若提示 **command not found**，安装：
   
   ```bash
   # CentOS / RHEL
   yum install net-tools -y
   ```

# Ubuntu

apt install net-tools -y

```
2. 现代系统推荐替代命令（更快）
```bash
ss -ntlp
```

---

## 五、日本IT/面试 必考短句

- `netstat -ntlp`：確認**待ち受けポート**と、ポートを占有しているプロセスを表示するコマンド
- ポート競合、プロセス特定、サーバー疎通確認で多用

---

需要我顺带补充：**根据PID杀进程、强制释放端口** 全套命令吗？

# 一、完整组合：查端口 → 看PID → 杀进程（日常运维全套）

## 1. 查看所有监听TCP端口+进程

```bash
netstat -ntlp
```

## 2. 过滤指定端口（比如 8080、3306、22）

```bash
netstat -ntlp | grep 8080
```

输出示例：

```
tcp6  0  0 :::8080  :::*  LISTEN  25689/java
```

👉 `25689` 就是 **PID**

---

# 二、根据 PID 杀进程

## 温柔关闭（推荐，先尝试正常退出）

```bash
kill 25689
```

## 强制杀死（卡死、关不掉时用）

```bash
kill -9 25689
```

---

# 三、直接一行：根据端口号直接杀进程（神器）

## Linux 通用

```bash
# 语法：fuser -k 端口/tcp
fuser -k 8080/tcp
```

## 无 fuser 时，一行组合命令

```bash
kill -9 $(netstat -ntlp | grep 8080 | awk '{print $7}' | cut -d/ -f1)
```

---

# 四、现代替代命令（CentOS7+/Ubuntu 推荐）

`net-tools` 逐渐淘汰，用 `ss` 更快

```bash
# 查看监听端口
ss -ntlp

# 查指定端口
ss -ntlp | grep 8080
```

---

# 五、额外高频实用命令

1. 查看当前系统所有进程
   
   ```bash
   ps -ef
   ```

2. 查找Java进程（后端必用）
   
   ```bash
   ps -ef | grep java
   ```

3. 查看端口连通性（测试服务器通不通）
   
   ```bash
   telnet 目标IP 端口
   # 或
   curl -v 目标IP:端口
   ```

---

# 六、日本現場用語メモ（面接・業務）

- ポート占有：端口占用
- プロセスID(PID)：进程ID
- kill -9：強制終了
- LISTEN：待ち受け状態

需要我给你整理一份 **Linux 运维高频命令极简清单（面试+現場両用）** 吗？

直接结论：
**`yum list installed` 只能看「当前已安装版本」，看不到有没有最新更新。**

## 一、三条精准命令，按需用

### 1. 只查【单个软件】有没有更新（最常用）

```bash
yum check-update 软件名
```

示例：

```bash
yum check-update nginx
```

- 有更新：会列出 旧版本 → 新版本
- 无更新：空白输出

### 2. 查【全部软件】可更新列表

```bash
yum check-update
```

只展示需要升级的包，干净无多余内容。

### 3. 对比：同时看「已安装版 + 仓库最新版」

```bash
yum list 软件名
```

输出会分两行：

- `installed`：你现在装的版本
- `available`：源里最新可升级版本

---

## 二、三者区别一目了然

| 命令                   | 作用           | 是否显示更新  |
| -------------------- | ------------ | ------- |
| `yum list installed` | 仅列出本机已安装包    | ❌ 看不到新版 |
| `yum list xxx`       | 已安装 + 仓库可用新版 | ✅ 能对比   |
| `yum check-update`   | 只列出可升级的包     | ✅ 纯更新列表 |

---

## 三、实操例子

```bash
# 1. 查看当前安装
yum list installed | grep openssl

# 2. 查有没有更新
yum check-update openssl

# 3. 对比本地&仓库最新版
yum list openssl
```

## 四、补充

如果只想**不升级、只巡检补丁**，就用：

```bash
yum check-update
```

需要升级单个就执行：

```bash
yum update 包名 -y
```

需要我顺带告诉你如何**只看安全补丁更新**，不更功能版本吗？

## YUM 更新「单个指定软件」完整操作（只更这一个，不全局升级）

### 一、核心命令（推荐）

```bash
# 更新单个软件，只升级该包+它的依赖，不更新系统其他软件
yum update -y 软件包名
```

---

### 二、实操步骤

1. **先查当前已安装版本 & 包全名**
   
   ```bash
   yum list installed | grep 关键词
   # 或
   rpm -qa | grep 关键词
   ```

示例：更新 `nginx`

```bash
yum list installed | grep nginx
```

2. **单独更新这一个软件**
   
   ```bash
   yum update -y nginx
   ```

---

### 三、扩展常用用法

1. **只检查是否有更新，不安装**
   
   ```bash
   yum check-update 包名
   ```

2. **不升级依赖，只更新软件本身（谨慎用）**
   
   ```bash
   yum update -y 包名 --exclude="*"
   ```

3. **忽略某个包、防止被意外升级**
   
   ```bash
   # 临时单次排除
   yum update --exclude=nginx
   ```

# 永久锁定不更新

yum versionlock add 包名

```
4. **重新安装/覆盖更新（损坏、文件缺失时）**
```bash
yum reinstall -y 包名
```

---

### 四、关键区别

- `yum update` 不带包名：**全系统所有软件全部更新**
- `yum update 包名`：**仅更新指定单个软件 + 必要依赖**

---

### 五、举例合集

```bash
# 更新 java
yum update -y java

# 更新 mysql
yum update -y mysql

# 更新 openssl（安全补丁常用）
yum update -y openssl
```

需要我教你「锁定某个软件版本，永远不自动升级」的配置吗？

# yum 安装**单个 rpm 包**的方法（超简单）

`yum` 完全可以直接安装本地/下载好的 `.rpm` 文件，**还能自动帮你解决依赖**（比 `rpm -ivh` 好用太多）。

## 1. 最常用命令（推荐）

```bash
yum install -y 包名.rpm
```

### 示例：

```bash
yum install -y google-chrome.rpm
```

---

## 2. 完整步骤（你照着做就行）

1. 先把 `.rpm` 包传到服务器，或用 `wget` 下载

2. 进入 rpm 所在目录

3. 执行安装命令：
   
   ```bash
   yum localinstall -y 文件名.rpm
   ```
   
   > **`localinstall` = 本地安装**，老版本 yum 专用，新版本用 `install` 也完全兼容。

---

## 3. 两个命令区别

| 命令                    | 作用                   |
| --------------------- | -------------------- |
| `rpm -ivh xxx.rpm`    | 只装包，**不解决依赖**，缺依赖会报错 |
| `yum install xxx.rpm` | 装包 + **自动联网补全依赖**    |

**日常安装一律用 yum 版本！**

---

## 4. 常用小技巧

### 强制重新安装

```bash
yum reinstall -y 包名.rpm
```

### 不安装，只检查依赖

```bash
yum install -y 包名.rpm --test
```

---

### 总结

你只需要记住这 **1 条命令** 就能装任何 rpm 包：

```bash
yum install -y 你的包.rpm
```

需要我帮你解决 **rpm 安装报错、依赖缺失、版本冲突** 吗？

这两个命令**长得像、但用途完全不一样**，我用最简单、最清晰的方式给你讲明白👇

# 一、核心区别（一句话记住）

**yum list = 查“包的版本/状态”**  
**yum search = 查“包叫什么名字”**

# 二、详细对比

## 1. yum list

**作用：列出包 + 显示版本**  

- 看**已安装**的包  
- 看**仓库里可安装**的包  
- 对比**当前版本 vs 最新版本**

**用法：**

```bash
yum list          # 列出所有包（巨多）
yum list installed  # 只看已安装
yum list nginx     # 只看 nginx 的版本信息
```

**输出样子：**

```
nginx.x86_64   1.20.1-9   @updates
```

告诉你：**装了没、版本多少、来自哪个源**。

---

## 2. yum search

**作用：搜索包名/描述**  

- 你**不知道包叫啥**时用  
- 按**关键词**找包  
- 不显示版本，只显示包名 + 简介

**用法：**

```bash
yum search nginx
yum search 浏览器
```

**输出样子：**

```
nginx.x86_64 : A high performance web server
```

告诉你：**有这个包、是干嘛的**。

---

# 三、查看的源是一样的吗？

## **答案：完全一样！**

- 它们**读取的都是你系统里配置的 YUM 仓库（/etc/yum.repos.d/）**
- 只是**展示方式不同**：
  - `list` → 展示**包名+版本**
  - `search` → 展示**包名+描述**

---

# 四、最实用的使用场景

### 你不知道包叫什么 → 用 **search**

```bash
yum search java
```

### 你知道包名，想看装了没、最新版是多少 → 用 **list**

```bash
yum list java
```

### 你想更新 → 用 **update**

```bash
yum update java
```

---

# 五、超级总结（最好记）

- **yum search**：找包（我忘了叫啥）
- **yum list**：看版本（装了没、最新是多少）
- **源一样，功能不一样**

需要我给你整理一份 **yum 最常用 10 条命令速查表** 吗？

## Windows CMD 切换盘符（最简直达）

### 1. 直接切盘（最常用）

直接输入**盘符+冒号**，回车

```cmd
# 切到 D盘
D:

# 切到 E盘
E:

# 切回 C盘
C:
```

---

### 2. 进入对应盘的文件夹

```cmd
# 先切盘
D:
# 再进入目录
cd D:\software\python
```

---

### 3. 一条命令跨盘进入目录（不用先切盘）

```cmd
cd /d D:\software\python
```

`/d` 参数：**同时切换盘符 + 进入文件夹**

---

### 补充小技巧

1. 查看当前目录文件
   
   ```cmd
   dir
   ```

2. 返回上一级
   
   ```cmd
   cd ..
   ```

3. 回到根目录
   
   ```cmd
   cd \
   ```

---

### 举个完整例子

```cmd
C:\Users\Admin> D:
D:\> cd project
D:\project>
```

全程就这么简单，不用复杂命令。

我给你用**最简单、最直白**的方式讲清楚：
**`curl -o` 就是：下载文件 + 自定义保存名字**

# 一、`curl -o` 是什么？

`-o` = **output**（输出）
作用：
**从网上下载文件，并且自己指定保存的文件名**

---

# 二、完整格式

```bash
curl -o 你想要的文件名 下载链接
```

---

# 三、最直观例子

比如下载一个图片：

```bash
curl -o test.jpg https://example.com/abc123.jpg
```

- 下载下来的文件 **不叫 abc123.jpg**
- 而是 **你指定的 test.jpg**

---

# 四、`-o` 和 `-O`（大写）的区别（超级重要）

## 1. **小写 -o** → 自定义文件名

```bash
curl -o myfile.zip https://xxx.com/file.zip
```

## 2. **大写 -O** → 用原来的文件名

```bash
curl -O https://xxx.com/file.zip
```

保存后文件名就是 **file.zip**

---

# 五、Windows CMD 里也能用吗？

能用！
Windows 10/11 自带 curl，直接 CMD 运行：

```cmd
curl -o aa.txt https://www.baidu.com
```

---

# 六、总结（10秒记住）

* **`curl -o 新名字 链接`** → 下载并改名
* **`curl -O 链接`** → 下载，用原名

需要我给你演示 **下载 Python、安装包、替换 yum源** 这种实战 curl 命令吗？

### 结论

**不加 `-o` / `-O`**：
curl 会把**内容直接输出到终端屏幕**，不会保存成文件。

---

## 1. 示例对比

### ① 不加参数（默认行为）

```bash
curl https://www.baidu.com
```

✅ 效果：
页面 HTML 代码**直接刷屏打印在控制台**
❌ 不会生成任何文件

### ② 小写 `-o`：指定保存文件名

```bash
curl -o baidu.html https://www.baidu.com
```

保存为：`baidu.html`

### ③ 大写 `-O`：自动使用远程原文件名

```bash
curl -O https://xxx.com/nginx.rpm
```

保存为：`nginx.rpm`

---

## 2. 拓展高频参数

- `curl -s 链接`：静默模式，不输出多余日志，只打内容
- `curl -f`：下载失败不显示杂乱错误
- `curl -L`：自动跟随重定向（必加，很多网址需要）

---

## 3. 最简记忆

1. 只想**看内容** → 直接 `curl 网址`
2. 想**存为自定义名字** → `curl -o 文件名 链接`
3. 想**直接下载原文件** → `curl -O 链接`

# Linux zip 压缩/解压命令大全（最常用、直接能用）

我给你整理**最实用、最常用**的 Linux zip 命令，不用记复杂参数，直接复制就能用。

## 一、安装 zip/unzip（没装的话先执行）

```bash
# CentOS/RHEL
sudo yum install zip unzip -y
# Ubuntu/Debian
sudo apt install zip unzip -y
```

---

## 二、最常用命令（直接背这几组）

### 1. 压缩文件/文件夹

```bash
zip -r 压缩包名.zip 要压缩的文件/目录
```

**示例**：把 `test` 文件夹压缩成 `test.zip`

```bash
zip -r test.zip test/
```

- `-r` = 递归压缩（文件夹必须加）

---

### 2. 解压 zip

```bash
unzip 压缩包名.zip
```

解压到**指定目录**（最常用）：

```bash
unzip test.zip -d /opt/myfolder/
```

---

### 3. 带密码压缩（重要）

```bash
zip -r -e 加密包.zip 文件夹/
```

执行后会让你输入两次密码。
---

### 4. 查看 zip 里有什么（不解压）

```bash
unzip -l test.zip
```

---

### 5. 解压时覆盖已存在文件

```bash
unzip -o test.zip
```

---

## 三、进阶常用（工作必备）

### 1. 压缩时排除某些文件/目录

```bash
zip -r test.zip test/ -x "test/log/*" "test/tmp/*"
```

- `-x` = 排除不压缩的内容
  
  ### 2. 压缩率更高（更小体积）
  
  ```bash
  zip -r -9 包名.zip 目录/
  ```

- `-9` = 最高压缩（速度慢一点）
  
  ### 3. 解压时不覆盖文件（安全）
  
  ```bash
  unzip -n test.zip
  ```
  
  ### 4. 检查压缩包是否损坏
  
  ```bash
  unzip -t test.zip
  ```

---

## 四、快速记忆表（最简单）

| 功能      | 命令                     |
| ------- | ---------------------- |
| 压缩文件夹   | `zip -r a.zip dir/`    |
| 解压      | `unzip a.zip`          |
| 解压到指定目录 | `unzip a.zip -d /path` |
| 加密压缩    | `zip -r -e a.zip dir/` |
| 查看内容    | `unzip -l a.zip`       |
| 强制覆盖解压  | `unzip -o a.zip`       |

---

### 总结

1. **压缩**必须加 `-r`（文件夹）
2. **解压**用 `unzip`，指定目录加 `-d`
3. **加密**用 `-e`
4. 日常用前 4 条就够 99% 场景
   你需要我给你做**一键压缩/解压脚本**吗？我可以直接写给你。

# Linux dd 命令超全常用用法

**dd = 数据拷贝神器，备份、克隆、测速、写盘、清空都用它**
语法：`dd if=源文件 of=目标文件 [参数]`

## 一、必记参数

- `if=` 输入文件（源）
- `of=` 输出文件（目标）
- `bs=` 单次读写块大小（推荐 `bs=1M` `bs=4M`）
- `count=` 拷贝块数量
- `status=none` 静默不输出日志
- `conv=noerror,sync` 出错继续、补齐块（克隆必加）

---

## 二、最常用实战命令

### 1. 全盘硬盘克隆（整盘备份）

```bash
dd if=/dev/sda of=/dev/sdb bs=4M conv=noerror,sync
```

sda 完整克隆到 sdb

### 2. 备份整个系统盘为镜像

```bash
dd if=/dev/sda of=/root/system-backup.img bs=4M
```

### 3. 镜像恢复写回硬盘

```bash
dd if=/root/system-backup.img of=/dev/sda bs=4M
```

### 4. 备份单个分区

```bash
dd if=/dev/sda1 of=/root/sda1.img bs=4M
```

### 5. 生成指定大小空文件

生成 1GB 空文件

```bash
dd if=/dev/zero of=test.bin bs=1M count=1024
```

### 6. 快速清空磁盘/分区

```bash
dd if=/dev/zero of=/dev/sda bs=4M status=none
```

### 7. 随机数据填充磁盘（擦除数据）

```bash
dd if=/dev/urandom of=/dev/sda bs=4M
```

### 8. 测试硬盘读写速度

**写速度**

```bash
dd if=/dev/zero of=tmp.test bs=1M count=500 oflag=direct
```

**读速度**

```bash
dd if=tmp.test of=/dev/null bs=1M count=500 iflag=direct
```

### 9. 制作 Linux 启动 U 盘

```bash
dd if=xxx.iso of=/dev/sdb bs=4M status=progress
```

### 10. 只拷贝前100MB数据

```bash
dd if=a.txt of=b.txt bs=1M count=100
```

### 11. 跳过开头部分数据拷贝

`skip=跳过源多少块`

```bash
dd if=src.bin of=dst.bin bs=1M skip=50
```

### 12. 静默执行（无刷屏）

```bash
dd if=xx.iso of=/dev/sdb bs=4M status=none
```

---

## 三、进度查看（dd 运行中看速度）

新开终端执行：

```bash
watch -n1 pgrep -x dd | xargs -I {} kill -USR1 {}
```

## 四、超级危险警告（必看）

1. **of=写反硬盘直接全盘清空，无法恢复**

2. 一定要先 `lsblk` 确认 `/dev/sda /dev/sdb` 哪个是目标盘

3. 虚拟机/服务器慎用 dd 擦盘
   
   ## 五、最简记忆口诀
   
   **if读源，of写目标，bs设大小，count定数量**
   需要我给你整理 **dd 做系统备份一键脚本** 吗？

# apt update 和 apt upgrade 区别（极简直白版）

## 1. 核心一句话

- **apt update**：**更新软件源索引**（查哪些软件能更）

- **apt upgrade**：**真正升级已装软件包**（下载安装更新）
  
  ## 2. 详细区别
  
  ### apt update
1. 只**刷新本地软件清单**

2. 去阿里云/官方源对比版本

3. 告诉你：哪些包可升级、多少个能更

4. **不下载、不安装、不改动系统**

5. 执行速度快
   
   ### apt upgrade

6. 基于刚才 `update` 获取的清单

7. **自动下载 + 安装所有可更新包**

8. 升级现有软件、内核、依赖

9. **不会删除任何旧包**（安全）

10. 耗流量、耗时间
    
    ## 3. 还有一个：apt full-upgrade
- 会**删除多余旧依赖**，大版本更新用

- 日常别乱用，容易删坏环境 
  
  ## 4. 标准固定顺序（必记）
  
  ```bash
  # 1. 更新索引
  sudo apt update
  # 2. 升级所有软件
  sudo apt upgrade -y
  ```
  
  ## 5. 最简单总结
1. **update = 查更新**
2. **upgrade = 装更新**
3. 先查再装，缺一不可
   需要我给你WSL一键更新提速命令吗？

# nvm 安装 & 管理 Node.js 最全教程（Windows/WSL/Linux 通用）

我给你**最简单、不踩坑**的版本，直接跟着做就能用。

## 一、先搞懂 nvm 是什么

- **nvm = Node Version Manager**
- 作用：**一台电脑随意切换多个 Node.js 版本**
- 比如：14、16、18、20 随便切，互不干扰

---

# 二、Windows 安装 nvm（最常用）

## 1. 下载安装包

去 GitHub 下载最新版：
👉 https://github.com/coreybutler/nvm-windows/releases
下载：`nvm-setup.exe`

## 2. 安装

一路下一步即可，**路径不要有中文、空格**

## 3. 检查是否安装成功

打开 **CMD / PowerShell** 输入：

```bash
nvm version
```

显示版本号 = 成功
---

# 三、WSL / Linux 安装 nvm

复制这一行直接运行：

```bash
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.7/install.sh | bash
```

安装完关闭终端重新打开，再检查：

```bash
nvm --version
```

---

# 四、nvm 最常用命令（背这 8 条就够）

## 1. 查看可安装的 Node 版本

```bash
nvm list available
```

## 2. 安装指定版本（推荐 LTS 稳定版）

```bash
nvm install 20
```

或

```bash
nvm install 18
```

## 3. 查看已安装的所有版本

```bash
nvm list
```

## 4. 切换使用某个版本

```bash
nvm use 20
```

## 5. 设置默认版本（开机自动用）

```bash
nvm alias default 20
```

## 6. 卸载某个版本

```bash
nvm uninstall 18
```

## 7. 查看当前使用版本

```bash
node -v
npm -v
```

## 8. 安装最新版

```bash
nvm install node
```

---

# 五、最实用的一套流程（直接复制）

```bash
# 安装 Node 20
nvm install 20
# 使用 Node 20
nvm use 20
# 设为默认
nvm alias default 20
# 检查
node -v
npm -v
```

---

# 六、常见问题

### 1. nvm use 报错

- Windows：**用管理员打开终端**

- Linux：重新开终端
  
  ### 2. node 命令找不到
  
  安装完必须执行：
  
  ```bash
  nvm use 版本号
  ```
  
  ### 3. 安装慢/失败
  
  国内镜像加速（Windows 直接用）：
  
  ```bash
  nvm node_mirror https://npmmirror.com/mirrors/node/
  nvm npm_mirror https://npmmirror.com/mirrors/npm/
  ```

---

## 超级简单总结

- **nvm install** 安装
- **nvm use** 切换
- **nvm list** 查看
- 想装哪个版本就装哪个版本
  需要我给你配 **npm 国内加速源** 吗？下载依赖超快那种！

# SSH 常用命令完整用法（新手直接复制用）

SSH 是**远程登录 Linux 服务器**、**文件传输**、**执行命令**的标准工具，我给你整理最常用、最实用的命令，一看就会、一用就通。

## 一、最基础：远程登录服务器

### 1. 默认端口 22 登录

```bash
ssh 用户名@服务器IP
```

示例：

```bash
ssh root@192.168.1.100
```

然后输入密码即可登录。

### 2. 非默认端口登录（比如端口 2222）

```bash
ssh -p 端口号 用户名@服务器IP
```

示例：

```bash
ssh -p 2222 root@192.168.1.100
```

---

## 二、直接在远程执行命令（不用登录）

```bash
ssh 用户名@IP "要执行的命令"
```

示例：查看远程服务器磁盘空间

```bash
ssh root@192.168.1.100 "df -h"
```

---

## 三、文件传输：scp 命令（最常用）

### 1. 本地文件 → 远程服务器

```bash
scp 本地文件 用户名@IP:远程路径
```

示例：把本地 `test.txt` 传到远程 `/root` 目录

```bash
scp test.txt root@192.168.1.100:/root
```

### 2. 远程文件 → 本地

```bash
scp 用户名@IP:远程文件路径 本地路径
```

示例：把远程 `/root/test.txt` 下载到本地当前目录

```bash
scp root@192.168.1.100:/root/test.txt ./
```

### 3. 传输整个文件夹（加 -r）

```bash
scp -r 文件夹路径 用户名@IP:目标路径
```

---

## 四、免密登录（不用每次输密码）

### 1. 本地生成密钥（只需一次）

```bash
ssh-keygen
```

一路回车即可。

### 2. 把公钥传到服务器

```bash
ssh-copy-id 用户名@IP
```

示例：

```bash
ssh-copy-id root@192.168.1.100
```

之后再登录 **直接进，不用输密码**。
---

## 五、SSH 其他实用命令

### 1. 查看 SSH 连接状态

```bash
who
```

或

```bash
w
```

### 2. 断开 SSH 连接

直接输入：

```bash
exit
```

或按快捷键 `Ctrl + D`

### 3. 后台保持连接（防止掉线）

```bash
ssh -o ServerAliveInterval=60 用户名@IP
```

### 4. 远程挂载目录（把服务器文件夹变成本地文件夹）

```bash
sshfs 用户名@IP:远程目录 本地空文件夹
```

---

## 六、常见问题

1. **连接超时**：检查 IP、端口、防火墙
2. **拒绝连接**：服务器没开 SSH 服务
3. **密码正确但登不上**：检查 SSH 配置是否允许密码登录

---

### 总结

- 登录：`ssh user@ip`
- 非默认端口：`ssh -p 端口 user@ip`
- 传文件：`scp 文件 user@ip:路径`
- 免密登录：`ssh-keygen` + `ssh-copy-id`
  需要我给你做一份 **Windows / Mac / Linux 三平台 SSH 速查表**吗？

# SSH免密登录原理

核心：**非对称密钥加密认证**，公私钥配对校验，无需明文密码

## 1. 密钥成对生成

本地生成一组密钥

- **私钥**：id_rsa，留存本机，绝不外泄

- **公钥**：id_rsa.pub，可公开分发
  
  ## 2. 公钥上传服务器
  
  把本地公钥内容，存入服务器`~/.ssh/authorized_keys`文件，相当于给本机登记授信
  
  ## 3. 认证交互流程
1. 客户端发起登录请求

2. 服务器随机生成一串**挑战验证码**

3. 服务器用本地存储的**客户端公钥**加密验证码，发回客户端

4. 客户端用自身**私钥**解密验证码，回传结果

5. 服务器比对结果一致，判定身份合法，直接放行登录
   
   ## 4. 关键特性
- 加密不可逆：公钥加密的数据，只能对应私钥解开

- 全程无密码明文传输，安全性远高于密码登录

- 一台私钥可授信多台服务器，一台服务器可接纳多个客户端公钥
  
  ## 5. 简易对比

- 密码登录：核对字符串，明文风险高

- 密钥免密：加密挑战校验，只认配对密钥

# 核心答案

服务器按**客户端账号+密钥指纹**匹配，逐个遍历授权公钥比对校验

## 1. 存储位置

服务器对应用户家目录下：`~/.ssh/authorized_keys`
文件里**一行对应一个客户端公钥**，存着所有授信设备公钥。

## 2. 匹配流程

1. 客户端连接时，主动带上自身**私钥对应的公钥指纹**

2. 服务器读取当前登录账号的`authorized_keys`

3. 逐行取出公钥，对比客户端发来的指纹

4. 找到完全匹配的那一条，就用该公钥做加密挑战验证

5. 匹配失败则换下一个，全部不符就退回密码登录
   
   ## 3. 补充要点
- 同一账号可存放几十上百个不同设备公钥，互不干扰

- 指纹是公钥唯一标识，快速筛选不用完整比对密钥串

- 密钥权限异常（文件/文件夹权限过宽）会直接拒绝匹配校验
  
  ## 4. 权限硬性规则（匹配失效常见原因）
  
  ```
  # 目录权限必须700
  chmod 700 ~/.ssh
  # 密钥文件必须600
  chmod 600 ~/.ssh/authorized_keys
  ```
  
  权限超标，服务器不会读取里面任何公钥。