# CPU基本电路的实现
>本文是对B站UP踌躇月光出的8位二进制CPU的设计和实现的文字教程复现第一部分 CPU基本电路的实现
相关 github 地址：https://github.com/StevenBaby/computer
PS：有错误的地方请指正，谢谢！共同学习，一起进步！

# 概述
> 这个视频主要是分享8位二进制CPU的设计和实现。
首先，解释一下为什么做这个视频。众所周知，计算机专业众多课程中有一门课是计算机组成原理，这门课程实际上并不那么好理解，因为他涉及到了硬件。但是组成原理这门课又没有对硬件部分做充分的描述，导致这门课没有其他课那么容易学习，特别是中央处理器(CPU)部分的实现。那么我们一起来实现一个8位二进制CPU，就可以更加自然的理解计算机组成原理究竟说了些什么。
其次，关于做CPU或者说做一台计算机，做一台什么样的计算机的问题，图灵(1912-1954)给出过答案，图灵在他的论文<可计算性>中给出了什么样的事情是计算机可以做的，什么是做不到的，以及给出了计算机的抽象模型-图灵机，其中就描述了计算机和机器的区别，区别就是支持任何条件转移指令的机器就是计算机，也就是说我们要做一个支持条件转移的计算机，在这篇论文中还介绍了另一种测试计算机的方法，这就是人们熟知的图灵测试。最早引入条件转移指令的人是英国数学家和经济学家查尔斯.巴贝奇(1792-1871)，大约在19世纪，巴贝奇就有了制造解析机的想法，不过巴贝奇的解析机并没有在他有生之年实现。那么我们要做的CPU就是可以支持条件转移指令的机器，这里推荐大家两本书，一本是查尔斯的《编码：隐匿在计算机软硬件背后的语言》，也是这本书让up有了想做此视频的想法，第二本是李忠的《穿越计算机的迷雾》，可以作为前一本书的补充，值得大家一读再读。以上两本就作为教材了，如果想尽快了解做什么，可以去下载这两本书先读一读。工欲善其事必先利其器，本教程使用一个电路仿真的软件logiccircuit(下面会提到)来制作CPU电路。

# 准备工作
- 小白可以看看上面的两本书的前面部分或者快速入门数电基础
- 下载仿真软件logiccircuit

## logiccircuit软件使用方法
打开软件，主窗口由以下几个部分组成
- 软件提供模块：软件自身提供的一些基本输入输出单元和基本模块
- 自建模块：通过下面的软件提供模块搭建的特殊功能模块
- 搭建操作窗口：拖动模块连线和仿真展示的窗口
- 开启仿真按钮：模块搭建和连线之后开启仿真的按钮
### 输入输出单元
- 输入单元
  - 位宽：可选位宽如1位，8位
  - 边：搭建模块时选择接口位置
- 输出单元
  - 位宽：可选位宽如1位，8位
  - 边：搭建模块时选择接口位置
- 按钮/切换器
  - 符号：名字是模块内部用的，特别是展示真值表时显示。符号是模块上表示该引脚的一个代号。
  - 切换器：切换器是锁定状态的，和按钮不同，按钮按下是一个状态，弹开是一个状态，切换器每按下弹起一次切换一次状态
  - 针脚：和边类似，搭建模块时选择接口位置
- 常量：和输入单元差不多，只不过是常量
- 传感器：暂时不涉及
- 时钟：提供时钟信号
- 分路器：用来合并分路针脚为总线
  - 分路针脚数目：引脚多的一端的针脚数
  - 分路针脚位宽：引脚多的一端的单个针脚的位宽
  - 组合针脚位置：单总线的位宽=分路针脚数目x分路针脚位宽，位置决定是输入还是输出，有三角标志的是低位
- LED：方便显示逻辑电平输出，1 亮 0灭
- 7段数码管：显示数字
- LED矩阵
- 图形序列
- 蜂鸣器
- 探针：方便看某一处的电平输出
### 基本元件
- 非门
  - 逻辑表达式：L = !A
  - 真值表
    | A | L |
    | - | - |
    | 0 | 1 |
    | 1 | 0 |
- 与门
  - 逻辑表达式：L = A & B
  - 真值表
    | A | B | L |
    | - | - | - |
    | 0 | 0 | 0 |
    | 0 | 1 | 0 |
    | 1 | 0 | 0 |
    | 1 | 1 | 1 |
- 或门
  - 逻辑表达式：L = A | B
  - 真值表
    | A | B | L |
    | - | - | - |
    | 0 | 0 | 0 |
    | 0 | 1 | 1 |
    | 1 | 0 | 1 |
    | 1 | 1 | 1 |
- 与非门
  - 逻辑表达式：L = !A & B
