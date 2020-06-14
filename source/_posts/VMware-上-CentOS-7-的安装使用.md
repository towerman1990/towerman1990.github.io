---
title: VMware 上 CentOS 7 的安装使用
date: 2020-02-03 11:30:00
tags:
  - VMware
  - Linux
  - CentOS
---

VMware Workstation是一款功能强大的桌面虚拟计算机软件，现在也已经提供面向个人用户的免费版本。虽然个人版的功能受限，不如商业版功能强大，但是作为个人开发使用还是够用的。这里就基于VMware Workstation，演示一下CentOS 7的安装步骤。

<!-- more -->

- 环境准备：

1. Windows10操作系统

  现在的win10无论是兼容性，还是健壮性都已经得到了验证。作为一个开发者，建议使用win10作为基础的操作系统。

2. VWvare虚拟机

  建议使用最新的15版本，之前的12版本在win10上会因为兼容性问题，阻止win10的功能性更新。而且自己开发使用，可以选择免费的基础版的Workstation 15 Player：[下载 VMware Workstation Player | VMware](https://www.vmware.com/cn/products/workstation-player/workstation-player-evaluation.html)

3. CentOS 7安装包

  CentOS 7 各版本的aliyun下载地址：[Index of /centos/7.7.1908/isos/x86_64/](http://mirrors.aliyun.com/centos/7.7.1908/isos/x86_64/)
  服务器开发选择最精简的minimal版即可，如果喜欢使用Linux桌面开发环境的可以选择普通的DVD版，本文使用minimal版进行安装演示。

- 安装

  1. 选择“创建新虚拟机”：

    {% asset_img 01.create_button.png create_button %}

  2. 点击“浏览”按钮，选择centos安装包：

    {% asset_img 02.select_iso.png select_iso %}

  3. 给新建虚拟机命名，并选择存放位置：

    {% asset_img 03.name.png name %}

  4. 分配硬盘空间，这里我们选择“将虚拟硬盘分拆成多个文件”：

    {% asset_img 04.disk_size.png disk_size %}

  5. 点击“完成”按钮：

    {% asset_img 05.complete.png complete %}

  6. 进入安装引导页面，首先将鼠标定位到虚拟机中，然后可以通过↑↓选择“Install CenOS 7"选项：

    {% asset_img 06.install.png install %}

  7. 选择语言，然后点击”继续“按钮：

    {% asset_img 07.select_langue.png select_langue %}

  8. 进入”安装信息摘要“界面后，点击”安装位置“，选择之前配置的硬盘，自动配置分区即可。然后点击”完成“，返回安装界面：

    {% asset_img 08.divide_disk.png divide_disk %}

  9. 点击”开始安装“，此时可以选择设置root密码和创建新用户，操作类似上一步选择的安装位置：

    {% asset_img 09.installing.png installing %}

  10. 完成后点击”重启“即可进入到登录界面，使用之前配置root用户或新建用户进行登录：

    {% asset_img 10.login.png login %}

- 配置IP地址与网关

  1. 新安装的centos是不能联网的，需要配置IP地址与网关。首先通过命令查看网卡配置文件：

    `/etc/sysconfig/network-scripts/ | grep ifcfg-en`

    {% asset_img 11.network.png network %}

  2. 可以看到网卡的文件名为”ifcfg-ens33“，然后使用vi编辑器打开网卡文件：

    `vi /etc/sysconfig/network-scripts/ifcfg-ens-33`

    {% asset_img 12.open_network.png open_network %}

  3. 普通版的workstation缺少虚拟网络管理功能，我们可以参照本机的网络设置，对虚拟机的网络进行配置。进入win10的设置→网络和Internet→查看网络属性，找到结尾为VMnet8的网络属性（此网络为NAT模式时使用）：

    {% asset_img 13.win_network.png win_network %}

  4. 修改虚拟机的网卡配置：

    将BOOTPROTO改为static（静态ip）

    将ONBOOT改为yes（开机启动）

    增加ip地址、网络掩码、网关、DNS等参数（虚拟机与主机ip要同网段）

    {% asset_img 14.net_config.png net_config %}

  5. wq保存后，重启网络服务，然后即可ping通百度了：

    `systemctl restart network`

    {% asset_img 15.success.png success %}