# 如何保护 Linux 服务器安全

一份不断完善的 Linux 服务器安全加固指南，希望能同时教会你一些关于安全的知识及其重要性。

[![CC-BY-SA](https://i.creativecommons.org/l/by-sa/4.0/88x31.png)](#license)

## 目录

- [简介](#简介)
  - [指南目标](#指南目标)
  - [为什么要保护你的服务器](#为什么要保护你的服务器)
  - [为什么还需要一份指南](#为什么还需要一份指南)
  - [其他指南推荐](#其他指南推荐)
  - [待办事项](#待办事项)
- [指南概述](#指南概述)
  - [关于本指南](#关于本指南)
  - [我的使用场景](#我的使用场景)
  - [编辑配置文件——给懒人的方法](#编辑配置文件给懒人的方法)
  - [贡献指南](#贡献指南)
- [开始之前](#开始之前)
  - [明确你的安全原则](#明确你的安全原则)
  - [选择 Linux 发行版](#选择-linux-发行版)
  - [安装 Linux](#安装-linux)
  - [安装前后要求](#安装前后要求)
  - [其他重要注意事项](#其他重要注意事项)
  - [使用 Ansible Playbook 加固你的 Linux 服务器](#使用-ansible-playbook-加固你的-linux-服务器)
- [SSH 服务器](#ssh-服务器)
  - [修改 SSH 配置前的重要提示](#修改-ssh-配置前的重要提示)
  - [SSH 公钥/私钥](#ssh-公钥私钥)
  - [创建 SSH 用户组用于 AllowGroups](#创建-ssh-用户组用于-allowgroups)
  - [加固 /etc/ssh/sshd_config](#加固-etcsshsshd_config)
  - [移除短 Diffie-Hellman 密钥](#移除短-diffie-hellman-密钥)
  - [为 SSH 启用双因素认证 (2FA/MFA)](#为-ssh-启用双因素认证-2famfa)
- [基础安全](#基础安全)
  - [限制可以使用 sudo 的用户](#限制可以使用-sudo-的用户)
  - [限制可以使用 su 的用户](#限制可以使用-su-的用户)
  - [使用 FireJail 在沙盒中运行应用程序](#使用-firejail-在沙盒中运行应用程序)
  - [NTP 客户端](#ntp-客户端)
  - [保护 /proc](#保护-proc)
  - [强制账户使用安全密码](#强制账户使用安全密码)
  - [自动安全更新和警报](#自动安全更新和警报)
  - [更安全的随机熵池 (WIP)](#更安全的随机熵池-wip)
  - [添加应急/备用/伪造密码登录安全系统](#添加应急备用伪造密码登录安全系统)
- [网络](#网络)
  - [使用 UFW（简易防火墙）](#使用-ufw简易防火墙)
  - [使用 PSAD 进行 iptables 入侵检测与防御](#使用-psad-进行-iptables-入侵检测与防御)
  - [使用 Fail2Ban 进行应用入侵检测与防御](#使用-fail2ban-进行应用入侵检测与防御)
  - [使用 CrowdSec 进行应用入侵检测与防御](#使用-crowdsec-进行应用入侵检测与防御)
- [审计](#审计)
  - [使用 AIDE 进行文件/文件夹完整性监控 (WIP)](#使用-aide-进行文件文件夹完整性监控-wip)
  - [使用 ClamAV 进行反病毒扫描 (WIP)](#使用-clamav-进行反病毒扫描-wip)
  - [使用 Rkhunter 进行 Rootkit 检测 (WIP)](#使用-rkhunter-进行-rootkit-检测-wip)
  - [使用 chrootkit 进行 Rootkit 检测 (WIP)](#使用-chrootkit-进行-rootkit-检测-wip)
  - [logwatch - 系统日志分析和报告工具](#logwatch---系统日志分析和报告工具)
  - [ss - 查看服务器正在监听的端口](#ss---查看服务器正在监听的端口)
  - [Lynis - Linux 安全审计工具](#lynis---linux-安全审计工具)
  - [OSSEC - 主机入侵检测系统](#ossec---主机入侵检测系统)
- [危险区域](#危险区域)
  - [Linux 内核 sysctl 加固](#linux-内核-sysctl-加固)
  - [使用密码保护 GRUB](#使用密码保护-grub)
  - [禁用 root 登录](#禁用-root-登录)
  - [更改默认 umask](#更改默认-umask)
  - [孤立软件包清理](#孤立软件包清理)
- [其他杂项](#其他杂项)
  - [使用 MSMTP 发送邮件（简易方式）](#使用-msmtp-发送邮件简易方式)
  - [使用 Gmail 和 Exim4 作为 MTA 并启用隐式 TLS](#使用-gmail-和-exim4-作为-mta-并启用隐式-tls)
  - [分离 iptables 日志文件](#分离-iptables-日志文件)
- [补充内容](#补充内容)
  - [联系作者](#联系作者)
  - [有用的链接](#有用的链接)
  - [致谢](#致谢)
  - [许可与版权](#许可与版权)

（目录由 [nGitHubTOC](https://imthenachoman.github.io/nGitHubTOC/) 生成）

---

## 简介

### 指南目标

本指南的目的是教你如何保护 Linux 服务器的安全。

你可以做很多事情来保护 Linux 服务器的安全，本指南将尽可能多地涵盖这些内容。随着作者的学习或社区的[贡献](#贡献指南)，会添加更多的主题和内容。

本指南的 Ansible playbook 版本可在 [How To Secure A Linux Server With Ansible](https://github.com/moltenbit/How-To-Secure-A-Linux-Server-With-Ansible) 找到（由 [moltenbit](https://github.com/moltenbit) 提供）。

### 为什么要保护你的服务器

假设你正在使用本指南，说明你很可能已经理解了安全为什么重要。这个话题本身就非常广泛，深入探讨超出了本指南的范围。如果你不知道这个问题的答案，建议你先自行研究一下。

从高层次来看，当一个设备（如服务器）处于公共网络中——即对外部世界可见——它就会成为恶意行为者的攻击目标。一个未受保护的设备是恶意行为者的游乐场，他们可能想要访问你的数据，或将你的服务器用作大规模 DDoS 攻击的另一个节点。

更糟糕的是，如果没有良好的安全措施，你可能永远不会知道你的服务器是否已被入侵。恶意行为者可能已经获得了你服务器的未授权访问并复制了你的数据，而没有改变任何东西，所以你将永远不会知晓。或者你的服务器可能已经成为 DDoS 攻击的一部分，你也无从得知。看看新闻中许多大规模数据泄露的事件——相关公司往往在恶意行为者早已离开之后才发现数据泄露或入侵。

与普遍看法相反，恶意行为者并不总是想要改变什么或[为了赎金而锁定你的数据](https://en.wikipedia.org/wiki/Ransomware)。有时他们只是想要你服务器上的数据用于他们的数据仓库（大数据中有大利润），或者秘密地利用你的服务器进行他们的恶意目的。

### 为什么还需要一份指南

本指南可能看起来是重复/不必要的，因为网上有无数的文章告诉你[如何保护 Linux](https://duckduckgo.com/?q=how+to+secure+linux&t=ffab&atb=v151-7&ia=web)，但这些信息分散在不同的文章中，涵盖不同的内容，以不同的方式呈现。谁有时间翻阅数百篇文章呢？

在作者为自己的 Debian 构建进行研究时，做了很多笔记。最终意识到，结合已知的知识和正在学习的内容，手头已经有了编写一份指南的素材。于是决定将其放到网上，希望能帮助其他人**学习**并**节省时间**。

作者从未找到过一份能涵盖所有内容的指南——本指南正是为此尝试。

本指南涵盖的许多内容可能相当基础/简单，但大多数人并不会每天安装 Linux，很容易忘记那些基础的东西。

### 其他指南推荐

有许多由专家、行业领导者和发行版本身提供的指南。将它们全部包含进来是不现实的，有时也违反版权。建议在开始本指南之前先查看它们：

- [互联网安全中心 (CIS)](https://www.cisecurity.org/) 提供了详尽、行业信赖、逐步操作的[基准指南](https://www.cisecurity.org/cis-benchmarks/)，用于保护多种 Linux 发行版。建议先阅读本指南，然后再阅读 CIS 的指南。这样他们的建议将优先于本指南中的任何内容。
- 针对特定发行版的加固/安全指南，请查阅你的发行版文档。
- https://security.utexas.edu/os-hardening-checklist/linux-7 - Red Hat Enterprise Linux 7 加固清单
- https://wiki.archlinux.org/index.php/Security - 许多人推荐的 Arch Linux 安全指南
- https://securecompliance.co/linux-server-hardening-checklist/
- https://seifried.org/lasg/

### 待办事项

- [ ] [为 Fail2ban 自定义 Jail](#)
- [ ] MAC（强制访问控制）和 Linux 安全模块 (LSM)，包括 SELinux 和 AppArmor
- [ ] 磁盘加密
- [ ] Rkhunter 和 chrootkit 完善
- [ ] 日志的传输/备份
- [ ] CIS-CAT 集成
- [ ] debsums - 验证已安装软件包的完整性

---

## 指南概述

### 关于本指南

本指南：

- **是**一个进行中的工作。
- **专注于**家庭环境下的 Linux 服务器。这里所有的概念/建议也适用于更大/更专业的环境，但这些场景需要更高级和更专业的配置，超出了本指南的范围。
- **不会**教你 Linux 基础知识、如何[安装 Linux](#安装-linux) 或如何使用 Linux。如果你是 Linux 新手，请查看 https://linuxjourney.com/。
- **旨在**做到[与 Linux 发行版无关](#选择-linux-发行版)。
- **不会**教你关于安全需要知道的一切，也不会涉及系统/服务器安全的所有方面。例如，物理安全就超出了本指南的范围。
- **不会**深入讨论程序/工具的工作原理及其细节。大多数本指南引用的程序/工具都功能强大且高度可配置。目标是覆盖最基本的必要内容——足够激发你的兴趣，让你渴望去学习更多。
- **旨在**通过提供可复制粘贴的代码来简化操作。你可能需要在粘贴前修改命令，所以请准备好你最喜欢的[文本编辑器](https://notepad-plus-plus.org/)。
- **按照**作者认为逻辑合理的顺序组织——例如，在安装防火墙之前先保护 SSH。因此，本指南旨在按照呈现的顺序执行，但并非必须如此。如果以不同顺序操作，请小心——某些部分需要先完成前面的部分。

### 我的使用场景

服务器有很多类型和不同的使用场景。虽然作者希望本指南尽可能通用，但有些内容可能不适用于所有/其他使用场景。请在阅读本指南时做出最佳判断。

为了帮助理解本指南中涉及的许多主题的背景，作者的使用场景/配置是：

- 一台台式机级别的计算机
- 带有单个网卡 (NIC)
- 连接到消费级路由器
- 从 ISP 获取动态 WAN IP
- WAN+LAN 使用 IPv4
- LAN 使用 [NAT](https://en.wikipedia.org/wiki/Network_address_translation)（网络地址转换）
- 希望能够从未知的计算机和未知的位置（例如朋友家）远程 SSH 到服务器

### 编辑配置文件——给懒人的方法

作者非常懒，不喜欢手动编辑文件。同时也假设其他人也是如此。:)

因此，在可能的情况下，本指南提供了 `code` 片段来快速完成所需的操作，例如在配置文件中添加或更改某行。

这些 `code` 片段使用 `echo`、`cat`、`sed`、`awk` 和 `grep` 等基本命令。每个命令/部分如何工作超出了本指南的范围——`man` 手册是你的好朋友。

**注意**：这些 `code` 片段不会验证/确认更改是否生效——即该行是否确实被添加或更改。验证部分留给你来执行。本指南中的步骤确实包含对所有将要更改的文件进行备份。

并非所有更改都可以用 `code` 片段自动完成。那些更改需要传统的、老式的手动编辑。例如，你不能只是向 [INI](https://en.wikipedia.org/wiki/INI_file) 类型的文件追加一行。请使用你[最喜欢的](https://en.wikipedia.org/wiki/Vi) Linux 文本编辑器。

### 贡献指南

作者希望将本指南放在 [GitHub](http://www.github.com) 上，以便于协作。贡献的人越多，这份指南就会变得越好、越完整。

要贡献，你可以 fork 项目并提交 pull request，或提交一个 [new issue](https://github.com/imthenachoman/How-To-Secure-A-Linux-Server/issues/new)。

---

## 开始之前

### 明确你的安全原则

在开始之前，你需要明确你的安全原则。你的[威胁模型](https://en.wikipedia.org/wiki/Threat_model)是什么？需要考虑的一些事情：

- 你为什么要保护你的服务器？
- 你想要多少安全或不想要多少安全？
- 你愿意为了安全妥协多少便利，反之亦然？
- 你想要防范哪些威胁？你的具体情况是什么？例如：
  - 对你的服务器/网络的物理访问是否是可能的攻击向量？
  - 你是否会在路由器上开放端口以便从家外访问你的服务器？
  - 你是否会在服务器上托管一个文件共享，该共享将被挂载到台式机上？台式机被感染并进而感染服务器的可能性有多大？
  - 如果你的安全措施将你锁在服务器之外，你是否有恢复的方法？例如你[禁用了 root 登录](#禁用-root-登录)或[为 GRUB 设置了密码保护](#使用密码保护-grub)。

这些只是**需要思考的几件事情**。在开始加固你的服务器之前，你需要了解你想要防范什么以及为什么要防范，这样你才知道需要做什么。

### 选择 Linux 发行版

本指南旨在与发行版无关，因此用户可以使用[任何发行版](https://distrowatch.com/)。话虽如此，有几件事需要牢记：

你想要的发行版：

- **是稳定的**。除非你喜欢在凌晨 2 点调试问题，否则你不会希望[无人值守升级](#自动安全更新和警报)或手动包/系统更新使你的服务器无法运行。但这也意味着你接受不运行最新、最好、最前沿的软件。
- **及时获得安全补丁更新**。你可以保护服务器上的一切，但如果核心操作系统或你运行的应用程序存在已知漏洞，你永远不会安全。
- **是你熟悉的**。如果你不了解 Linux，建议你先在某个发行版上练习一下，然后再尝试保护它。你应该对它感到舒适，知道如何操作，比如如何安装软件、配置文件在哪里等等。
- **有良好的支持**。即使是最有经验的管理员也时常需要帮助。有一个可以求助的地方将拯救你的理智。

### 安装 Linux

安装 Linux 超出了本指南的范围，因为每个发行版的方式不同，而且安装说明通常都有很好的文档记录。如果你需要帮助，请从你的发行版文档开始。不管是什么发行版，高层次的过程通常是这样的：

1. 下载 ISO 镜像
2. 刻录/复制/传输到你的安装介质（如 CD 或 USB 盘）
3. 从安装介质启动你的服务器
4. 按照提示完成安装

在适用的情况下，使用专家安装选项，这样你可以更严格地控制服务器上运行的内容。**只安装你绝对需要的东西。** 作者个人除了 SSH 之外不安装任何东西。另外，勾选磁盘加密选项。

### 安装前后要求

- 如果你要在路由器上开放端口以便从外部访问你的服务器，在网络加固完成并确保系统安全之前，请先禁用端口转发。
- 除非你所有操作都是物理连接到服务器进行的，否则你需要远程访问，所以请确保 SSH 正常工作。
- 保持你的系统最新（例如在基于 Debian 的系统上执行 `sudo apt update && sudo apt upgrade`）。
- 确保完成你特定设置所需的任务，如：
  - 配置网络
  - 在 `/etc/fstab` 中配置挂载点
  - 创建初始用户账户
  - 安装你需要的核心软件，如 `man`
  - 等等
- 你的服务器需要能够发送电子邮件，这样你才能收到重要的安全警报。如果你不打算设置邮件服务器，请查看[使用 Gmail 和 Exim4 作为 MTA 并启用隐式 TLS](#使用-gmail-和-exim4-作为-mta-并启用隐式-tls)。
- 建议在开始本指南之前**通读** [CIS 基准指南](https://www.cisecurity.org/cis-benchmarks/)，以便消化理解他们要说的内容。推荐的顺序是先阅读本指南，再阅读 CIS 的指南。这样他们的建议将优先于本指南中的任何内容。

### 其他重要注意事项

- 本指南是在 Debian 上编写和测试的。下面的大多数内容应该可以在其他发行版上运行。如果你发现有什么不适用，请[联系作者](#联系作者)。区分各发行版的主要因素是其包管理系统。由于作者使用 Debian，将提供适当的 `apt` 命令，这些命令应适用于所有[基于 Debian 的发行版](https://www.debian.org/derivatives/)。
- 文件路径和设置可能也略有不同——如果遇到问题，请查阅你的发行版文档。
- 在开始之前通读整份指南。你的使用场景和/或原则可能要求不做某些事情或改变顺序。
- 不要**盲目地**复制粘贴而不理解你粘贴的内容。某些命令需要根据你的需求进行修改后才能工作——例如用户名。

### 使用 Ansible Playbook 加固你的 Linux 服务器

本指南的 Ansible playbook 版本可在 [How To Secure A Linux Server With Ansible](https://github.com/moltenbit/How-To-Secure-A-Linux-Server-With-Ansible) 找到。

请确保根据你的需要编辑变量，并事先阅读所有任务以确认它不会损坏你的系统。运行 playbook 后，请确保所有设置都符合你的需求！

1. 安装 [Ansible](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html)

2. git clone [How To Secure A Linux Server With Ansible](https://github.com/moltenbit/How-To-Secure-A-Linux-Server-With-Ansible)

3. [创建 SSH 公钥/私钥](#ssh-公钥私钥)：

   ```
   ssh-keygen -t ed25519
   ```

4. 根据你的需求更改 *group_vars/variables.yml* 中的所有变量

5. 在运行 playbook 之前启用 SSH root 访问

6. 推荐：在你的系统上配置静态 IP 地址

7. 将你的系统 IP 地址添加到 *hosts.yml*

运行 requirements playbook：

```
ansible-playbook --inventory hosts.yml --ask-pass requirements-playbook.yml
```

运行主 playbook：

```
ansible-playbook --inventory hosts.yml --ask-pass main-playbook.yml
```

如果需要多次运行 playbook，请使用 SSH 密钥和新端口：

```
ansible-playbook --inventory hosts.yml -e ansible_ssh_port=SSH_PORT --key-file /PATH/TO/SSH/KEY main-playbook.yml
```

---

## SSH 服务器

### 修改 SSH 配置前的重要提示

强烈建议在**进行和应用 SSH 配置更改之前**，保持第二个终端连接到你的服务器。这样，如果你将自己锁在第一个终端会话之外，你还有一个已连接的会话可以修复问题。

感谢 [Sonnenbrand](https://github.com/Sonnenbrand) 提供的这个[建议](https://github.com/imthenachoman/How-To-Secure-A-Linux-Server/issues/56)。

### SSH 公钥/私钥

#### 为什么

使用 SSH 公钥/私钥比使用密码更安全。这也使得连接服务器更容易、更快速，因为你不需要输入密码。

#### 工作原理

从高层次来看，公钥/私钥通过使用一对密钥来验证身份：

1. 一个密钥，即**公钥**，**只能加密数据**，不能解密
2. 另一个密钥，即**私钥**，可以解密数据

对于 SSH，在客户端创建公钥和私钥。你需要保证两个密钥的安全，尤其是私钥。即使公钥本来就是要公开的，也要确保两个密钥都不会落入坏人之手。

当你连接到 SSH 服务器时，SSH 会在你正在连接的服务器上的 `~/.ssh/authorized_keys` 文件中查找与你正在连接的客户端匹配的公钥。请注意该文件位于你尝试连接的身份的**主文件夹**中。因此，在创建公钥后，你需要将其追加到 `~/.ssh/authorized_keys`。

公钥/私钥被认为更安全，因为你需要私钥才能建立 SSH 连接。如果你在 `/etc/ssh/sshd_config` 中设置了 `PasswordAuthentication no`，那么 SSH 就不会让你在没有私钥的情况下连接。

我们将会使用 Ed25519 密钥，根据 [https://linux-audit.com/](https://linux-audit.com/using-ed25519-openssh-keys-instead-of-dsa-rsa-ecdsa/)：

> 它使用椭圆曲线签名方案，提供比 ECDSA 和 DSA 更好的安全性。同时，它也具有很好的性能。

#### 目标

- Ed25519 公钥/私钥 SSH 密钥：
  - 私钥在你的客户端
  - 公钥在你的服务器

#### 步骤

1. 从你将要用来连接服务器的计算机（**客户端**），使用 `ssh-keygen` 创建 [Ed25519](https://linux-audit.com/using-ed25519-openssh-keys-instead-of-dsa-rsa-ecdsa/) 密钥：

   ```bash
   ssh-keygen -t ed25519
   ```

   **注意**：如果你设置了密码短语，每次使用此密钥连接服务器时都需要输入它，除非你使用 `ssh-agent`。

2. 现在你需要将客户端上的公钥 `~/.ssh/id_ed25519.pub` **追加**到服务器上的 `~/.ssh/authorized_keys` 文件。由于我们可能还在家里的局域网中，我们可能安全免受 [MIM](https://en.wikipedia.org/wiki/Man-in-the-middle_attack) 攻击，所以我们将使用 `ssh-copy-id` 来传输和追加公钥：

   ```bash
   ssh-copy-id user@server
   ```

### 创建 SSH 用户组用于 AllowGroups

#### 为什么

为了方便控制谁可以 SSH 到服务器。通过使用组，我们可以快速添加/删除账户到组中以快速允许或不允许 SSH 访问服务器。

#### 工作原理

我们将使用 SSH 配置文件 [`/etc/ssh/sshd_config`](#加固-etcsshsshd_config) 中的 [AllowGroups 选项](#allowgroups) 来告诉 SSH 服务器只允许属于特定 UNIX 组的用户通过 SSH 登录。不在该组中的任何人都无法通过 SSH 登录。

#### 步骤

1. 创建一个组：

   ```bash
   sudo groupadd sshusers
   ```

2. 将账户添加到该组：

   ```bash
   sudo usermod -a -G sshusers user1
   sudo usermod -a -G sshusers user2
   ```

   你需要为服务器上每个需要 SSH 访问的账户执行此操作。

### 加固 /etc/ssh/sshd_config

#### 为什么

SSH 是进入你服务器的一扇门。如果你在路由器上开放端口以便从家庭网络外部通过 SSH 访问服务器，这一点尤其重要。如果它没有得到适当的保护，恶意行为者可能利用它来获得对你的系统的未授权访问。

#### 步骤

1. 备份 OpenSSH 服务器的配置文件 `/etc/ssh/sshd_config` 并删除注释以便阅读：

   ```bash
   sudo cp --archive /etc/ssh/sshd_config /etc/ssh/sshd_config-COPY-$(date +"%Y%m%d%H%M%S")
   sudo sed -i -r -e '/^#|^$/ d' /etc/ssh/sshd_config
   ```

2. 编辑 `/etc/ssh/sshd_config`，找到并编辑或添加以下应该应用于任何配置/设置的选项：

   ```
   # 来自 Mozilla OpenSSH 指南的支持的主机密钥算法（按优先顺序排列）
   HostKey /etc/ssh/ssh_host_ed25519_key
   HostKey /etc/ssh/ssh_host_rsa_key
   HostKey /etc/ssh/ssh_host_ecdsa_key
   
   KexAlgorithms curve25519-sha256@libssh.org,ecdh-sha2-nistp521,ecdh-sha2-nistp384,ecdh-sha2-nistp256,diffie-hellman-group-exchange-sha256
   
   Ciphers chacha20-poly1305@openssh.com,aes256-gcm@openssh.com,aes128-gcm@openssh.com,aes256-ctr,aes192-ctr,aes128-ctr
   
   MACs hmac-sha2-512-etm@openssh.com,hmac-sha2-256-etm@openssh.com,hmac-sha2-512,hmac-sha2-256,umac-128@openssh.com
   
   # LogLevel VERBOSE 在登录时记录用户的密钥指纹，用于清晰的审计追踪
   LogLevel VERBOSE
   
   # 不允许用户设置环境变量
   PermitUserEnvironment no
   
   # 以日志级别 INFO 记录 SFTP 文件访问
   Subsystem sftp  internal-sftp -f AUTHPRIV -l INFO
   
   # 禁用 X11 转发（X11 非常不安全）
   X11Forwarding no
   
   # 禁用端口转发
   AllowTcpForwarding no
   AllowStreamLocalForwarding no
   GatewayPorts no
   PermitTunnel no
   
   # 不允许空密码登录
   PermitEmptyPasswords no
   
   # 忽略 .rhosts 和 .shosts
   IgnoreRhosts yes
   
   # 验证主机名是否匹配 IP
   UseDNS yes
   
   Compression no
   
   # TCP keepalive 是可欺骗的
   TCPKeepAlive no
   
   AllowAgentForwarding no
   PermitRootLogin no
   
   # 不允许 .rhosts 或 /etc/hosts.equiv
   HostbasedAuthentication no
   
   # OpenSSH 9.1 及更高版本：强制最小 RSA 密钥大小为 3072 位
   # RequiredRSASize 3072
   
   HashKnownHosts yes
   ```

3. 根据你的需求设置以下参数：

   | 设置                       | 有效值         | 示例                        | 描述                         |
   | -------------------------- | -------------- | --------------------------- | ---------------------------- |
   | **AllowGroups**            | 本地 UNIX 组名 | `AllowGroups sshusers`      | 允许 SSH 访问的组            |
   | **ClientAliveCountMax**    | 数字           | `ClientAliveCountMax 3`     | 无响应的最大客户端存活消息数 |
   | **ClientAliveInterval**    | 秒数           | `ClientAliveInterval 15`    | 响应请求的超时秒数           |
   | **LoginGraceTime**         | 秒数           | `LoginGraceTime 30`         | 登录超时前的等待秒数         |
   | **MaxAuthTries**           | 数字           | `MaxAuthTries 2`            | 允许尝试登录的最大次数       |
   | **MaxSessions**            | 数字           | `MaxSessions 2`             | 最大打开会话数               |
   | **MaxStartups**            | 数字           | `MaxStartups 2`             | 最大登录会话数               |
   | **PasswordAuthentication** | `yes` 或 `no`  | `PasswordAuthentication no` | 是否允许密码登录             |
   | **Port**                   | 任何可用端口号 | `Port 22`                   | `sshd` 监听的端口            |

4. 确保没有相互矛盾的重复设置。以下命令不应有输出：

   ```bash
   awk 'NF && $1!~/^(#|HostKey)/{print $1}' /etc/ssh/sshd_config | sort | uniq -c | grep -v ' 1 '
   ```

5. 重启 SSH：

   ```bash
   sudo service sshd restart
   ```

6. 你可以使用 `sshd -T` 验证配置是否生效：

   ```bash
   sudo sshd -T
   ```

### 移除短 Diffie-Hellman 密钥

#### 为什么

根据 [Mozilla 的 OpenSSH 6.7+ 指南](https://infosec.mozilla.org/guidelines/openssh#modern-openssh-67)，"所有使用的 Diffie-Hellman 模数至少应为 3072 位长"。

Diffie-Hellman 算法被 SSH 用来建立安全连接。模数（密钥大小）越大，加密越强。

#### 步骤

1. 备份 SSH 的 moduli 文件：

   ```bash
   sudo cp --archive /etc/ssh/moduli /etc/ssh/moduli-COPY-$(date +"%Y%m%d%H%M%S")
   ```

2. 移除短模数（保留 ≥ 3072 位的）：

   ```bash
   sudo awk '$5 >= 3071' /etc/ssh/moduli | sudo tee /etc/ssh/moduli.tmp
   sudo mv /etc/ssh/moduli.tmp /etc/ssh/moduli
   ```

### 为 SSH 启用双因素认证 (2FA/MFA)

#### 为什么

尽管 SSH 是一个相当好的安全卫士，但它仍然是一扇可见的门，恶意行为者可以看到并尝试暴力破解。要求两个因素增加了额外的安全层。

使用双因素认证 (2FA) / 多因素认证 (MFA) 要求任何人进入都拥有**两把**钥匙，这使得恶意行为者更难入侵。这两把钥匙是：

1. 他们的密码
2. 一个每 30 秒变化一次的 6 位数字令牌

没有两把钥匙，他们无法进入。

#### 工作原理

我们将使用 Google 的 libpam-google-authenticator PAM 模块来创建和验证 [TOTP](https://en.wikipedia.org/wiki/Time-based_One-time_Password_algorithm) 密钥。我们将告诉服务器的 SSH PAM 配置要求用户先输入密码，然后再输入他们的数字令牌。只有当一切都正确时，认证才会成功，用户才被允许登录。

#### 步骤

1. 安装 libpam-google-authenticator（基于 Debian 的系统）：

   ```bash
   sudo apt install libpam-google-authenticator
   ```

2. **确保你以想要启用 2FA/MFA 的身份登录**，然后执行 `google-authenticator` 创建必要的令牌数据：

   ```bash
   google-authenticator
   ```

   注意这是**以非 root 用户身份运行**的。对所有问题选择默认选项（大多数情况下是 y），并请记住保存紧急备用代码。

3. 备份 PAM 的 SSH 配置文件：

   ```bash
   sudo cp --archive /etc/pam.d/sshd /etc/pam.d/sshd-COPY-$(date +"%Y%m%d%H%M%S")
   ```

4. 在 `/etc/pam.d/sshd` 中添加以下行以启用它作为 SSH 的认证方法：

   ```
   auth       required     pam_google_authenticator.so nullok
   ```

5. 在 `/etc/ssh/sshd_config` 中添加或编辑以下行以告知 SSH 利用它：

   ```
   ChallengeResponseAuthentication yes
   ```

6. 重启 SSH：

   ```bash
   sudo service sshd restart
   ```

---

## 基础安全

### 限制可以使用 sudo 的用户

#### 为什么

sudo 允许账户以其他账户（包括 **root**）身份运行命令。我们希望确保只有我们需要的账户才能使用 sudo。

#### 步骤

1. 创建一个组：

   ```bash
   sudo groupadd sudousers
   ```

2. 将账户添加到该组：

   ```bash
   sudo usermod -a -G sudousers user1
   ```

3. 备份 sudo 的配置文件：

   ```bash
   sudo cp --archive /etc/sudoers /etc/sudoers-COPY-$(date +"%Y%m%d%H%M%S")
   ```

4. 使用 visudo 编辑 sudo 的配置文件：

   ```bash
   sudo visudo
   ```

5. 添加以下行以告知 sudo 仅允许 `sudousers` 组中的用户使用 sudo：

   ```
   %sudousers   ALL=(ALL:ALL) ALL
   ```

### 限制可以使用 su 的用户

#### 为什么

su 也允许账户以其他账户（包括 **root**）身份运行命令。我们希望确保只有我们需要的账户才能使用 su。

#### 步骤

1. 创建一个组：

   ```bash
   sudo groupadd suusers
   ```

2. 将账户添加到该组：

   ```bash
   sudo usermod -a -G suusers user1
   ```

3. 使只有此组中的用户才能执行 `/bin/su`：

   ```bash
   sudo dpkg-statoverride --update --add root suusers 4750 /bin/su
   ```

### 使用 FireJail 在沙盒中运行应用程序

#### 为什么

对于许多应用程序来说，在沙盒中运行绝对更好。浏览器（尤其是闭源浏览器）和电子邮件客户端强烈建议这样做。

#### 步骤

1. 安装软件：

   ```bash
   sudo apt install firejail firejail-profiles
   ```

   注意：对于 Debian 10 Stable，建议使用官方 Backport：

   ```bash
   sudo apt install -t buster-backports firejail firejail-profiles
   ```

2. 允许应用程序（安装在 `/usr/bin` 或 `/bin` 中）仅在沙盒中运行：

   ```bash
   sudo ln -s /usr/bin/firejail /usr/local/bin/firefox
   sudo ln -s /usr/bin/firejail /usr/local/bin/chromium
   sudo ln -s /usr/bin/firejail /usr/local/bin/thunderbird
   ```

3. 像平常一样运行应用程序，检查它是否在沙盒中运行：

   ```bash
   firejail --list
   ```

4. 恢复沙盒化的应用程序以像以前一样运行（例如 firefox）：

   ```bash
   sudo rm /usr/local/bin/firefox
   ```

### NTP 客户端

#### 为什么

许多安全协议利用时间。如果你的系统时间不正确，可能对你的服务器产生负面影响。NTP 客户端可以通过保持你的系统时间与[全球 NTP 服务器](https://en.wikipedia.org/wiki/Network_Time_Protocol)同步来解决这个问题。

> **注意：** 从 **Debian 13 (Trixie)** 开始，传统的 `ntp` 软件包已被移除。由于本指南仅将 NTP 用作**客户端**（同步服务器时钟），在 Debian 13+ 上推荐的方法是使用预安装的 `systemd-timesyncd`。

#### Debian 13 (Trixie) 及更高版本：systemd-timesyncd

1. 启用 NTP 同步：

   ```bash
   sudo timedatectl set-ntp true
   ```

2. 验证是否正常工作：

   ```bash
   timedatectl status
   ```

   你应该看到输出中显示 `NTP service: active` 和 `System clock synchronized: yes`。

3. 配置受信任的 NTP 服务器。编辑 `/etc/systemd/timesyncd.conf`：

   ```
   [Time]
   NTP=pool.ntp.org
   FallbackNTP=0.debian.pool.ntp.org 1.debian.pool.ntp.org 2.debian.pool.ntp.org
   ```

4. 重启服务以应用更改：

   ```bash
   sudo systemctl restart systemd-timesyncd
   ```

#### Debian 12 (Bookworm) 及更早版本：ntp 软件包

1. 安装 ntp：

   ```bash
   sudo apt install ntp
   ```

2. 备份配置并确保使用 `pool` 指令而非 `server` 指令。在 `/etc/ntp.conf` 中添加：

   ```
   pool pool.ntp.org iburst
   ```

3. 重启 ntp：

   ```bash
   sudo service ntp restart
   ```

### 保护 /proc

#### 为什么

引用 https://linux-audit.com/linux-system-hardening-adding-hidepid-to-proc/：

> 当查看 `/proc` 时你会发现很多文件和目录。其中许多只是数字，代表特定进程 ID (PID) 的信息。默认情况下，Linux 系统部署为允许所有本地用户查看所有这些信息。这包括其他用户的进程信息。这可能包含你不想与其他用户分享的敏感详细信息。通过应用一些文件系统配置调整，我们可以改变这种行为并提高系统的安全性。

#### 步骤

1. 备份 `/etc/fstab`：

   ```bash
   sudo cp --archive /etc/fstab /etc/fstab-COPY-$(date +"%Y%m%d%H%M%S")
   ```

2. 在 `/etc/fstab` 中添加以下行，使 `/proc` 以 `hidepid=2` 挂载（用户只能看到关于自己进程的信息）：

   ```
   proc     /proc     proc     defaults,hidepid=2     0     0
   ```

3. 重启系统或重新挂载：

   ```bash
   sudo mount -o remount,hidepid=2 /proc
   ```

### 强制账户使用安全密码

#### 工作原理

我们将告诉 PAM 的密码任务将请求的新密码传递给 libpam-pwquality，以确保它满足我们的要求。

#### 步骤

1. 安装 libpam-pwquality（基于 Debian 的系统）：

   ```bash
   sudo apt install libpam-pwquality
   ```

2. 在 `/etc/pam.d/common-password` 中将 pam_pwquality.so 行更改为：

   ```
   password        requisite                       pam_pwquality.so retry=3 minlen=10 difok=3 ucredit=-1 lcredit=-1 dcredit=-1 ocredit=-1 maxrepeat=3 gecoschec
   ```

   上述选项的含义：

   - `retry=3` = 返回错误前提示用户 3 次
   - `minlen=10` = 密码的最小长度
   - `dcredit=-1` = 必须至少有一个**数字**
   - `ucredit=-1` = 必须至少有一个**大写字母**
   - `lcredit=-1` = 必须至少有一个**小写字母**
   - `ocredit=-1` = 必须至少有一个**非字母数字字符**
   - `difok=3` = 新密码中至少有 3 个字符不能出现在旧密码中
   - `maxrepeat=3` = 最多允许 3 个重复字符
   - `gecoschec` = 不允许使用包含账户名的密码

### 自动安全更新和警报

#### 为什么

保持服务器更新到最新的**关键安全补丁和更新**非常重要。否则你将面临已知安全漏洞的风险，恶意行为者可能利用这些漏洞获得对你的服务器的未授权访问。

#### 基于 Debian 的系统

在基于 Debian 的系统上，你可以使用：

- **unattended-upgrades** 自动进行你想要的系统更新（例如关键安全更新）
- **apt-listchanges** 获取软件包更改的详细信息
- **apticron** 获取待处理软件包更新的电子邮件通知

我们将使用 unattended-upgrades 来应用**关键安全补丁**。

#### 步骤

1. 安装软件包：

   ```bash
   sudo apt install unattended-upgrades apt-listchanges apticron
   ```

2. 创建文件 `/etc/apt/apt.conf.d/51myunattended-upgrades` 并添加配置：

   ```
   APT::Periodic::Enable "1";
   APT::Periodic::Update-Package-Lists "1";
   APT::Periodic::Download-Upgradeable-Packages "1";
   APT::Periodic::AutocleanInterval "7";
   APT::Periodic::Unattended-Upgrade "1";
   
   Unattended-Upgrade::Origins-Pattern {
         "o=Debian,a=stable";
         "o=Debian,a=stable-updates";
         "origin=Debian,codename=${distro_codename},label=Debian-Security";
   };
   
   Unattended-Upgrade::AutoFixInterruptedDpkg "true";
   Unattended-Upgrade::InstallOnShutdown "false";
   Unattended-Upgrade::Mail "root";
   Unattended-Upgrade::MailOnlyOnError "false";
   Unattended-Upgrade::Remove-Unused-Dependencies "true";
   Unattended-Upgrade::Remove-New-Unused-Dependencies "true";
   Unattended-Upgrade::Automatic-Reboot "true";
   Unattended-Upgrade::Automatic-Reboot-WithUsers "true";
   ```

3. 测试配置：

   ```bash
   sudo unattended-upgrade -d --dry-run
   ```

4. 配置 apt-listchanges：

   ```bash
   sudo dpkg-reconfigure apt-listchanges
   ```

### 添加应急/备用/伪造密码登录安全系统

#### 为什么

一个很好的工具，用于添加额外的密码安全性，以防范物理攻击（面对面的勒索/抢劫/袭击手段）。

#### 工作原理

pamduress 将为用户 X 添加一个备用密码（应急密码），当这个密码匹配时将启动一个脚本。实际例子：如果攻击者破解了一个虚拟/应急密码并以 sudoer 用户 'admin' 登录，与此密码关联的脚本将删除所有文件、配置、系统、引导，然后占用 RAM 和 CPU 以迫使攻击者重启系统。

#### 步骤

运行安装脚本（由 [hellresistor](https://gist.github.com/hellresistor/a4c542415a2d437e21afc235260d2366) 提供的懒人工具脚本）：

```bash
#!/bin/bash
# 克隆并编译 pam-duress
cd "$HOME" || exit 1
git clone https://github.com/nuvious/pam-duress.git
cd pam-duress || exit 1
make
sudo make install
# 配置应急脚本和 PAM
```

---

## 网络

### 使用 UFW（简易防火墙）

#### 为什么

作者的观点是默认拒绝所有传入和传出的流量，只允许明确需要的例外。确保只有我们明确允许的流量通过是防火墙的职责。

#### 工作原理

Linux 内核提供了监控和控制网络流量的能力。UFW 是 iptables 的前端，它简化了管理 iptables 规则的过程。

**UFW** 通过让你配置规则来工作，这些规则：

- **允许**或**拒绝**
- **输入**或**输出**流量
- **到**或**从**端口

#### 步骤

1. 安装 ufw：

   ```bash
   sudo apt install ufw
   ```

2. 拒绝所有传出流量（如果你和作者一样偏执）：

   ```bash
   sudo ufw default deny outgoing comment '拒绝所有传出流量'
   ```

   如果你不想拒绝所有传出流量，可以允许：

   ```bash
   sudo ufw default allow outgoing comment '允许所有传出流量'
   ```

3. 拒绝所有传入流量：

   ```bash
   sudo ufw default deny incoming comment '拒绝所有传入流量'
   ```

4. 允许 SSH 连接（使用 limit 可以在 IP 地址在 30 秒内尝试 6 次以上连接时自动拒绝）：

   ```bash
   sudo ufw limit in ssh comment '允许 SSH 连接进入'
   ```

5. 根据你的需要允许额外的流量：

   ```bash
   # 允许 DNS 传出
   sudo ufw allow out 53 comment '允许 DNS 传出'
   
   # 允许 NTP 传出
   sudo ufw allow out 123 comment '允许 NTP 传出'
   
   # 允许 HTTP、HTTPS、FTP 传出（apt 可能需要这些）
   sudo ufw allow out http comment '允许 HTTP 流量传出'
   sudo ufw allow out https comment '允许 HTTPS 流量传出'
   sudo ufw allow out ftp comment '允许 FTP 流量传出'
   
   # 允许 whois
   sudo ufw allow out whois comment '允许 whois'
   
   # 允许邮件传出用于状态通知
   sudo ufw allow out 25 comment '允许 SMTP 传出'
   sudo ufw allow out 587 comment '允许 SMTP 传出'
   
   # 如果你使用 DHCP
   sudo ufw allow out 67 comment '允许 DHCP 客户端更新'
   sudo ufw allow out 68 comment '允许 DHCP 客户端更新'
   ```

6. 启动 ufw：

   ```bash
   sudo ufw enable
   ```

7. 查看状态：

   ```bash
   sudo ufw status verbose
   ```

#### 自定义应用程序配置

如果你不想通过显式提供端口号来创建规则，可以在 `/etc/ufw/applications.d` 中创建自己的应用程序配置文件：

```bash
cat /etc/ufw/applications.d/plexmediaserver
```

```
[PlexMediaServer]
title=Plex Media Server
description=This opens up PlexMediaServer for http (32400), upnp, and autodiscovery.
ports=32469/tcp|32413/udp|1900/udp|32400/tcp|32412/udp|32410/udp|32414/udp|32400/udp
```

然后像其他应用一样启用它：

```bash
sudo ufw allow plexmediaserver
```

### 使用 PSAD 进行 iptables 入侵检测与防御

#### 为什么

即使你有防火墙守卫你的门，也有可能尝试在任何守卫的门上进行暴力破解。我们希望监控所有网络活动以检测潜在的入侵尝试，例如反复尝试进入，并阻止它们。

#### 工作原理

Fail2Ban 扫描各种应用程序（如 Apache、SSH 或 FTP）的日志文件并自动封禁显示恶意迹象的 IP。PSAD 则扫描 iptables 和 ip6tables 日志消息，以检测并可选地阻止扫描和其他类型的可疑流量，如 DDoS 或操作系统指纹识别尝试。可以同时使用这两个程序，因为它们在不同的层面上运行。

#### 步骤

1. 安装 psad：

   ```bash
   sudo apt install psad
   ```

2. 查看并更新 `/etc/psad/psad.conf` 中的配置选项。特别注意以下设置：

   | 设置                     | 设定值                      |
   | ------------------------ | --------------------------- |
   | `EMAIL_ADDRESSES`        | 你的电子邮件地址            |
   | `HOSTNAME`               | 你的服务器主机名            |
   | `ENABLE_PSADWATCHD`      | `ENABLE_PSADWATCHD Y;`      |
   | `ENABLE_AUTO_IDS`        | `ENABLE_AUTO_IDS Y;`        |
   | `ENABLE_AUTO_IDS_EMAILS` | `ENABLE_AUTO_IDS_EMAILS Y;` |

3. 编辑 `/etc/ufw/before.rules` 和 `/etc/ufw/before6.rules`，在 **COMMIT 行之前**添加：

   ```
   # 记录所有流量以便 psad 分析
   -A INPUT -j LOG --log-tcp-options --log-prefix "[IPTABLES] "
   -A FORWARD -j LOG --log-tcp-options --log-prefix "[IPTABLES] "
   ```

4. 重新加载 ufw 和 psad：

   ```bash
   sudo ufw reload
   sudo psad -R
   sudo psad --sig-update
   sudo psad -H
   ```

5. 检查 psad 状态：

   ```bash
   sudo psad --Status
   ```

### 使用 Fail2Ban 进行应用入侵检测与防御

#### 为什么

UFW 告诉你的服务器哪些门应该封住，哪些门允许授权用户通过。PSAD 监控网络活动以检测和阻止潜在的入侵。但是你的服务器运行的应用程序/服务（如 SSH 和 Apache）呢？即使访问被允许，也不意味着所有访问尝试都是有效和无害的。这就是 Fail2ban 的作用。

#### 工作原理

Fail2ban 监控你的应用程序日志（如 SSH 和 Apache），通过阻止可疑活动（例如短时间内连续多次失败连接）来检测和阻止潜在的入侵。

#### 步骤

1. 安装 fail2ban：

   ```bash
   sudo apt install fail2ban
   ```

2. 创建 `/etc/fail2ban/jail.local` 并添加：

   ```
   [DEFAULT]
   # 我们想要忽略的 IP 地址范围
   ignoreip = 127.0.0.1/8 [LAN网段]
   
   # 发送电子邮件的收件人
   destemail = [你的电子邮件]
   
   # 电子邮件发件人
   sender = [你的电子邮件]
   
   # 使用 exim4 发送邮件
   mta = mail
   
   # 获取电子邮件警报
   action = %(action_mwl)s
   ```

3. 为 SSH 创建 jail，创建 `/etc/fail2ban/jail.d/ssh.local`：

   ```
   [sshd]
   enabled = true
   banaction = ufw
   port = ssh
   filter = sshd
   logpath = %(sshd_log)s
   maxretry = 5
   ```

4. 启用 fail2ban：

   ```bash
   sudo fail2ban-client start
   sudo fail2ban-client reload
   ```

5. 检查状态：

   ```bash
   sudo fail2ban-client status
   sudo fail2ban-client status sshd
   ```

#### 解除封禁 IP

```bash
fail2ban-client set sshd unbanip 192.168.1.100
```

### 使用 CrowdSec 进行应用入侵检测与防御

#### 为什么

CrowdSec 与 Fail2Ban 类似，监控你的应用程序日志并阻止入侵，但它还与一个社区耦合，该社区将威胁情报分享回 CrowdSec，然后将社区阻止列表分发给所有用户。

#### 步骤

1. 安装 CrowdSec 安全引擎 (IDS)：

   ```bash
   curl -s https://install.crowdsec.net | sudo sh
   sudo apt install crowdsec
   ```

2. 安装补救组件 (IPS)：

   ```bash
   sudo apt install crowdsec-firewall-bouncer-iptables
   ```

3. 检查检测和补救是否正常工作：

   ```bash
   sudo cscli metrics
   ```

#### 解除封禁 IP

```bash
cscli decisions delete --ip 192.168.1.100
```

---

## 审计

### 使用 AIDE 进行文件/文件夹完整性监控 (WIP)

AIDE（高级入侵检测环境）是一个文件和目录完整性检查器。它创建文件/文件夹的数据库，记录各种属性，然后可以在之后检查是否有变化。通常配置为每天运行并通过电子邮件通知你。

#### 步骤

1. 安装 AIDE：

   ```bash
   sudo apt install aide aide-common
   ```

2. 创建新数据库：

   ```bash
   sudo aideinit
   ```

3. 测试是否工作：

   ```bash
   sudo aide.wrapper --check
   ```

4. 每次你对 AIDE 监控的文件/文件夹进行更改后，更新数据库：

   ```bash
   sudo aideinit -y -f
   ```

### 使用 ClamAV 进行反病毒扫描 (WIP)

ClamAV 是一个病毒扫描器。clamav-freshclam 是保持病毒定义更新的服务。

#### 步骤

1. 安装：

   ```bash
   sudo apt install clamav clamav-freshclam clamav-daemon
   ```

2. 启动 freshclam 服务：

   ```bash
   sudo service clamav-freshclam start
   ```

3. 扫描文件：`clamscan /path/to/file`

4. 扫描目录：`clamscan -r /path/to/folder`

5. 使用 `-i` 开关仅打印受感染的文件

### 使用 Rkhunter 进行 Rootkit 检测 (WIP)

Rkhunter（Rootkit Hunter）是一个扫描 rootkit、后门和本地漏洞的 POSIX 兼容工具。

#### 步骤

1. 安装：

   ```bash
   sudo apt install rkhunter
   ```

2. 复制配置文件：

   ```bash
   sudo cp -p /etc/rkhunter.conf /etc/rkhunter.conf.local
   ```

3. 更新 rkhunter 及其数据库：

   ```bash
   sudo rkhunter --versioncheck
   sudo rkhunter --update
   sudo rkhunter --propupd
   ```

4. 手动扫描：

   ```bash
   sudo rkhunter --check
   ```

5. 配置每天运行：

   ```bash
   sudo dpkg-reconfigure rkhunter
   ```

### 使用 chrootkit 进行 Rootkit 检测 (WIP)

chkrootkit 是一个在本地检查 rootkit 迹象的工具。

#### 步骤

1. 安装：

   ```bash
   sudo apt install chkrootkit
   ```

2. 手动扫描：

   ```bash
   sudo chkrootkit
   ```

3. 启用每天运行：

   ```bash
   sudo dpkg-reconfigure chkrootkit
   ```

### logwatch - 系统日志分析和报告工具

#### 为什么

你的服务器将生成大量可能包含重要信息的日志。除非你计划每天检查服务器，否则你需要一种方法来获取服务器日志的电子邮件摘要。我们将使用 logwatch 来完成此任务。

#### 步骤

1. 安装：

   ```bash
   sudo apt install logwatch
   ```

2. 查看 logwatch 收集的样本：

   ```bash
   sudo /usr/sbin/logwatch --output stdout --format text --range yesterday --service all
   ```

3. 修改每日 cron 作业 `/etc/cron.daily/00logwatch`，将执行行改为：

   ```
   /usr/sbin/logwatch --output mail --format html --mailto root --range yesterday --service all
   ```

4. 测试 cron 作业：

   ```bash
   sudo /etc/cron.daily/00logwatch
   ```

### ss - 查看服务器正在监听的端口

#### 为什么

我们显然不希望服务器在我们不知道的端口上监听。我们将使用 `ss` 查看所有服务正在监听的端口。

#### 步骤

查看所有正在监听流量的端口：

```bash
sudo ss -lntup
```

**开关说明**：

- `l` = 显示监听 socket
- `n` = 不尝试解析服务名称
- `t` = 显示 TCP socket
- `u` = 显示 UDP socket
- `p` = 显示进程信息

如果你看到任何可疑的内容，比如你不知道的端口或不认识的进程，请调查并进行必要的修复。

### Lynis - Linux 安全审计工具

来自 https://cisofy.com/lynis/：

> Lynis 是一个经过实战检验的安全工具，适用于运行 Linux、macOS 或基于 Unix 的操作系统。它对系统执行广泛的健康扫描，以支持系统加固和合规性测试。

#### 步骤

1. 在基于 Debian 的系统上安装（使用 CISOFY 的社区软件仓库）：

   ```bash
   sudo apt install ca-certificates host
   sudo mkdir -p /etc/apt/keyrings
   wget -O - https://packages.cisofy.com/keys/cisofy-software-public.key | sudo gpg --dearmor -o /etc/apt/keyrings/cisofy-lynis.gpg
   echo "deb [signed-by=/etc/apt/keyrings/cisofy-lynis.gpg] https://packages.cisofy.com/community/lynis/deb/ stable main" | sudo tee /etc/apt/sources.list.d/cisofy-lynis.list
   sudo apt update
   sudo apt install lynis
   ```

2. 更新：

   ```bash
   sudo lynis update info
   ```

3. 运行安全审计：

   ```bash
   sudo lynis audit system
   ```

   这将扫描你的服务器，报告审计发现，并在最后给出建议。花些时间查看输出并处理发现的问题。

### OSSEC - 主机入侵检测系统

来自 https://github.com/ossec/ossec-hids：

> OSSEC 是一个完整的监控和控制你的系统的平台。它将 HIDS（基于主机的入侵检测）、日志监控和 SIM/SIEM 的所有方面混合在一个简单、强大和开源的解决方案中。

#### 步骤

1. 从源码安装 OSSEC-HIDS：

   ```bash
   sudo apt install -y libz-dev libssl-dev libpcre2-dev build-essential libsystemd-dev
   wget https://github.com/ossec/ossec-hids/archive/3.7.0.tar.gz
   tar xzf 3.7.0.tar.gz
   cd ossec-hids-3.7.0/
   sudo ./install.sh
   ```

2. 查看所有警报：

   ```bash
   tail -f /var/ossec/logs/alerts/alerts.log
   ```

---

## 危险区域

**!! 继续操作需自行承担风险 !!**

本部分涵盖高风险的内容，因为它们可能使你的系统无法使用，或被许多人认为是不必要的，因为风险大于任何回报。

### Linux 内核 sysctl 加固

#### 为什么

内核是 Linux 系统的大脑。保护它是有道理的。

#### 为什么不

使用 sysctl 更改内核设置是有风险的，可能会损坏你的服务器。如果你不知道自己在做什么，没有时间调试问题，或者只是不想承担风险，建议不要执行这些步骤。

#### 免责声明

作者对加固/保护 Linux 内核的知识不如期望的那样深入。作者并不确切知道所有这些设置的作用。由于作者不能 100% 确定每个设置的具体功能，因此从众多站点获取了推荐设置并进行了整合。

#### 步骤

1. 内核 sysctl 设置可以在 [linux-kernel-sysctl-hardening.md](linux-kernel-sysctl-hardening.md) 文件中找到。该文件包含了一个综合列表，涵盖了来自多个来源的所有 sysctl 加固建议，包括文件系统、内核、网络 (IPv4/IPv6) 和虚拟内存等关键领域的参数配置。

2. 在将内核 sysctl 更改设为永久之前，可以先使用 sysctl 命令进行测试：

   ```bash
   sudo sysctl -w kernel.ctrl-alt-del=0
   ```

3. 测试后通过将值添加到 `/etc/sysctl.conf` 使其永久生效：

   ```
   kernel.ctrl-alt-del = 0
   fs.file-max = 65535
   kernel.kptr_restrict = 2
   kernel.randomize_va_space = 2
   kernel.sysrq = 0
   net.ipv4.conf.all.accept_redirects = 0
   net.ipv4.conf.all.rp_filter = 1
   net.ipv4.conf.all.send_redirects = 0
   net.ipv4.tcp_syncookies = 1
   net.ipv6.conf.all.accept_redirects = 0
   ...
   ```

4. 重新加载设置：

   ```bash
   sudo sysctl -p
   ```

### 使用密码保护 GRUB

#### 为什么

如果恶意行为者可以物理访问你的服务器，他们可以使用 GRUB 获取对系统的未授权访问。

#### 步骤

1. 创建密码的 PBKDF2 哈希：

   ```bash
   grub-mkpasswd-pbkdf2 -c 100000
   ```

2. 创建 `/etc/grub.d/01_password` 并添加：

   ```bash
   #!/bin/sh
   set -e
   
   cat << EOF
   set superusers="grub"
   password_pbkdf2 grub [哈希值]
   EOF
   ```

3. 设置执行位：

   ```bash
   sudo chmod a+x /etc/grub.d/01_password
   ```

4. 更新 GRUB：

   ```bash
   sudo update-grub
   ```

### 禁用 root 登录

#### 为什么

如果你已经[正确配置了 sudo](#限制可以使用-sudo-的用户)，那么 **root** 账户基本上不需要直接登录——无论是在终端还是远程。

**警告：这可能会导致某些配置出现问题！**

#### 步骤

锁定 **root** 账户：

```bash
sudo passwd -l root
```

### 更改默认 umask

#### 为什么

umask 控制文件/文件夹创建时的**默认**权限。不安全的文件/文件夹权限会给其他账户潜在的未授权访问。

- 对于**非 root** 账户，其他账户默认不需要对账户的文件/文件夹进行任何访问。
- 对于 **root** 账户，文件/文件夹的主要组或其他账户默认不需要对 **root** 文件/文件夹进行任何访问。

#### 步骤

1. 备份将要编辑的文件：

   ```bash
   sudo cp --archive /etc/profile /etc/profile-COPY-$(date +"%Y%m%d%H%M%S")
   sudo cp --archive /etc/bash.bashrc /etc/bash.bashrc-COPY-$(date +"%Y%m%d%H%M%S")
   sudo cp --archive /etc/login.defs /etc/login.defs-COPY-$(date +"%Y%m%d%H%M%S")
   sudo cp --archive /root/.bashrc /root/.bashrc-COPY-$(date +"%Y%m%d%H%M%S")
   ```

2. 在 `/etc/profile` 和 `/etc/bash.bashrc` 中设置非 root 账户默认 umask 为 **0027**：

   ```
   umask 0027
   ```

3. 在 `/etc/login.defs` 中添加：

   ```
   UMASK 0027
   ```

4. 在 `/root/.bashrc` 中设置 root 账户默认 umask 为 **0077**：

   ```
   umask 0077
   ```

### 孤立软件包清理

#### 为什么

随着系统的使用和软件包的安装/卸载，你最终会得到孤立的或未使用的软件/包/库。当安全是优先事项时，任何不是明确需要的东西都是潜在的安全威胁。

#### 基于 Debian 的系统

使用 [deborphan](http://freshmeat.sourceforge.net/projects/deborphan/) 查找孤立软件包：

```bash
sudo apt install deborphan
sudo deborphan
```

删除所有孤立软件包：

```bash
sudo apt --autoremove purge $(deborphan)
```

---

## 其他杂项

### 使用 MSMTP 发送邮件（简易方式）

一个使用 Gmail 账户通过 MSMTP 发送电子邮件的简化方法：

```bash
#!/bin/bash
USEREMAIL="usernameemail"
DOMPROV="gmail.com"
PWDEMAIL="passwordStrong"
MAILPROV="smtp.google.com:583"

apt install -y msmtp
ln -s /usr/bin/msmtp /usr/sbin/sendmail

# 创建配置并使用 GPG 加密密码
# 详细配置包括 TLS、认证和日志记录
```

### 使用 Gmail 和 Exim4 作为 MTA 并启用隐式 TLS

#### 为什么

除非你计划设置自己的邮件服务器，否则你需要一种方法来从你的服务器发送电子邮件。这对于系统警报/消息非常重要。

**重要**：Google 不再允许使用你的账户密码进行认证。你需要启用双因素认证然后使用应用专用密码。

#### 步骤

1. 安装 exim4 和依赖：

   ```bash
   sudo apt install exim4 openssl ca-certificates
   ```

2. 配置 exim4：

   ```bash
   sudo dpkg-reconfigure exim4-config
   ```

   配置提示回答：

   | 提示                              | 回答                                    |
   | --------------------------------- | --------------------------------------- |
   | 邮件配置的通用类型                | `mail sent by smarthost; no local mail` |
   | 系统邮件名称                      | `localhost`                             |
   | 监听传入 SMTP 连接的 IP 地址      | `127.0.0.1; ::1`                        |
   | 传出 smarthost 的 IP 地址或主机名 | `smtp.gmail.com::465`                   |
   | 保持最小 DNS 查询？               | `No`                                    |
   | 拆分配置为小文件？                | `No`                                    |

3. 在 `/etc/exim4/passwd.client` 中添加：

   ```
   smtp.gmail.com:yourAccount@gmail.com:yourAppPassword
   *.google.com:yourAccount@gmail.com:yourAppPassword
   ```

4. 保护密码文件：

   ```bash
   sudo chown root:Debian-exim /etc/exim4/passwd.client
   sudo chmod 640 /etc/exim4/passwd.client
   ```

5. 生成 TLS 证书：

   ```bash
   sudo bash /usr/share/doc/exim4-base/examples/exim-gencert
   ```

6. 创建 `/etc/exim4/exim4.conf.localmacros`：

   ```
   MAIN_TLS_ENABLE = 1
   REMOTE_SMTP_SMARTHOST_HOSTS_REQUIRE_TLS = *
   TLS_ON_CONNECT_PORTS = 465
   REQUIRE_PROTOCOL = smtps
   IGNORE_SMTP_LINE_LENGTH_LIMIT = true
   ```

7. 更新并重启 exim4：

   ```bash
   sudo update-exim4.conf
   sudo service exim4 restart
   ```

8. 如果使用 UFW，允许 465 端口传出：

   ```bash
   sudo ufw allow out 465 comment '开放 TLS 端口 465 用于 SMTP 发送邮件'
   ```

9. 在 `/etc/aliases` 中添加邮件别名：

   ```
   user1: user1@gmail.com
   ```

10. 测试设置：

    ```bash
    echo "test" | mail -s "Test" email@gmail.com
    ```

### 分离 iptables 日志文件

#### 步骤

1. 确保防火墙已配置为在所有日志条目前添加前缀。如果使用 psad，[步骤 4](#psad_step4) 已完成此操作。

2. 创建 `/etc/rsyslog.d/10-iptables.conf`：

   ```
   :msg, contains, "[IPTABLES] " /var/log/iptables.log
   & stop
   ```

3. 在 `/etc/psad/psad.conf` 中更新 psad 的新日志文件路径：

   ```
   IPT_SYSLOG_FILE /var/log/iptables.log;
   ```

4. 创建 `/etc/logrotate.d/iptables` 用于日志轮转：

   ```
   /var/log/iptables.log
   {
       rotate 7
       daily
       missingok
       notifempty
       delaycompress
       compress
       postrotate
           invoke-rc.d rsyslog rotate > /dev/null
       endscript
   }
   ```

---

## 补充文件

### Nginx 安全配置

本项目还包含一个 [nginx.md](nginx.md) 文件，提供了 Nginx Web 服务器的安全头部配置建议：

- **禁用服务器令牌** (`server_tokens off;`) - 隐藏 Nginx 版本信息
- **内容安全策略** (Content-Security-Policy) - 限制资源加载来源
- **X-Frame-Options** - 防止点击劫持攻击
- **X-XSS-Protection** - 启用浏览器 XSS 过滤
- **Referrer-Policy** - 控制引用信息发送
- **Permissions-Policy** - 限制浏览器功能 API 使用
- **X-Content-Type-Options** - 防止 MIME 类型嗅探

### Linux 内核 sysctl 加固参考

[linux-kernel-sysctl-hardening.md](linux-kernel-sysctl-hardening.md) 文件包含了一个综合列表，整合了来自多个来源的所有 sysctl 加固建议，涵盖：

- 文件系统设置 (`fs.*`)
- 内核设置 (`kernel.*`)
- 网络核心设置 (`net.core.*`)
- IPv4 设置 (`net.ipv4.*`)
- IPv6 设置 (`net.ipv6.*`)
- Unix 域套接字设置 (`net.unix.*`)
- 虚拟内存设置 (`vm.*`)

---

## 补充内容

### 联系作者

如有任何问题、评论、顾虑、反馈或问题，请提交 [new issue](https://github.com/imthenachoman/How-To-Secure-A-Linux-Server/issues/new)。

### 有用的链接

- [https://github.com/pratiktri/server_init_harden](https://github.com/pratiktri/server_init_harden) - 自动化 Linux 服务器初始安全加固的 Bash 脚本
- [How To Secure A Linux Server With Ansible](https://github.com/moltenbit/How-To-Secure-A-Linux-Server-With-Ansible) - 本指南的 Ansible playbook 版本

### 致谢

感谢在 Reddit、Hacker News 等社区提供反馈和建议的所有人，以及为本项目做出贡献的各位开发者。

特别感谢：

- [moltenbit](https://github.com/moltenbit) 提供的 Ansible playbook 版本
- [remyabel](https://github.com/remyabel) 帮助解决了 Exim4 TLS 配置问题
- [Sonnenbrand](https://github.com/Sonnenbrand) 提供了关于保持第二个 SSH 会话的建议
- 所有提交 Issue 和 Pull Request 的贡献者

### 许可与版权

[![CC-BY-SA](https://i.creativecommons.org/l/by-sa/4.0/88x31.png)](http://creativecommons.org/licenses/by-sa/4.0/)

[How To Secure A Linux Server](https://github.com/imthenachoman/How-To-Secure-A-Linux-Server) 由 [Anchal Nigam](https://github.com/imthenachoman) 创作，依据 [Creative Commons Attribution-ShareAlike 4.0 International License](http://creativecommons.org/licenses/by-sa/4.0/) 许可。

详见 [LICENSE](LICENSE.txt) 获取完整许可文本。
