---
layout: post
title:  "Python实现文件定期维护"
author: lpj
categories: [ Python, 技术 ]
image: assets/images/7.jpg
---

题目为，用程序实现如下功能：

1. 程序运行时需要指定两个参数arg1、arg2，arg1为生成文件间隔，arg2为文件超时间隔，单位为秒
2. 定时（时间由运行参数指定）生成文件存放在程序所在目录下的 data 文件夹里
3. 生成的文件名为“yyyyddmmHHMMSS.txt”，即当前时间的字符串形式如20190425120356.txt
4. 要维护data文件夹内的文件，对于旧文件（距当前时间大于超时间隔）予以删除

即data文件夹中每arg1秒增加一个文件，文件名即它创建的时间，若某个文件距当前时间超过 arg2 秒，则将其删除, 其中arg1、arg2在程序运行时指定，如：
```bash
doc_manage.py arg1 arg2
```

Python2代码如下：
```python
import threading
import datetime
import sys
import os
 
# 定时器响应函数
def manage_file(t_gap, t_over):
    global timer
    # 先创建文件
    fpath = 'data/'
    now = datetime.datetime.now()
    fname = now.strftime('%Y%m%d%H%M%S') + '.txt'
    with open(fpath+fname, 'w') as f:
        print '创建文件:',fname
 
    # 获取文件列表，删除过期文件
    for f in os.listdir(fpath):
        ftime = datetime.datetime.strptime(f[:-4], '%Y%m%d%H%M%S')
        pt = now - ftime
        if pt.total_seconds() >= t_over:
            os.remove(fpath+f)
            print '删除文件:',f
 
    # 重新启动定时器
    timer = threading.Timer(t_gap, manage_file, [t_gap, t_over])
    timer.start()
 
if __name__ == '__main__':
    # 判断参数个数
    if len(sys.argv) != 3:
        print '参数错误.'
        exit()
 
    # 检查 data 文件夹是否存在，不存在则创建
    if not os.path.isdir('data'):
        os.mkdir('data')
 
    # 获取参数，1 秒后开启定时器
    gap = int(sys.argv[1])
    over = int(sys.argv[2])
    timer = threading.Timer(1, manage_file, [gap, over])
    timer.start()
```
