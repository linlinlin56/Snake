# 贪吃蛇游戏设计与实现实验报告

## 摘要  
简述贪吃蛇游戏的设计背景、核心功能及实现技术，说明项目采用MASM 32汇编语言开发，通过Win32 API实现图形界面与交互逻辑，完成了贪吃蛇运动控制、碰撞检测、多界面切换等核心功能，最终实现一个具有不同难度模式的交互式贪吃蛇游戏。  

## Abstract  
A brief introduction to the design background, core functions and implementation technologies of the Greedy Snake game. This project is developed using MASM 32 assembly language, realizes graphical interface and interactive logic through Win32 API, completes core functions such as snake movement control, collision detection, and multi-interface switching, and finally realizes a Greedy Snake game with different difficulty modes.  


## 第1章 背景与需求说明  
### 1.1 设计背景  
- 汇编语言在底层图形交互与硬件控制中的优势  
- 贪吃蛇游戏作为经典小游戏的代表性与实现价值  
- 基于Win32 API开发图形化应用的技术意义  

### 1.2 软件需求（游戏介绍）  
#### 1.2.1 基本运行  
- 运行环境：基于MASM 32 SDK的Windows平台  
- 核心功能：贪吃蛇移动、食物获取、碰撞判定、游戏状态管理  

#### 1.2.2 游戏基本规则  
- 贪吃蛇通过方向键控制移动，吃到食物后长度增加  
- 碰到边界、自身或障碍物时游戏结束  
- 得分规则：吃到食物累加分数，不同难度模式分数系数不同  

#### 1.2.3 游戏模式设计  
- 入门、简单、进阶、困难四种模式，通过计时器刷新间隔控制速度  
- 进阶模式随蛇长度增加动态提升速度  

#### 1.2.4 界面交互需求  
- 支持多界面切换（开始界面、模式选择界面、游戏界面、帮助界面）  
- 实时显示分数、当前模式、速度等信息  


## 第2章 总体功能框架设计  
- 核心模块划分：游戏逻辑模块、界面交互模块、模式控制模块、环境配置模块  
- 模块间关系：界面交互模块接收用户输入并传递至游戏逻辑模块，模式控制模块调节游戏速度，环境配置模块提供基础运行环境  
- 整体流程：用户交互→输入处理→逻辑更新（移动、碰撞检测）→界面刷新→状态反馈  


## 第3章 详细方案设计  
### 3.1 实验环境  
- 开发工具：MASM 32 SDK（ml汇编器、link链接器）  
- 运行环境：Windows操作系统（兼容32位应用）  
- 环境配置：include路径与lib路径设置，汇编与链接命令说明  

### 3.2 核心数据结构  
- 贪吃蛇位置存储：`dwX`/`dwY`数组（记录蛇身各节坐标）  
- 方向控制：`dwDirection`变量（0-停止，1-上，2-下，3-左，4-右）  
- 障碍物与食物：`barrierX`/`barrierY`数组（障碍物坐标）、`dwNextX`/`dwNextY`（食物坐标）  
- 模式与状态：`dwModuleflag`（模式标识）、计时器刷新间隔变量  

### 3.3 游戏逻辑模块  
#### 3.3.1 方向控制逻辑  
- 基于`dwDirection`计算下一步位置（`dwXTemp`/`dwYTemp`）  
- 方向反转限制（如当前向上时不可直接向下）  

#### 3.3.2 碰撞检测逻辑  
- 边界碰撞：判断`dwXTemp`/`dwYTemp`是否超出游戏地图范围  
- 自身碰撞：对比`dwXTemp`/`dwYTemp`与`dwX`/`dwY`数组中已有坐标  
- 障碍物碰撞：对比`dwXTemp`/`dwYTemp`与`barrierX`/`barrierY`数组  

#### 3.3.3 运动更新逻辑  
- 普通移动：数组元素移位，更新蛇头位置  
- 食物获取：蛇头坐标与食物坐标重合时，蛇长度增加并重新生成果物（避免与蛇身/障碍物重叠）  

