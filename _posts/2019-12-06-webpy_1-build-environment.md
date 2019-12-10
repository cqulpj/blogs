---
layout: post
title:  "web.py学习(一)、搭建web.py+Jinja2+bootstrap环境"
author: lpj
categories: [ web.py, 技术 ]
image: assets/images/4.jpg
tags: featured
---

## 前言

忙里偷闲想看看python的web框架以备不时之需，对比了web框架、模板框架、前端框架等各种工具、库、例程后，准备采用web.py + Jinja2 + bootstrap模式来作为基本框架，下面记录环境搭建步骤：

>本文主要针对web设计零基础的看客，也是为了记录一下以方便自己回顾，所以资深web设计者请勿见笑

## 一、web.py简介及安装

1. 本来想直接写安装的，想了想还是写写web.py吧，目前python网络框架主要有django、flask、web2py（注意区别web.py，不是同一个）、web.py等，其中django大而全，但入门难度比较大，flask比较小巧灵活，社区支持也比较好，如果没有特别倾向的也特别推荐试试这个，web2py好像不支持python3（具体没验证，这个没去试过），web.py最pythonic，功能强大，使用简单，推荐入门学习，但因为太小，所以很多组件都没有，有时需要自己实现某些常用功能。

    多说一句，web.py目前版本是0.40，2012年它的版本是0.37，也就是说近7年来web.py几乎没有更新，因为它的作者——美国作家、Reddit联合创始人、RSS规格合作创造者、著名计算机黑客、非盈利组织Open Library创始人、反抗互联网审查法案SOPA/PIPA运动发起人Aaron Swartz于2013年自杀离世（26岁）（来龙去脉请自行搜索，在此不再详述），0.37版本的web.py在作者离开的情况下能在十几个python网络框架中至今仍在一席之地，足以证明它的强大和价值，**谨此纪念天才程序员、自由斗士Aaron Swartz……**  

2. 安装web.py
    * python 2.7安装web.py 0.38版本
      ```bash
      pip install web.py==0.38
      ```

    * python 3.x安装web.py 0.40版本（最新版本）
      ```bash
      pip3 install web.py==0.40
      ```

3. 简单测试
    * 新建文件hello.py，粘贴以下内容到文件中并保存退出
      ```python
      import web  

      urls = (  
          '/(.*)', 'hello'  
      )  

      class hello:  
          def GET(self, name):  
              i = web.input(times=1)  
              if not name:   
                  name = 'world'  
              return 'Hello, ' + name*int(i.times) + '!'  

      app = web.application(urls, globals())  

      if __name__ == "__main__":  
          app.run()  
      ```

    * 执行该文件`python hello.py`，然后打开浏览器访问`localhost:8080`,如果页面显示Hello, world!表示运行成功，还可以试试`localhost:8080/name`和`localhost:8080/name?times=3`看看浏览器的显示结果

4. 使用模板

    在上一步里可以在浏览器中看到打印的字符，但通常浏览器显示的内容是HTML，比单纯的文本内容丰富且更加美观，那怎么显示HTML呢，一个办法时在return时直接返回HTML代码，但这样代码会非常凌乱而且可重用性非常非常差，对于大多数网络应用（网站、CMS等）来说，可能只有几类页面或HTML片段，如博客有首页、博文列表页、文章详情页几类，那如果事先做好一个HTML文件，能不能在return里直接调用呢，当然能，这就是模板，这里我们先演示一下基础的HTML模板使用，至于带python代码的模板下一节再详细介绍

    * 在hello.py所在目录新建index.html文件，用文本编辑器打开，粘贴以下内容：
      ```html
      <!doctype html>
      <html>
          <head>
              <title>你好</title>
          </head>
          <body>
              <h1> Webpy, 模板 </h1>
              <ul>
                  <li>第一行</li>
                  <li>第二行</li>
              </ul>
          </body>
      </html>
      ```
    
    * 修改hello.py文件内容如下：
      ```python
      import web  
      
      urls = (  
          '/(.*)', 'hello'  
      )  
      
      render = web.template.render('')
      
      class hello:  
          def GET(self, name):  
              return render.index()
      
      app = web.application(urls, globals())  
      
      if __name__ == "__main__":  
          app.run()  
      ```
    
    * 运行hello.py文件，再用浏览器打开`localhost:8080`，就能看到index.html的呈现效果了
  

## 二、Jinja2模板系统安装使用