- 或非门
  - 逻辑表达式：L = !A | B
  - 真值表
    | A | B | L |
    | - | - | - |
    | 0 | 0 | 1 |
    | 0 | 1 | 0 |
    | 1 | 0 | 0 |
    | 1 | 1 | 0 |
- 异或门
  - 逻辑表达式：L = A ^ B
  - 真值表
    | A | B | L |
    | - | - | - |
    | 0 | 0 | 0 |
    | 0 | 1 | 1 |
    | 1 | 0 | 1 |
    | 1 | 1 | 0 |
- 异或非门
  - 逻辑表达式：L = !A ^ B
- 与或非门
  - 逻辑表达式：L = !(A & B)
- 或非门
- 与非门
  - 逻辑表达式：L = !A & B
  - 真值表
    | A | B | L |
    | - | - | - |
    | 0 | 0 | 1 |
    | 0 | 1 | 1 |
    | 1 | 0 | 1 |
    | 1 | 1 | 0 |
- 异或非门
  - 逻辑表达式：L = !A ^ B
- NAND门
  - 逻辑表达式：L = !(A | B)
- 或非门
  - 逻辑表达式：L = !A | B
- 异或非门
  - 逻辑表达式：L = !A ^ B
- NAND门
  - 逻辑表达式：L = !(A & B)

>真值表：选中自建模块后，确保有输入输出单元，然后点击菜单栏电路-真值表，即可查看该模块的真值表

### 操作方法
- 新建模块
  点击菜单栏电路-新建逻辑电路，即可新建模块
- 修改自建模块名称
  选中模块，点击菜单栏电路-逻辑电路，即可修改名字，符号，类别
- 交叉线的画法
  按住Alt，点击交叉点
- 旋转元器件
  ctrl+L

# 基本逻辑电路
## 半加器
单位二进制加法：0 + 0 = 0(无进位)、0 + 1 = 1(无进位)、1 + 0 = 1(无进位)、1 + 1 = 0(进位1)
半加器：实现单位二进制加法，不涉及进位
- 逻辑表达式：S = A ^ B、C = A & B
- 真值表
  | A | B | S | C |
  | - | - | - | - |
  | 0 | 0 | 0 | 0 |
  | 0 | 1 | 1 | 0 |
  | 1 | 0 | 1 | 0 |
  | 1 | 1 | 0 | 1 |
实现这样一个运算的逻辑电路称为半加器
> 半加器电路是指对两个输入数据位相加，输出一个结果位和进位，没有进位输入的加法器电路。 是实现两个一位二进制数的加法运算电路。

根据经验我们得知，逻辑表达式：
$$\ S = A \oplus B \\ C = A \& B$$

打开logiccircuit根据上面异或和与关系搭建门电路，验证

- 将异或(XOR)门用与、或、非门实现出来
- 然后用异或(XOR)门和与门共同搭建半加器(Half Adder)
- 搭建完之后通过按键和LED测试，符合真值表逻辑即可

## 全加器
全加器：实现二进制加法，涉及进位
- 逻辑表达式：S = A ^ B ^ C、C = (A & B) | (B & C) | (A & C)
- 真值表
  | A | B | C | S | C |
  | - | - | - | - | - |
  | 0 | 0 | 0 | 0 | 0 |
  | 0 | 0 | 1 | 1 | 0 |
  | 0 | 1 | 0 | 1 | 0 |
  | 0 | 1 | 1 | 0 | 1 |
  | 1 | 0 | 0 | 1 | 0 |
  | 1 | 0 | 1 | 0 | 1 |
  | 1 | 1 | 0 | 0 | 1 |
  | 1 | 1 | 1 | 1 | 1 |
实现这样一个运算的逻辑电路称为全加器
根据经验我们得知，逻辑表达式：
$$\ S = A \oplus B \oplus C \\ C = (A \& B) | (B \& C) | (A \& C)$$

打开logiccircuit根据上面异或和与关系搭建门电路，验证

- 将异或(XOR)门用与、或、非门实现出来
- 然后用异或(XOR)门和与门共同搭建半加器(Half Adder)

- 然后用半加器(Half Adder)和与门共同搭建全加器(Full Adder)
- 搭建完之后通过按键和LED测试，符合真值表逻辑即可

利用只读存储器，通过真值表
（ROM）实现全加器
- 将全加器电路用ROM实现出来
- 搭建完之后通过按键和LED测试，符合真值表逻辑即可
- ROM中的数据写入为全加器的真值表

## 8位加法器 8Adder
- 8位加法器：实现8位二进制加法，涉及进位
> 补充：这种全加器叫串形加法器，由于高位的运算需要等待低位的进位输出，所以会有延迟，效率不是很高。
还有一种并行加法器可以实现同步，这里不多做介绍。事实上还可以通过前面的ROM实现方法实现低延迟运算。

**制作测试器测试**
也就是八位按钮和八位的LED，通过测试器测试，符合预期即可