### 3.4 界面交互模块  
#### 3.4.1 多界面设计  
- 界面切换原理：通过隐藏/显示控件实现（开始界面、模式选择界面等）  
- 核心控件：按钮（开始、暂停、模式选择）、文本框（分数、模式显示）  

#### 3.4.2 绘制功能  
- 蛇身绘制：基于`dwX`/`dwY`数组绘制蓝色方块  
- 食物与障碍物绘制：橙色方块（食物）、固定颜色方块（障碍物）  

### 3.5 模式控制模块  
- 速度调节机制：通过计时器刷新间隔控制（入门500ms、简单300ms、困难100ms，进阶动态调整）  
- 分数计算：不同模式下食物得分系数差异  


## 第4章 关键算法与代码实现  
### 4.1 游戏逻辑核心算法  
#### 4.1.1 方向与位置更新  
```assembly
; 示例：根据方向计算下一步位置（伪代码）
updatePosition proc
    mov eax, dwDirection
    .if eax == 1 ; 向上
        mov eax, dwY[0]
        sub eax, step ; 步长
        mov dwYTemp, eax
        mov dwXTemp, dwX[0]
    .elseif eax == 2 ; 向下
        ; 类似逻辑
    .endif
    ret
updatePosition endp
```

#### 4.1.2 碰撞检测实现  
```assembly
; 示例：边界碰撞检测（伪代码）
checkBoundary proc
    mov eax, dwXTemp
    .if eax < 0 || eax >= mapWidth || dwYTemp < 0 || dwYTemp >= mapHeight
        call gameOver ; 触发游戏结束
    .endif
    ret
checkBoundary endp
```

### 4.2 界面交互实现  
#### 4.2.1 多界面切换  
通过`ShowWindow` API控制控件显示/隐藏，示例：  
```assembly
; 从开始界面切换到游戏界面（伪代码）
switchToGame proc
    ; 隐藏开始界面控件
    invoke ShowWindow, hStartBtn, SW_HIDE
    ; 显示游戏界面控件
    invoke ShowWindow, hGameMap, SW_SHOW
    ret
switchToGame endp
```

#### 4.2.2 图形绘制  
基于GDI函数绘制蛇身与食物：  
```assembly
; 绘制蛇头（伪代码）
drawSnakeHead proc
    invoke CreateSolidBrush, RGB(0,0,255) ; 蓝色
    mov hBrush, eax
    ; 绘制方块（基于dwX[0], dwY[0]）
    invoke Rectangle, hdc, dwX[0], dwY[0], dwX[0]+step, dwY[0]+step
    ret
drawSnakeHead endp
```

### 4.3 模式与速度控制  
```assembly
; 进阶模式速度调整（伪代码）
adjustSpeed proc
    .if dwModuleflag == 2 ; 进阶模式
        mov eax, snakeLength
        mov ebx, 300
        sub ebx, eax*10
        .if ebx < 100
            mov ebx, 100
        .endif
        mov timerInterval, ebx
        invoke SetTimer, hWnd, 1, timerInterval, NULL
    .endif
    ret
adjustSpeed endp
```


## 第5章 系统测试与结果分析  
### 5.1 功能测试  
- 基本操作：方向控制响应、食物获取与长度增加  
- 碰撞判定：边界、自身、障碍物碰撞触发游戏结束的准确性  
- 模式切换：不同模式下速度与分数计算的正确性  

### 5.2 界面测试  
- 多界面切换流畅性（无控件残留、布局正常）  
- 信息显示：分数、模式、速度实时更新的准确性  

### 5.3 测试结果分析  
- 核心功能均实现，无逻辑漏洞  
- 界面交互响应及时，模式难度梯度合理  


## 第6章 人员分工  
- 成员1：环境搭建与基础框架实现（汇编环境配置、窗口创建）  
- 成员2：游戏逻辑开发（方向控制、碰撞检测、运动更新）  
- 成员3：界面交互设计（多界面切换、控件绘制与事件响应）  
- 成员4：模式控制与优化（速度调节、分数计算、难度平衡）  


## 结论  
总结项目实现的核心价值，分析汇编语言在图形化游戏开发中的优势与局限，提出可扩展方向（如增加音效、自定义地图等）。  
