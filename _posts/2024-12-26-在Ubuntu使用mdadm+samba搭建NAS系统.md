---
layout: post
title: "在Ubuntu使用mdadm+samba搭建NAS系统"
---

介绍在VM虚拟机上，在Ubuntu 22.04上，使用mdadm创建RAID 5，并结合samba，实现NAS系统的方法，并介绍RAID 5的扩容方法。本文不介绍samba中用户权限管理。

### 预备知识

- VMware Workstation的基本使用方法
- Ubuntu 22.04的基本使用方法

### 环境

- VMware Workstation 17.6.2 build-24409262
- 虚拟机Ubuntu 22.04运行在单独的系统盘上
- 宿主机Windows 10

### 步骤

#### 虚拟硬件环境

1. 在VMware Workstation中，添加4块硬盘。接口是SCSCI，大小是0.1G（约100MB）。虚拟机共有5个硬盘。

2. 启动虚拟机。

3. 检查新硬盘是否被系统识别：
    ```
    lsblk
    ```
    
    示例输出：
    ```
    （省略若干行）
    sda      8:0    0    20G  0 disk 
    ├─sda1   8:1    0     1M  0 part 
    ├─sda2   8:2    0   513M  0 part /boot/efi
    └─sda3   8:3    0  19.5G  0 part /
    sdb      8:16   0   102M  0 disk 
    sdc      8:32   0   102M  0 disk 
    sdd      8:48   0   102M  0 disk 
    sde      8:64   0   102M  0 disk 
    ```
    示例输出解释：`sda`是系统盘，`sdb, sdc, sdd, sde`是4个后续用于创建RAID的硬盘。

#### 使用mdadm创建RAID 5

1. 创建RAID设备
    ```
    sudo mdadm --create --verbose /dev/md0 --level=5 --raid-devices=3 /dev/sdb /dev/sdc /dev/sdd
    ```
    命令解释：
    - `--verbose /dev/md0`指定RAID设备文件
    - `--level=5`指定RAID 5
    - `--raid-devices=3 /dev/sdb /dev/sdc /dev/sdd`指定3个硬盘

2. 查看RAID状态
    ```
    cat /proc/mdstat
    ```

    示例输出：
    ```
    Personalities : [raid6] [raid5] [raid4] [linear] [multipath] [raid0] [raid1] [raid10] 
    md0 : active raid5 sdd[3] sdc[1] sdb[0]
       204800 blocks super 1.2 level 5, 512k chunk, algorithm 2 [3/3] [UUU]
          
    unused devices: <none>
    ```

3. 格式化RAID设备为ext4文件系统
    ```
    sudo mkfs.ext4 /dev/md0
    ```

4. 查看RAID设备状态
    ```
    sudo mdadm --detail /dev/md0
    ```

    示例输出和解释：
    ```
    /dev/md0:
               Version : 1.2
         Creation Time : Thu Dec 26 23:40:13 2024
            Raid Level : raid5
            Array Size : 206848 (202.00 MiB 211.81 MB)  # RAID设备的可用容量
         Used Dev Size : 103424 (101.00 MiB 105.91 MB)  # RAID设备的组成设备的容量
          Raid Devices : 3
         Total Devices : 3
           Persistence : Superblock is persistent
    
           Update Time : Thu Dec 26 23:43:37 2024
                 State : clean 
        Active Devices : 3
       Working Devices : 3
        Failed Devices : 0
         Spare Devices : 0
    
                Layout : left-symmetric
            Chunk Size : 512K
    
    Consistency Policy : resync
    
                  Name : XXXXX
                  UUID : XXXXXXXXXXXXXXXXXX
                Events : 35
    
        Number   Major   Minor   RaidDevice State
           0       8       16        0      active sync   /dev/sdb
           1       8       32        1      active sync   /dev/sdc
           3       8       48        2      active sync   /dev/sdd
    ```

5. 保存RAID配置
    ```
    mdadm -Ds > /etc/mdadm/mdadm.conf
    update-initramfs -u
    ```

