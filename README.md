# 基于 GEC-6818 开发板的五子棋小游戏

> 制作人 ：梁振                    时间：2023年8月25日

### 1.概述

>这是一个基于 linux 平台的五子棋小游戏，主要的功能如下：
>
>* 实现一个简单的游戏开始界面
>* 实现五子棋游戏的基本规则，双人对战，可以实现投降，重新开始，和返回
>* 实现游戏的判断胜负和计分，并将成绩保存在外部文档 1.txt 中
>* 实现排行榜，可以将计分以黑点的形式打印在排行榜上
>* 实现背景音乐的实现，并且游戏退出，音乐停止
>* 实现蜂鸣器，一方 胜出，蜂鸣器报警

### 2.开发环境

>**vscode，ubuntu，gec6818**
>
>* vscode : 负责代码的编写，模块的分区
>* ubuntu:负责编译
>* gec6818：实现功能

### 3.游戏流程

#### 3.1 初始化设备屏幕

>使用linux内置文件接口完成屏幕的初始化
>
>* 用系统IO 函数 open 打开 设备文件 /dev/fb0
>* 计算需要的内存大小,调用mmap完成内存映射
>* 获取内存首地址指针,用于后续像素访问
>* 相关代码在 src 中的lcd.c中。

#### 3.2 相关图形的显示

>* 基于内存映射的指针,可以直接对内存进行读写操作,实现像素级的访问。
>
>* 例如 Lcd_Draw_point() 函数可以画单个像素。Lcd_background可以设置整个屏幕的背景色。
>
>* 基于像素级操作,实现了各种图形的绘制,如圆形、图片等。
>
>* 例如draw_round1可以半圆,Show_bmp() 可以绘制bmp图片。

#### 3.3 用触屏来接受玩家的指令

>* 用触摸屏来获取玩家点击的位置以达到不同的效果
>* src/chess.c 中实现了简单的下棋的步骤和判断
>* 用二维数组保存棋盘布局，记录每个位置棋子情况

#### 3.4 检查胜负

>遍历整个棋盘，当寻找到对应的棋子时开始判断。
>
>* 分别定义两个整数计数白子和黑子的相连棋子个数，为了方便之后的调用，定义在主函数中。
>
>* 判读棋子之后的各个方向棋子是否相等，相等计数等于相连棋子数，不相等计数清零。
>
>* 当计数达到5时，判定为胜利，计数不为零则返回计数位。不为5时计数位为0
>
>* 如果当前棋子投降，判定为另一方胜利。

#### 3.5 排行榜

>* 记录双方获胜次数,显示在排行榜界面。
>* 利用标准IO写入胜利次数
>
>* 获胜次数同时保存在外置文件“1.txt"中。

#### 3.6 线程函数

>* 用一个线程来播放背景音乐，游戏退出，音乐停止
>* 用另外一个线程来写蜂鸣器报警，一方胜利，蜂鸣器报警

### 3.主要功能函数

> ```c
> void Lcd_Draw_Point(int x,int y,unsigned int color)
> ```
>
>  * 功能：在指定的位置画像素点
>  * 参数：
>  * x,y：像素点的位置
>  * color：像素点的颜色值
>  * 根据坐标,直接设置对应显存位置的颜色值,就可以画出一个像素点。

>
>
>```c
>int chess(unsigned int color1,unsigned int color2,int num,Touch_lab *t)  
>```
>
>这个函数实现下棋逻辑,参数包括:
>
>\- color1/color2: 棋子的两种颜色
>
>\- num: 记录当前是第几步
>
>\-  t : 结构体存储坐标
>
>函数利用循环遍历棋盘的每个位置,判断点击位置是否有效,如果有效就在该位置画一个棋子,并记录到棋盘数组
>
>交替调用该函数,就可以实现黑白棋子的落子。



>```c
>int Black_Win(int sum1)  int White_Win(int sum2)
>@sum1:黑色棋子的连子个数
>@sum2:白色棋子的连子个数
>   返回值：返回各个棋子的连子个数
>```
>
>这两个函数实现 判断输赢
>
>判断黑白棋是否达成五子连线,并返回连线数。
>
>基本思路是遍历数组,找到己方棋子后,判断四个方向是否有相连的五子
>
>如果有，直接判为赢



> 标志IO 的相关函数，fputs fputc 将输赢获取输入到 1.txt 中
>
> getc 将胜场读取出来
>
> ```c
> /*
>  leaderboard：将排行榜打印
>      @victory1:黑棋胜场
>      @victory2:白棋胜场
>  返回值：无
> */
> void leaderboard(int victory1,int victory2)
> ```
>
> 这是一个排行榜打印函数：
>
> ​		用getc的返回值传入形参，每次读取之后，可以在排行榜上打印一个黑色圆圈，一个圆圈表示一个胜场



>```c
>void *play_music(void *arg)     //音乐播放线程函数，启动游戏就播放
>void *Beep(void *arg)   //蜂鸣器线程函数，一方胜出，蜂鸣器响
>```
>
>用设置标志位的方法 将线程函数的进行规划，识别到编制为，线程函数执行相应的操作

















