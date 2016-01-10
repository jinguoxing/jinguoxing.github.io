---
layout: post
title: "Mac里面Sublime Text 3的常用快捷键"
description: ""
author: "静中细思"
category: [Sublime,mac]
tags: 
    - Sublime Text 3
    - mac
---

###Sublime Text3 插件的安装

方法1： 通过 [Package Control](http://wbond.net/sublime_packages/package_control)
       
        a. 打开控制命版(Ctrl+Shift+P on Windows and Linux. ⌘+⇧+P on OS X) 
        b. Type "Install" and select "Package Control: Install Package".
        c. 选择你要的包 自动安装
方法2： 下载安装包或者使用Git

        Without Git: Download the latest source code, and extract to the Packages directory.
        With Git: Type the following command in your Sublime Text Packages directory:
        git clone git://github.com/timonwong/OmniMarkupPreviewer.git

    The "Packages" directory for Sublime Text 3 is located at:
        Windows: %APPDATA%\Sublime Text 3\Packages\
        Linux: ~/.config/sublime-text-3/Packages/
        OS X: ~/Library/Application Support/Sublime Text 3/Packages/


###一些常用的快捷键

-----------------

`⌘（command）+D`  选词（反复按快捷键，即可继续向下同时选中下一个相同的文本进行同时编辑） 

`⌘（command）+P`  搜索项目中的文件  

`⌘（command）+ CTRL + p`  打开项目列表面板  
```
Ctrl + `
```
命令控制台   

    ⌘（command）+ W 关闭当前打开文件
    ⌘（command）+ Shift + W 关闭所有打开文件
    ⌘（command）+L 选择行，重复可依次增加选择下一行
    ⌘（command）+Shift+ L 选择多行
    ⌘（command）+Shift+Enter　在当前行前插入新行
    ⌘（command）+X 删除当前行
    ⌘（command）+M  最小化窗口
    ⌘（command) +U  软撤销，撤销光标位置
    ⌘（command) +J 选择标签内容 
    ⌘（command) +F 查找内容
    ⌘（command) +Shift+F 查找并替换
    ⌘（command) +H 替换
    ⌘（command) +R 前往 method的函数
    ⌘（command) +N 新建窗口
    ⌘（command) +数字 窗口切换
    ⌘（command) + Alt + 2    分成两屏
    control + 数字    分屏时移动到不同的屏幕
    ⌘（command) +K+B 开关侧栏
    ⌘（command) +fn+F2 设置/删除标记
    ⌘（command) + /  注释当前行
    CTRL +M 跳转到对应括号
    alt+ ⌘（command)+/ 当前位置插入注释
    Ctrl+Shift+A　选择当前标签前后，修改标签用的
    Alt+F3　选择所有相同的词
    Alt + .  闭合标签
    Alt+Shift+数字 分屏显示
    Shift+右键拖动  可切换Tab文件
    按Ctrl，依次点击或选取，可需要编辑的多个位置
    按Ctrl+Shift+上下键，可替换行
    按Ctrl，依次点击或选取，可需要编辑的多个位置
    按 ⌘（command)键 ，依次点击或选取，可需要编辑的多个位置
    Ctrl+G 跳转到第几行


    //按Ctrl+Shift+上下键，可替换行


    shift + cmd + p 打开命令面板
    control + ` 控制台
    cmd + n 新建标签
    cmd + 数字    标签切换
    cmd + option + 2    分成两屏
    control + 数字    分屏时移动到不同的屏幕
    cmd + delelte   删除光标前所有字符, 貌似是Mac快捷键
    cmd + f 查找
    option + cmd + f    查找替换
    cmd + t 文件跳转
    control + g 行跳转, 类似vim中的num + gg
    cmd + r 函数跳转
    cmd + / 给选中行添加或去掉注释
    cmd + [或 cmd + ]    智能行缩进
    cmd + k + b 开关侧边栏
    cmd+ctrl+p 获取已保存项目
    cmd + p 搜索打开的文件
    CMD+CTRL+↓或CMD+CTRL+↑上下移动当前行。
    CMD+K+B  隐藏、显示侧边栏




    ctrl+m 光标移动至括号内结束或开始的位置。

    command +K+K 从光标处开始删除代码至行尾。
    Ctrl+Shift+K 删除整行


    command + k + u  转换大写。
    command + k + l  转换小写。_



    选择类

    Ctrl+D 选中光标所占的文本，继续操作则会选中下一个相同的文本。

    Alt+F3 选中文本按下快捷键，即可一次性选择全部的相同文本进行同时编辑。举个栗子：快速选中并更改所有相同的变量名、函数名等。

    Ctrl+L 选中整行，继续操作则继续选择下一行，效果和 Shift+↓ 效果一样。

    Ctrl+Shift+L 先选中多行，再按下快捷键，会在每行行尾插入光标，即可同时编辑这些行。

    Ctrl+Shift+M 选择括号内的内容（继续选择父括号）。举个栗子：快速选中删除函数中的代码，重写函数体代码或重写括号内里的内容。

    Ctrl+M 光标移动至括号内结束或开始的位置。

    Ctrl+Enter 在下一行插入新行。举个栗子：即使光标不在行尾，也能快速向下插入一行。

    Ctrl+Shift+Enter 在上一行插入新行。举个栗子：即使光标不在行首，也能快速向上插入一行。

    Ctrl+Shift+[ 选中代码，按下快捷键，折叠代码。

    Ctrl+Shift+] 选中代码，按下快捷键，展开代码。

    Ctrl+K+0 展开所有折叠代码。

    Ctrl+← 向左单位性地移动光标，快速移动光标。

    Ctrl+→ 向右单位性地移动光标，快速移动光标。

    shift+↑ 向上选中多行。

    shift+↓ 向下选中多行。

    Shift+← 向左选中文本。

    Shift+→ 向右选中文本。

    Ctrl+Shift+← 向左单位性地选中文本。

    Ctrl+Shift+→ 向右单位性地选中文本。

    Ctrl+Shift+↑ 将光标所在行和上一行代码互换（将光标所在行插入到上一行之前）。

    Ctrl+Shift+↓ 将光标所在行和下一行代码互换（将光标所在行插入到下一行之后）。

    Ctrl+Alt+↑ 向上添加多行光标，可同时编辑多行。

    Ctrl+Alt+↓ 向下添加多行光标，可同时编辑多行。

    编辑类

    Ctrl+J 合并选中的多行代码为一行。举个栗子：将多行格式的CSS属性合并为一行。

    Ctrl+Shift+D 复制光标所在整行，插入到下一行。

    Tab 向右缩进。

    Shift+Tab 向左缩进。

    Ctrl+K+K 从光标处开始删除代码至行尾。

    Ctrl+Shift+K 删除整行。

    Ctrl+/ 注释单行。

    Ctrl+Shift+/ 注释多行。

    Ctrl+K+U 转换大写。

    Ctrl+K+L 转换小写。

    Ctrl+Z 撤销。

    Ctrl+Y 恢复撤销。

    Ctrl+U 软撤销，感觉和 Gtrl+Z 一样。

    Ctrl+F2 设置书签

    Ctrl+T 左右字母互换。

    F6 单词检测拼写

    搜索类

    Ctrl+F 打开底部搜索框，查找关键字。

    Ctrl+shift+F 在文件夹内查找，与普通编辑器不同的地方是sublime允许添加多个文件夹进行查找，略高端，未研究。

    Ctrl+P 打开搜索框。举个栗子：1、输入当前项目中的文件名，快速搜索文件，2、输入@和关键字，查找文件中函数名，3、输入：和数字，跳转到文件中该行代码，4、输入#和关键字，查找变量名。

    Ctrl+G 打开搜索框，自动带：，输入数字跳转到该行代码。举个栗子：在页面代码比较长的文件中快速定位。

    Ctrl+R 打开搜索框，自动带@，输入关键字，查找文件中的函数名。举个栗子：在函数较多的页面快速查找某个函数。

    Ctrl+： 打开搜索框，自动带#，输入关键字，查找文件中的变量名、属性名等。

    Ctrl+Shift+P 打开命令框。场景栗子：打开命名框，输入关键字，调用sublime text或插件的功能，例如使用package安装插件。

    Esc 退出光标多行选择，退出搜索框，命令框等。

    显示类

    Ctrl+Tab 按文件浏览过的顺序，切换当前窗口的标签页。

    Ctrl+PageDown 向左切换当前窗口的标签页。

    Ctrl+PageUp 向右切换当前窗口的标签页。

    Alt+Shift+1 窗口分屏，恢复默认1屏（非小键盘的数字）

    Alt+Shift+2 左右分屏-2列

    Alt+Shift+3 左右分屏-3列

    Alt+Shift+4 左右分屏-4列

    Alt+Shift+5 等分4屏

    Alt+Shift+8 垂直分屏-2屏

    Alt+Shift+9 垂直分屏-3屏

    Ctrl+K+B 开启/关闭侧边栏。

    F11 全屏模式

    Shift+F11 免打扰模式
