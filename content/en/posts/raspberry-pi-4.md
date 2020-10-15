+++
title = "raspberry pi 4"
author = ["emacle"]
date = 2019-09-16T09:34:00+08:00
tags = ["pi"]
draft = false
authorEmoji = "🎅"
image = "https://www.raspberrypi.org/app/uploads/2011/10/Raspi-PGB001.png"
pinned = false
+++

## <span class="org-todo done DONE">DONE</span> 配置 dhcpcd-run-hook dhcpcd vs /etc/network/interfaces {#配置-dhcpcd-run-hook-dhcpcd-vs-etc-network-interfaces}

pi 使用 dhcpcd 作为dhcp client 配置 静态IP也在这里进行配置
必须dhcp获取到ip地址后, 执行 frpc 命令, 否则会出现 frpc不能连接自动断开退出的情况

 <https://blog.csdn.net/yjbaobo/article/details/75146840>
这主要是由于版本差异 Debian 'Jessie' in place of Debian 'Wheezy'.
我的是jessie选择修改的是 dhcpcd.conf这个。其中作者也说了如何设置usb无线的配置文件。

```bash
#!/bin/sh
# /etc/dhcpcd.exit-hook - runs as last dhcpcd hook
# man dhcpcd-run-hooks 查看参数 $if_up $interface $reason etc
if [ "$interface" = eth0 ] && [ "$if_up" = true ] ; then
    # 测试 dhcpcd 变量变化, grep reason /tmp/variables.txt,  if_up interface等 测试完成需要关闭防止文件变巨大
    # echo "================" >> /tmp/variables.txt
    # echo "Environment Variables" >> /tmp/variables.txt
    # printenv >> /tmp/variables.txt
    # echo "===============" >> /tmp/variables.txt
    # echo "Shell Variables" >> /tmp/variables.txt
    # set >> /tmp/variables.txt

    # dhcpcd 每一次dhcp信息发生变化时 都会调用该脚本 $reason 每次不太相同 通过上面变量观察使用dhcp时 BOUND 只在第一次分配ip时出现
    # $reason为 BOUND 时 执行一次 frp 命令 防止重复添加/也可以判断 frp 进程存在与否来做判断
    case "$reason" in
      BOUND)
  	  # logger信息 写入syslog
  	  logger 'Running frp/DYNDNS update...'
  	  # curl --stderr - -v "https://dynv6.com/api/update(blah)"
  	  nohup /opt/frp_0.29.0_linux_arm/frpc -c /opt/frp_0.29.0_linux_arm/frpc.ini >/dev/null 2>&1 &
  	  ;;
    esac
fi
```

ip route add 10.11.12.0/24 via 192.168.192.5

```text
# man dhcpcd-run-hooks

BOUND | BOUND6    dhcpcd obtained a new lease from a DHCP server.
RENEW | RENEW6    dhcpcd renewed it's lease.
REBIND | REBIND6  dhcpcd has rebound to a new DHCP server.
ROUTERADVERT      dhcpcd has received an IPv6 Router Advertisement, or
  		  one has expired.

$interface                   _the name of the interface._
$protocol                    the protocol that triggered the event.
$reason                      _as described above._
$pid                         the pid of dhcpcd.
$if_up                       true if the interface is up, otherwise
  			     false.
```

/etc/dhcp/dhclient-enter-hooks.d 是 isc-dhcp-client 包的内容 也是一个dhcp 客户端, 可做为 dhcpcd 的备选
isc-dhcp-client 与 dhcpcd一样 都可作为 dhcp client busty 默认使用 dhcpcd


## <span class="org-todo done DONE">DONE</span> 开启SSH {#开启ssh}

sd卡 /boot 目录 默认不开启SSH，需要在SD卡根目录（boot中）新建 "SSH" 文件（无后缀）
ssh 进入系统后 /boot/SSH 文件应该被系统删除,但是ssh 服务是永远开启了


## <span class="org-todo done DONE">DONE</span> mdadm raid {#mdadm-raid}

```sh
apt install mdadm
# 装配原有的磁盘阵列 组合成新的磁盘阵列 md0 此操作在里而且会自动更新 /etc/mdadm/mdadm.conf 文件
# 同时也更新了内核映象 (update-initramfs -u) 启动时内核会自动装配md0 ，如果 sd[a-c]存在的话
mdadm -A /dev/md0  /dev/sd[a-c]1  或 mdadm -A /dev/md0 /dev/sda1 /dev/sdb1 /dev/sdc1

# definitions of existing MD arrays cat /etc/mdadm/mdadm.conf
ARRAY /dev/md/0  metadata=1.2 UUID=fabd24e6:e0160fe0:a0af4d3c:4498ec65 name=debian:0
```


## <span class="org-todo done DONE">DONE</span> fstab nofail {#fstab-nofail}

外部设备(/dev/md0)在插入时挂载，在未插入时忽略。
这需要 nofail 选项，可以在启动时若设备不存在直接忽略它而不报错进入recovery mode Contrl + D
<span class="underline">Mount an external drive at boot time only if it is plugged in</span>

cat /etc/fstab

```text
/dev/md0 /mnt/raid5 ext4 defaults,nofail 0 0
```


## <span class="org-todo done DONE">DONE</span> hdmi {#hdmi}

hdmi: 对于chuangwei\_TV , pi 的/boot/config里默认参数即可，不需要做额外配置


## <span class="org-todo done DONE">DONE</span> pi未连接显示器时如果使用vnc连接出现Cannot currently show the dekstop {#pi未连接显示器时如果使用vnc连接出现cannot-currently-show-the-dekstop}

`Do not use default resolution`

```sh
The RPi must be set to boot to desktop (service mode).
If a HDMI monitor is not attached then you need to specify a screen resolution
sudo raspi-config
Advanced Options > Resolution > DMT Mode 85 1280x720

raspi-config 命令 相当于： 修改 /boot/config.txt 内容
hdmi_force_hotplug=1
hdmi_group=2
hdmi_mode=85
```
