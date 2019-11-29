---
layout: post
title:  "win下用Python获取计算机硬盘和分区信息"
author: lpj
categories: [ Python ]
image: assets/images/1.jpg
tags: [featured]
---

windows下需要用Python读取硬盘（U盘）扇区数据，因为需要GUI显示，所以也需要显示对应的硬盘和分区信息，方法记录如下：

## 1、读取当前系统中的硬盘信息

要用到wmi库，安装就不说了，用pip或者直接下载安装程序都可以，wmi库获取硬盘的函数是Win32_DiskDrive(),示例代码如下：  
```python
# 获取硬盘信息
def get_disks():
    dl = []
    disks = wmi.WMI().Win32_DiskDrive()
    for i in disks:
        tmp = {}
        tmp['inst'] = i
        tmp['name'] = i.name
        tmp['caption'] = i.caption
        dl.append(tmp)

    return dl
```
用下面的代码简单测试： 
```python
disks = get_disks()
print disks
```

输出如下：
```bash
[{'caption': u'SAMSUNG MZNTE256HMHP-00000', 'inst': <_wmi_object: \\85P68OL0AUZVMZB\root\cimv2:Win32_DiskDrive.DeviceID="\\\\.\\PHYSICALDRIVE0">, 'name': u'\\\\.\\PHYSICALDRIVE0'}]
```

## 2、读取当前系统中的所有分区信息

有两种办法可以获取分区信息，第一种是还用之前的wmi库，获取到disk信息后，再分别遍历每一个disk，得到所有的分区；第二种是使用win32file库获取分区，两种办法的示例代码分别如下：

```python
# 用wmi模块获取分区信息
def get_partitions():
    parts = []
    for disk in wmi.WMI().Win32_DiskDrive():
        for part in disk.associators('Win32_DiskDriveToDiskPartition'):
            for logic_part in part.associators('Win32_LogicalDiskToPartition'):
                parts.append(logic_part.caption[:-1])

    return parts
```

```python
# 用win32file模块获取当前分区卷标列表
def get_parts():
    ret = []
    r = win32file.GetLogicalDrives()
    for d in range(26):
        if(r>>d & 1):
            ret.append(string.ascii_letters[d].upper())

    return ret
```

测试代码及结果如下：
```bash
parts = get_partitions()
print parts
pl = get_parts()
print pl
```

```bash
[u'C', u'D']
['C', 'D']
```