在简介模板系统前，先思考一个问题：在第一节里我们安装了web.py，并且利用模板显示了index.html网页，但这个网页里的内容是固定的，能不能让里面的数据动态更新呢，比如刚才那个无序列表，能不能依次显示姓名、日期、地址呢？答案是能！就是修改index.html文件，按模板系统语法把变量嵌入到网页里，然后使用的时候把变量传参进去，web.py自带有一套模板语法，用法参见[web.py自带模板系统](http://webpy.org/docs/0.3/templetor.zh-cn)，也比较好用，但Jinja2模板系统更为常见，也被多个python网络框架如flask、django所支持，本节简单讲述如何安装Jinja2模板系统以及在web.py工程中如何使用

1. Jinja2安装
    ```bash
    pip install Jinja2
    ```

2. 在web.py工程中使用Jinja2

    * 修改hello.py文件内容为以下：
      ```python
      import web  
      from web.contrib.template import render_jinja
      
      urls = (  
          '/(.*)', 'hello'  
      )  
      
      render = render_jinja('', encoding='utf-8',)
      
      class hello:  
          def GET(self, name):  
              return render.jinja_index(name=u'张三', 
                      date=u'2019-12-06 15:21', 
                      addr=u'中国')
      
      app = web.application(urls, globals())  
      
      if __name__ == "__main__":  
          app.run()  
      ```

    * 复制一份index.html文件，重命名为jinja_index.html，修改内容为以下：
      ```html
      {% raw %}
      <!doctype html>
      <html>
          <head>
              <meta http-equiv='Content-Type' content='text/html; charset=utf-8'>
              <title>你好</title>
          </head>
          <body>
              <h1> Webpy, 模板 </h1>
              <ul>
                  <li>{{ name }}</li>
                  <li>{{ date }}</li>
                  <li>{{ addr }}</li>
              </ul>
          </body>
      </html>
      {% endraw %}
      ```

    * 运行hello.py文件，再用浏览器打开`localhost:8080`，就能看到变量已经在网页里显示了，这是最基础的Jinja2模板系统使用，详细的Jinja2教程见[Jinja2使用](https://www.cnblogs.com/yanzi-meng/p/8342798.html)



## 三、Bootstrap使用

其实用前面两节的web.py和Jinja，已经可以做出一个满足业务需要的web系统了，但为了系统美观，需要使用HTML、CSS等对页面元素做大量的修饰，bootstrap等前端框架是一组css和js集合，里面涵盖了大量空间装饰的现成代码，因此学习一下如何使用可以在后面的web系统设计中节省很多界面美化的工作量

1. Bootstrap使用

    一般来说，要使用bootstrap只需在页面中引入bootstrap.min.css和bootstrap.min.js，其中bootstrap.min.js以来jQuery库，所以还要引入jquery.min.js文件，这三个文件可以下载了存在在本地，也可以直接用网络路径，比如在index.html文件中`<head>...</head>`间加入以下代码就可以使用CDN上的bootstrap相关文件:
    ```html
    <!-- 新 Bootstrap 核心 CSS 文件 -->
    <link href="https://cdn.staticfile.org/twitter-bootstrap/3.3.7/css/bootstrap.min.css" rel="stylesheet">
    <!-- jQuery文件。务必在bootstrap.min.js 之前引入 -->
    <script src="https://cdn.staticfile.org/jquery/2.1.1/jquery.min.js"></script>
    <!-- 最新的 Bootstrap 核心 JavaScript 文件 -->
    <script src="https://cdn.staticfile.org/twitter-bootstrap/3.3.7/js/bootstrap.min.js"></script>
    ```

    引入了bootstrap文件后就可以在html文件中对页面元素使用bootstrap类了，具体见[Bootstrap全局CSS样式](https://v3.bootcss.com/css/)

2. Bootstrap可视化布局工具

    虽然上一步中已经可以直接对页面元素使用bootstrap类进行修饰以美化界面了，但对于我这种完全没做过前端，极其不熟悉CSS、HTML的人来说，要设计一个完整的页面还是相当抽象，还好经过搜索找到一个[可视化的Bootstrap布局工具](http://www.ibootstrap.cn/)，使用很简单，按自己想要的效果把页面元素摆放好后，点击上部的“下载”按钮，就可以看到生成的HTML代码，将代码复制到自己的html文件中即可，这个工具并不完善，只有元素的基础属性，不能直接修改属性看到效果，但用来布局和做基础的页面设计已经很顺手了

3. 自定义CSS

    累了，先不写了，以后再补充
