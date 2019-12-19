---
layout: post
title:  "webpy学习(三)、apache+mod_wsgi部署web.py生产环境"
author: lpj
categories: [ web.py, 技术 ]
image: assets/images/7.jpg
tags: featured
---

## 前言

web.py自带有用于调试的网络服务器，编码测试阶段使用是没有问题的，编码完成后呢，正式运行还是要用专门的网络服务器来处理高负载高并发（具体的我也不懂，反正意思就是正式运行还是用专业的网络服务器），常见的有apache、nginx、iis、lighthttp等，linux一般推荐nginx+fastcgi，windows一般推荐iis或者apache，本文只针对windows下web.py生产环境部署，重点考虑iis和apache，在网上找了很多iis搭建python网络框架的例程，都是针对flask或者django，所以选用apache+mod_wsgi来搭建，下面是具体步骤：

## 一、安装正确的apache版本

系统环境是win10 64位，python版本是2.7.10 32位，在cmd中启动python可以看到它对应的VC版本，这里显示是v.1500。该版本对应的是Visual C++ 2008 (9.0)，所以直接在网上下载apache 2.4 VC9 32位版本，下载后解压到C盘如C:\Apache24，将C:\Apache24\bin添加到系统path，然后就可以在cmd或powershell里启动apache了，主要用到以下几个命令：
```bash
httpd -k install  安装
httpd -k uninstall 卸载
httpd -k start 启动
httpd -k stop 停止
httpd -k restart 重启
```

执行httpd.exe -k install和httpd.exe -k start后，用浏览器打开`http://localhost`，如果能显示apache欢迎界面表示apache安装成功，否则到C:\Apache24\logs目录中查看error.log文件可以检查错误信息

## 二、安装对应的mod_wsgi版本

1. **下载mod_wsgi：**上一节安装了apache后，要根据apache版本、windows系统版本和python版本选择对应的mod_wsgi，打开mod_wsgi的[下载网址](https://www.lfd.uci.edu/~gohlke/pythonlibs/#mod_wsgi)，本文apache版本为2.4VC9、windows系统为64位、python为2.7 32位，所以下载mod_wsgi-4.6.8+ap24vc9-cp27-cp27m-win32.whl

2. **安装mod_wsgi：**在mod_wsgi-xxx.whl文件所在目录打开cmd或者powershell，执行`pip install mod_wsgi-4.6.8+ap24vc9-cp27-cp27m-win32.whl`即可完成mod_wsgi安装，安装后在Python安装目录的Scripts文件夹内会有mod_wsgi-express.exe文件，因为这个目录在安装python时已经加入到Path，所以打开cmd就可以随时执行mod_wsgi-express

3. **配置mod_wsgi：**打开cmd或powershell，执行`mod_wsgi-express module-config`，将会输出如下内容：

    ```bash
    C:\Python27\Scripts>mod_wsgi-express.exe module-config
    LoadModule wsgi_module "c:/python27/lib/site-packages/mod_wsgi/server/mod_wsgiNone"
    WSGIPythonHome "c:/python27"
    ```

    这是mod_wsgi的配置信息，不过生成的文件名有错误，应该是mod_wsgi.pyd，打开C:\Apache24\conf\httpd.conf文件，在LoadModule部分的最后加入以下两行：

    ```bash
    LoadModule wsgi_module "c:/python27/lib/site-packages/mod_wsgi/server/mod_wsgi.pyd"
    WSGIPythonHome "c:/python27"
    ```

    重启apache，如果不报错就表示mod_wsgi配置成功


## 三、修改项目py文件

1. **添加绝对路径：**打开之前编码好并调试通过的主文件如main.py，导入os和sys库，然后在开始添加以下代码：
 
    ```python
    abspath = os.path.dirname(__file__)
    sys.path.append(abspath)
    os.chdir(abspath)
    ```

2. **修改服务器启动部分：**一般主py文件最后的代码如下：

    ```python
    if __name__ == '__main__':
        app = web.application(urls, globals())
        app.run()
    ```

    现在改用apache+mod_wsgi模式启动，所以不需要app.run，同时要引入wsgi，所以代码修改为:
    ```python
    app = web.application(urls, globals()).wsgifunc()
    ```

## 四、修改httpd.conf文件，配置Apache虚拟服务器

1. 打开apache配置文件C:\Apache24\conf\httpd.conf，添加以下内容：

    ```xml
    AddType text/html .py
    ServerName localhost
    WSGIScriptAlias / D:/Works/WebPyWorks/note_apache_modwcgi/main.py/
    Alias /static/ D:/Works/WebPyWorks/note_apache_modwcgi/static/
    Alias /templates/ D:/Works/WebPyWorks/note_apache_modwcgi/templates/
    <Directory D:/Works/WebPyWorks/note_apache_modwcgi/>
        Order deny,allow
        Allow from all
    </Directory>
    ```
    其中项目路径请根据自己的实际情况进行替换

2. 重启apache，打开浏览器访问`http://localhost`或者`http://localhost:80`就可以看到效果了，上一步的配置代码还可以通过<VirtualHost>块来配置成其他端口，具体请自行查阅apache帮助文档

