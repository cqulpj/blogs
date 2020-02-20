---
layout: post
title:  "华容道游戏实现--pygame设计"
author: lpj
categories: [ python, pygame ]
image: assets/images/9.jpg
tags: featured
---

## 前言

受疫情影响，从初一就开始了苦逼的停休模式，没有周末、没有假期，值班、备勤、守卡……，看网友们都发牢骚说在家闲的发疯，莫名的很羡慕他们，不过想想湖北等疫情重灾区的人们，还是祈祷病毒早点儿消退，社会、生活、工作早点儿回到正轨……

前两天工作实在烦闷了，决定写一下华容道游戏，一是复习一下pygame并练练手，二是再重新写一下华容道的破解算法，所以分成两篇日志来描述，这一篇主要是用pygame实现华容道游戏。关于pygame的学习从博主xishui的教程[《用Python和Pygame写游戏-从入门到精通》](https://eyehere.net/2011/python-pygame-novice-professional-index/)受益颇多，在此表示感谢，也推荐给有兴趣的朋友作为参考。

## 一、总体思路

华容道游戏逻辑简单，界面元素也不复杂，从游戏过程角度可以分为选关(主界面)、走棋(用户走棋)、解密(程序演示棋局解法)三个阶段，每个阶段的内容及界面元素详细如下：

* **选关阶段：**也是游戏主界面，在该界面可以通过点击左右按钮切换上一关、下一关，同时界面会显示该关的棋局名称和棋局样式，切换到期望的关卡后，可以点击“开始”按钮开始走棋，也可以点击“解密”按钮观看程序对该棋局的自动解密演示；  
  
* **走棋阶段：**在该阶段，玩家可以拖拉棋盘中的角色移动位置，以达到把“曹操”移出来的目的，除此之外，该阶段的界面还要有重置棋局、悔棋（撤销）、返回主界面（选关）的功能按钮，为了增加游戏体验，该阶段还要对玩家耗时和走棋步数进行统计显示，在玩家最终过关时，要设计效果较好的过关动画和音效；  
  
* **解密阶段：**该阶段主要针对玩家对某一棋局屡战屡败、一筹莫展时观看解法用的，在该阶段，程序定时自动移动角色，一步步对棋局进行解密，为了方便玩家观察、琢磨，该解密有暂停/恢复、返回主界面和悔棋（撤销、返回上一步）三个功能按钮。

## 二、素材准备

根据上一节的描述，需要准备棋盘、角色（小兵、横向的大将、竖向的大将、曹操）、按钮（撤销、主页、重置、暂停/继续、上一关、下一关、开始、解密）等图片素材，为了提高游戏体验，还要准备背景音乐、走棋声音(duang)、过关欢呼鼓掌声等声音素材，此外要显示步数、耗时和关卡名称，要用的字体文件（可以使用系统字体，但这样在别的电脑上运行时，可能因为找不到该字体而导致显示异常），所以还要准备字体素材，此处推荐使用文泉驿微米黑，字体文件较小且显示效果几乎与微软雅黑相同；声音素材可以去网上搜索下载，然后根据需要编辑转换格式，图片素材推荐使用PS制作。

## 三、棋局、角色、按钮等数据结构定义

1. **棋局：**棋局有棋局数据和棋局类两种，前者是用单纯的数据表示，比如一个矩阵或二维数组，后者是一个类，不光包括棋局数据，还包括可显示、可交互的棋局角色，每个角色有图片、位置等属性，可以理解成后者是指定了外观、独立了各角色属性的前者，对应的代码分别如下：

    ```python
    # 棋局数据定义
    # 对于多格角色，只有左上角格用K、H、V，其他占位格用S
    K = 7       # king，棋局中的'曹操'
    H = 2       # 长度为2的横向元素，棋局中的横向大将
    V = 3       # 长度为2的纵向元素，棋局中的纵向大奖
    P = 4       # 长宽都为1的元素，棋局中的小兵
    B = 0       # 长度都为1的空白位置
    S = 1       # 多个角色的占位格

    # 存储常见的棋局及名称
    chrs = [
        {
            'name':u'1. 三羊开泰',
            'data':
               [[P, K, S, P],
                [V, S, S, V],
                [S, H, S, S],
                [H, S, H, S],
                [B, P, P, B]]
        },
    ```

    ```python
    # 棋局类
    class Aspect():
    
        def __init__(self, target):
            self.surface = target
            self.kboard = None
            self.elets = []

        # 从一个二维数组生成棋局
        def load_board(self, pdata):

        # 绘制棋局元素
        def draw(self):

        # 打印棋局
        def display(self):

        # 判断结束
        def is_finished(self):

        # 返回棋局数据(不含角色元素)
        def get_data(self):

        # 清空棋局
        def clear(self):
    ```

2. **角色：**棋局中的某个角色，有小兵、横向的大将、竖向的大将、曹操(king)等，这里的角色类还指定了它关联的图片和棋局，以便它在棋局上进行绘制，此外还有可否向某个方向移动等函数，代码如下：
    ```python
    # 角色类
    class Elet(object):

        # 角色的宽高格数
        tsize = { K:(2, 2), H:(2, 1), V:(1, 2), P:(1, 1)}
        
        # 初始化
        # target : 对应的surface
        # type ：角色类型，数字2、3、4、7，意义见上文宏定义
        # loct : 逻辑位置，整个棋盘为5x4矩阵，loct为矩阵行列，如(0, 2)表示第一行第三列
        # kboard ：当前棋局数据，为一个二维数组，意义见'棋局数据宏定义'
        def __init__(self, target, type, loct, kboard):
            self.surface = target
            self.type = type
            self.kboard = kboard
            self.loct = loct
            self.image = pygame.image.load(t2img_dict[type]).convert_alpha()
        
        # 序列化，用于打印显示
        def __str__(self):

        # 在surface上绘制角色
        def draw(self):

        # 判断某坐标点是否在角色范围内
        def is_over(self, pos, WID=100, HID=100):

        # 将角色向某方向移动一格
        # dest : 方向，取值为'up', 'left', 'right', 'down'
        def move_once(self, dest):

        # 判断角色是否能向某个方向移动
        # dest : 方向，取值为'up', 'left', 'right', 'down'
        def can_move(self, dest):
    ```

3. **按钮：**pygame库没有封装好的按钮类，所以准备好按钮对应的图片素材后，需要定义按钮类，然后在实例化的时候指定按钮位置和图片，值得说明的是，有一类按钮比较特殊，即暂停/继续按钮，该按钮会根据当前状态确定显示的图片，所以对它进行单独的定义，相关定义代码如下：

    ```python
    # 按钮类
    class Button(object):
        def __init__(self, image_filename, position):
            self.pos = position
            self.image = pygame.image.load(image_filename)

        def draw(self, surface):
            surface.blit(self.image, self.pos)

        # 如果point在自身范围内，返回True
        def is_over(self, pos):
            pos_x, pos_y = pos
            x, y = self.pos
            w, h = self.image.get_size()
            return (x < pos_x <= x+w) and (y < pos_y <= y+h)

    # 播放/暂停按钮类
    class PlayPauseButton(Button):
        def __init__(self, pauseimg, playimg, position):
            Button.__init__(self, pauseimg, position)
            self.pauseimg = pauseimg
            self.playimg = playimg
            self.status = True

        def reverse(self):
            if self.status:
                self.image = pygame.image.load(self.playimg)
                self.status = False
            else:
                self.image = pygame.image.load(self.pauseimg)
                self.status = True
        
        def get_status(self):
            return self.status
    ```
    
## 四、其他细节

定义了上述数据结构后，除了解密算法，华容道的游戏部分已经基本完成了，主函数就是一个while死循环，里面根据当前所处的阶段来响应鼠标点击，如果是点击了按钮就执行对应的操作，如果是拖动了角色就判断角色能否移动，能就移动角色并更新棋局数据，如果角色King(曹操)走出来了则游戏过关。这里主要对一些需要提及的实现细节进行代码示例:

1. **按钮点击判断:**把每个阶段要显示的按钮实例化到一个字典里，有鼠标点击时，判断是哪个按钮区域并记录，然后执行对应操作

    ```python
    # 棋局选择界面的按钮
    buttons = {}
    buttons['pre'] = Button(prebtnimg, (BLD_x, BLD_y))
    buttons['next'] = Button(nextbtnimg, (BLD_x+320, BLD_y))
    buttons['start'] = Button(startbtnimg, (BLD_x, BLD_y+60))
    buttons['answer'] = Button(answerbtnimg, (BLD_x+200, BLD_y+60))
    
    ...

    for event in pygame.event.get():
        if event.type == MOUSEBUTTONDOWN:
            # 判断哪个按钮被按下
            for button_name, button in buttons.iteritems():
                if button.is_over(event.pos):
                    button_pressed = button_name
                    break

    if button_pressed is not None:
        if button_pressed == 'pre':
            cpinx = (cpinx-1) if (cpinx>0) else (len(chrs)-1)
        if button_pressed == 'next':
            cpinx = (cpinx+1) if (cpinx<len(chrs)-1) else 0
        if button_pressed == 'start':
            run_mode = 1
        if button_pressed == 'answer':
            run_mode = 2
    ```

2. **声音播放：**声音播放有两种，一种是背景音乐，使用pygame.mixer.music模块来播放，但这个模块不能叠加播放，所以比较适合背景音乐，不适合走棋过程中的动态音效；另一种就是走棋动态音效，比如棋子移动的撞击声、游戏过关的庆祝声，这种使用pygame.mixer.Sound模块来实现，示例代码如下：

    ```python
    # 加载背景音乐并播放
    pygame.mixer.music.load(bgsound)
    pygame.mixer.music.play(loops=-1)

    # 加载撞击音效
    dong = pygame.mixer.Sound(htsound)

    # 加载鼓掌音效
    wa = pygame.mixer.Sound(wasound)

    ... 

    dong.play()
    ```

3. **拖动棋子：**pygame监测的事件中有鼠标按下、放开、移动等，但没有拖动事件，所以需要自己来实现，这里的实现思路是：当有鼠标按下时记录点击的角色，然后鼠标移动时，判断之前有无角色被点击（按下），若有，则根据移动的方位判断偏向于上下左右哪个方向，如果该角色可以向这个方向移动则直接移动过去，示例代码如下：

    ```python
    # 捕获事件
    for event in pygame.event.get():
        if event.type == pygame.MOUSEBUTTONDOWN:
            if event.button == 1:
                pos = event.pos
                for e in chessp.elets:
                    if e.is_over(pos):
                        is_press = True
                        elet_pressed = e
                        break

        elif event.type == pygame.MOUSEMOTION:
            if event.buttons[0] and is_press:
                rel = event.rel
                dest = None
                if abs(rel[0])>abs(rel[1]) and (abs(rel[0]) > 3):
                    dest = 'right' if rel[0]>0 else 'left'
                elif abs(rel[0])<abs(rel[1]) and (abs(rel[1]) > 3):
                    dest = 'down' if rel[1]>0 else 'up'
                
                if (dest!=None) and elet_pressed.can_move(dest):
                    dong.play()
                    elet_pressed.move_once(dest)
                    pre_steps.append((elet_pressed, dest))
                    steps += 1
                    chessp.display()
                    is_press = False
                    elet_pressed = None
    ```

4. **定时器：**在游戏中两处需要用到定时器，一处是走棋阶段需要显示玩家耗时，显示的时间格式为MM:SS，每秒更新一次，另一处是解密阶段，程序定时（1-2S）走一步期，python核心库有定时器可以使用，不过既然是用pygame实现，推荐使用pygame的定时器，示例代码如下：

    ```python
    # 创建定时器事件
    ONESECOND = USEREVENT + 1
    pygame.time.set_timer(ONESECOND, 1000)
    
    # 捕获事件
    for event in pygame.event.get():
        if event.type == ONESECOND:
            secs += 1
            mins, secs = (mins+1, 0) if secs>=60 else (mins, secs)
            mins, secs = (0, 0) if mins>=60 else (mins, secs)
    ```

## 五、总结

华容道游戏的pygame设计实现介绍到这里，工程打包上传到了csdn（[下载地址](https://download.csdn.net/download/xiaomiaolpj/12169543)）提供下载，解密算法下一篇再详细讲解。
    