6. 自动挂载RAID设备
    创建挂载点：
    ```
    sudo mkdir /mnt/raid
    ```
    
    挂载RAID：
    ```
    sudo mount /dev/md0 /mnt/raid
    ```
    
    设置开机自动挂载，向`/etc/fstab`末尾添加以下内容：
    ```
    /dev/md0 /mnt/raid ext4 defaults 0 0
    ```

7. 验证RAID，检查RAID可用空间
    ```
    df -h
    ```
    
    示例输出与解释：
    ```
    文件系统        容量  已用  可用 已用% 挂载点
    /dev/md0        186M  120K  177M    1% /mnt/raid
    （省略若干行的输出。使用3个100M硬盘的RAID 5理论上可组成200M空间，实际有186M）
    ```

#### 使用Samba设置共享目录

1. 设置共享目录的权限
    ```
    sudo chmod -R 775 /mnt/raid
    sudo chown -R nobody:nogroup /mnt/raid
    ```

2. 配置Samba。向`/etc/samba/smb.conf`末尾添加以下内容：
    ```
    [RAIDShare]
    comment = RAID Shared Folder
    path = /mnt/raid
    browsable = yes
    writable = yes
    read only = no
    guest ok = yes
    ```
    说明：
    - `RAIDShare`是共享目录名称。在Windows上等同于"本地磁盘"的命名和辨识作用。
    - 公开的读写权限。本文不介绍samba中用户权限管理。

3. 重启Samba服务
    ```
    sudo systemctl restart smbd
    ```

4. 验证共享目录

    在宿主机上访问共享目录。选择此电脑-添加一个网络位置-选择自定义网络位置。地址填入Ubuntu虚拟机的IP和共享目录名称，例如：
    ```
    \\192.168.2.150\RAIDShare
    ```
    可见目录下仅有名为`lost+found`的目录，忽略它。创建文件`Hi.txt`，内容为`Hi 123`。在Ubuntu虚拟机中，进入`/mnt/raid`目录，可见相同的`Hi.txt`。反之亦然。

#### 使用mdadm扩充RAID

至此，已经使用3个硬盘`sdb, sdc, sdd`创建RAID 5，将RAID设备放在`/dev/md0`。同时，验证了NAS可用性。接下来添加新的硬盘`sde`来扩充。

1. 向RAID设备添加硬盘
    ```
    sudo mdadm --grow /dev/md0 --raid-devices=4 --add /dev/sde
    ```
 
    示例输出：
    ```
    mdadm: added /dev/sde
    ```

2. 扩建文件系统（ext4）
    ```
    sudo resize2fs /dev/md0
    ```
    
    示例输出：
    ```
    resize2fs 1.46.5 (30-Dec-2021)
    /dev/md0 上的文件系统已被挂载于 /mnt/raid；需要进行在线调整大小
    old_desc_blocks = 1, new_desc_blocks = 1
    /dev/md0 上的文件系统大小已经调整为 77568 个块（每块 4k）。
    ```

3. 验证RAID

    检查RAID可用空间
    ```
    df -h
    ```
    
    示例输出和解释：
    ```
    文件系统        容量  已用  可用 已用% 挂载点
    /dev/md0        281M  124K  270M    1% /mnt/raid
    （省略若干行的输出。即使理论上可用空间有300M，实际容量从186M增长为281M，）
    ```
    在宿主机上进入共享目录，发现之前的`Hi.txt`文件存在，文件内容符合预期，说明成功扩容。

### 参考文献

[1] Standard RAID levels [Z/OL]. https://en.wikipedia.org/wiki/Standard_RAID_levels. 

[2] MDADM - Adding a new hard disk to an existing RAID1 [Z/OL]. https://unix.stackexchange.com/questions/665389/mdadm-adding-a-new-hard-disk-to-an-existing-raid1. 

[3] mdadm.conf file needed? How to create and where? [Z/OL]. https://askubuntu.com/questions/1105829/mdadm-conf-file-needed-how-to-create-and-where.
