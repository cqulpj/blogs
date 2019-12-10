---
layout: post
title:  "webpy学习(二)、实现分页效果"
author: lpj
categories: [ web.py, 技术 ]
image: assets/images/6.jpg
tags: featured
---

## 前言

分页是网页中非常常见的空间，多用于表格内容过长需要分页显示、列表项太多需要分页显示，其形式一般是在数据（表格、列表）下部的一组横向排列的页码导航，在django、flask等python框架中有现成的pagination类可以用，web.py中没有，这里探讨如何实现：

## 一、修改bootstrap生成的分页代码实现导航

1. 在webpy学习(一)中我们讲过如何使用bootstrap可视化布局工具简化网页设计，在该工具中添加分页控件，然后复制它的代码到我们的模板html文件中，这时刷新网页（服务器端python程序保持运行）就可以看到页面上的分页了，但这时它只是静态的，不能实现导航，页面数也跟实际数据没有关联

2. 修改分页相关的html代码，按照jinja2模板语法（这里是使用的jinja2模板系统，如果使用web.py自带模板系统也对应修改）动态生成页数以及每一页的链接地址，此外还要对“前一页”、“后一页”的有效性进行判断并用CSS标记，最后还要对当前页进行突出显示，修改后的模板代码如下（需要page（当前页）和pages（总页数）两个变量）：
    ```html
    {% raw %}
    <ul class="pagination">
        {% if page==1 %}
            <li class="disabled">
                <a href="/">Prev</a>
            </li>
        {% else %}
            <li>
                <a href="/{{page-1}}">Prev</a>
            </li>
        {% endif %}
        {% for pi in range(1, pages+1) %}
            {% if pi==page %}
                <li class="active">
                    <a href="/{{pi}}">{{ pi }}</a>
                </li>
            {% else %}
                <li>
                    <a href="/{{pi}}">{{ pi }}</a>
                </li>
            {% endif %}
        {% endfor %}
        {% if page==pages %}
            <li class="disabled">
                 <a href="/">Next</a>
            </li>
        {% else %}
            <li>
                <a href="/{{ page+1 }}">Next</a>
            </li>
        {% endif %}
    </ul>
    {% endraw %}
    ```  
  
3. 将上面的代码直接嵌入模板html中，然后刷新页面，就可以看到分页效果了，如果觉得模板html里代码太乱或者想重用分页代码，可以将上面的代码单独保存成一个html文件比如pagination.html，然后在需要的地方`{% raw %}{% include 'pagination.html' %}{% endraw %}`进来即可

## 二、实现页码过多时缩略显示

经过上一节的步骤，现在已经可显示分页，但这里还有个问题，那就是页码会全部显示，如果10页以下还能接收（可能一行就能显示完），如果有上百页的话，这个分页显示就显得特别不科学了，现在常见的做法是把第一页、末页、当前页及前后几页显示页面，其他页面用省略号代替，这样不管有多少页都可以在一行显示完，下面探讨怎么实现：

1. 仔细看上一节中的代码，编译页码显示是range(1, pages+1)，那如果这里的列表只有我们想要显示的页码，不就可以只显示这些页码了吗，当然还要判断中间要不要显示省略号，flask的pagination类有个page_iter函数，可以根据给定的当前页、总页数生成一个列表如[1,None,3,4,5,None,9]，其中数字是要显示的页码，None是显示成省略号，但web.py中没有这个函数和类，所以我们需要手动实现它，所幸还不算难

2. 现在问题抽象为：要实现一个函数，输入当前页和总页数，按[1, 1, 3, 1]的格式生成结果列表，即首页、前一页、当前页及后面2页、尾页，下面是python代码实现：  

    ```python
    # 根据当前页数cp和总页数pages生成要显示的页码序列
    # 显示规则是1-1-3-1, 即：
    #   1、第一页显示
    #   2、当前页的前一页显示
    #   3、当前页+后面两页（共三页）显示
    #   4、末页显示
    # 如(cp, pages)=(4, 10),生成的结果形式为[1,None,3,4,5,6,None,10]
    # 其中数字为要显示的页码，None为中间省略显示的部分
    def page_iter(cp, pages):
        # 输入非法则返回空序列
        if (pages<=0) or (cp<=0) or (cp>pages):
            return []

        # 首页
        ret = [1]

        # 前页
        if (cp-1) > 2:
            ret.extend([None, cp-1])
        elif (cp-1) == 2:
            ret.extend([cp-1])

        # 当前及后面两页
        se = max(2, cp)
        me = min(cp+2, pages) 
        for i in range(se, me+1):
            ret.append(i)

        # 尾页
        if pages > (me+1):
            ret.extend([None, pages])
        elif pages == (me+1):
            ret.append(pages)

        return ret
    ```

3. 现在需要在上一节的模板代码中修改range(1, pages+1)为page_iter(page, pages)，但……慢着……，好像哪里不对，啊，模板中能直接使用这个函数吗，这个函数的文件放在哪里？嗯……好问题，这个函数放在一个.py文件（比如dealops.py）里就行，然后在主函数里添加以下代码将函数导入jinja2库就可用了
    ```python
    import dealops
    render = render_jinja(
        'templates',
        encoding='utf-8',
        )

    render._lookup.globals.update(
        page_iter = dealops.page_iter,
        )
    ```

4. 现在还得再修改一下pagination.html文件，首先是使用page_iter得到页码列表，其次是对列表中为None的页码显示为省略号，修改后的代码如下：
    ```html
    {% raw %}
    <ul class="pagination">
        {% if page==1 %}
            <li class="disabled">
                <a href="/">Prev</a>
            </li>
        {% else %}
            <li>
                <a href="/{{page-1}}">Prev</a>
            </li>
        {% endif %}
        {% for pi in page_iter(page, pages) %}
            {% if pi==None %}
                <li class="disable">
                    <a href="#">&hellip;</a>
                </li>
            {% elif pi==page %}
                <li class="active">
                    <a href="/{{pi}}">{{ pi }}</a>
                </li>
            {% else %}
                <li>
                    <a href="/{{pi}}">{{ pi }}</a>
                </li>
            {% endif %}
        {% endfor %}
        {% if page==pages %}
            <li class="disabled">
                 <a href="/">Next</a>
            </li>
        {% else %}
            <li>
                <a href="/{{ page+1 }}">Next</a>
            </li>
        {% endif %}
    </ul>
    {% endraw %}
    ```

5. 现在再刷新页码，就可以看到分页的最终显示效果了

## 三、扩展，封装成宏以方便重用

1. 虽然上一节把分页代码弄到pagination.html中，在主页面模板中使用时include进来感觉挺方便了，但是这样使用还是不够方便，因为分页里的page和pages是用的全局的，设想一下假如页面上有多个表格需要用到分页，而每个表格又是独立的，那这时刚才那种作法就行不通了，所以更有效的方法是封装成带参数的宏，使用时导入并传参给它自动生成对应的代码，jinja2模板系统中的宏定义和使用如下，请自行修改代码，这里不赘述了：
    ```html
    {% raw %}
    <!--定义宏-->
    {% macro input(name, value='', type='text') %}
    <input type="{{ type }}" name="{{ name }}" value="{{
    value|e }}">
    {% endmacro %}

    <!--使用宏-->
    <p>{{ input('username') }}</p>
    <p>{{ input('password', type='password') }}</p>
    
    <!--在其他文件中使用宏-->
    {% import 'forms.html' as forms %}  //导入宏文件
    <p>{{ forms.input('username') }}</p>
    <p>{{ forms.input('password', type='password') }}</p>
    {% endraw %}
    ```

