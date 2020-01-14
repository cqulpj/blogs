---
layout: post
title:  "webpy学习(四)、用户登录和注销实现"
author: lpj
categories: [ web.py, 技术 ]
image: assets/images/8.jpg
tags: featured
---

利用网络框架搭建博客、管理系统、网站等，都少不了登录、注销操作，其实原理很简单，就是登录后在session里保存登录信息，然后打开页面时通过检查这个信息来判断有没有登录、谁登录，最后注销的时候清除信息，下面就webpy如果实现用户登录、注销做一个简单介绍：

## 一、开启session支持

1. **关闭调试模式：**webpy默认开启调试模式，但session和调试模式下的重调用有冲突，所以使用session需要先关闭调试模式，代码如下：
    ```python
    web.config.debug = False
    ```

2. **使用session：**sessionx信息可以选择存在磁盘文件里或者数据库里，如果存入数据库，需要先连接数据库并创建sessions表，而选择存在磁盘文件里则比较简单，两种方式的代码如下：
    ```python
    # 在数据库中存储session信息
    create table sessions (
      session_id char(128) UNIQUE NOT NULL,
      atime timestamp NOT NULL default current_timestamp,
      data text
    );

    db = web.database(dbn='postgres', db='mydatabase', user='myname', pw='')
    store = web.session.DBStore(db, 'sessions')
    session = web.session.Session(app, store)

    # 在磁盘文件里存储session信息
    session = web.session.Session(app, web.session.DiskStore('sessions'))
    ```

3. **在模板中使用session：**有时需要在网页模板中访问session中的信息，比如网页代码中需要判断当前访问有无登录、登录名是什么，而这些信息是保存在session里的，所以需要在模板中获取session信息，对于webpy自带的模板系统，可以将session作为全局参数传给模板系统，对于jinja2模板系统，可以用全局函数的方式传进去，两种方式下的代码分别为：
    ```python
    # webpy自带模板系统访问session
    render = web.template.render('templates', globals={'context': session})

    # jinja2模板系统中访问session
    # 增加全局函数
    render._lookup.globals.update(
        session = session,)
    ```


## 二、登录验证并保存登录信息到session

1. **登录验证：**用户在登录页面的表单输入用户名和密码后点击“登录”，会触发post请求，请求中包含用户名、密码信息，所以登录页面对应的后台代码要先提取信息，然后对信息进行验证，验证通过则保存信息到session，验证失败则提示并按期望跳转，验证过程比较自由，但一般用户名和密码都保存在数据库中。

2. **保存登录信息：**在上一步验证通过后，需要将登录状态、用户名等信息保存到session，保存的字段名和信息格式也是自由裁定，一般用session.logged_in保存登录状态（是否登录），用session.user保存用户名，下面的代码演示了从获取数据到验证再到保存信息的整个过程：
    ```python
    # 登录
    class login:
        def GET(self):
            return render.login(LoginTips=u'请登录')

        def POST(self):
            i = web.input()
            user = i.get('Name')
            pswd = i.get('Password')
            print(user,pswd)
            ret = db.query_user_byname(user)
            if (ret!=None) and (ret.password == pswd):
                session.logged_in = True
                session.user = user
                session.userid = ret.id
                print('ok,loginin')
                raise web.seeother('/')
            else:
                print('fail,error')
                return render.login(LoginTips=u'用户名或密码错误')
   ````

## 三、使用session中的登录信息

将登录信息保存到session以后，在任何地方（后台、模板）就可以通过代码来判断是否登录已经哪个用户登录了，方式很简单，先判断session.logged_in是否为True，如果是表示已经登录，获取session.user就是当前登录的用户，若session.logged_in为False则表示当前未登录

## 四、注销

注销就是登录的反过程，所以只需要将seesion.logged_in置为False，然后清空session.user信息即可
  
  
