# 设计

## 注意事项

* 由于python对基础数据操作（求补码等）比较麻烦，所以所有的负数都以8位无符号整型表示，如-1在本程序的表示为129

## 磁盘

* 文件组织结构
  * 使用文件分配表
  * 系统占用前三个表项
  * 表项内容为-1则表示该表项已被占用
  * 表项为0代表对应文件块未被分配
  * 表项为130表示对应文件块损坏
* 磁盘空间管理
  * 一个磁盘有128个物理块
  * 每个物理块大小为64个字节
  * 盘块号从0开始到127
  * 文件分配表有128项，每项占用一个字节
  * 文件分配表占用前两个表项
  * C盘占用第三块表项
  * 表项内容为3~127，则表项内容表示当前表项的下一项的地址
* 文件
  * 长度8Byte
  * 3Byte名字（实验中合法文件名仅可以使用字母、数字和除“$”、“.”和“/”以外的字符，第一个字节的值为$时表示该目录为空目录项，文件名和类型名之用“.”分隔，用“/”作为路径名中目录间分隔符）
  * 2Byte扩展名
  * 1Byte判断是文件夹还是文件
  * 1Byte起始地址
  * 1Byte文件长度
  * 文件块内前12个字节记录创建时间
* 文件夹
  * 长度8Byte
  * 3Byte目录名
  * 2Byte扩展名(空)
  * 1Byte表示这是文件夹
  * 1Byte起始盘块号
  * 1Byte未使用(0)
  * 第六个字节(0代表否，1代表是)
    * 0：只读文件
    * 1：系统文件
    * 2：普通文件
    * 3：目录属性
* 文件块
  * 一个根目录文件块可以存下8个文件或目录
  * 一个普通文件块可以存下8个文件或

## 命令设计

* 文件
  * create 文件名
  * copy 源文件名  目标文件名
  * delete文件名
  * move 源文件名  目标文件名
  * edit 文件名
  * type  文件名   仅仅是显示文件内容。
  * aa.ex 运行当前目录对应的文件
* 磁盘
  * format 盘符
* 文件
  * mkdir 创建文件夹
  * rmdir 删除文件夹
  * move 原路径 目标路径
  * dir 显示当前目录所有文件和文件夹
  * cd 移动当前目录
  * rdir 删除当前目录的空目录
  
## 主存管理

* 主存分配策略
  * 当有程序要存放入主存时，查看空闲块总数是否够用，如果够用，先分配一块用来存放页表，然后查位示图中为“0”的位，根据查到的位所在的字号和位号可计算出对应的块号，同时在该位填上占用标志“1”，并填写页表；不够用，分配失败。
  * 块号=字号*字长+位号
* 主存回收策略：
  * 字号=[块号/位示图中字长]
  * 位号=块号mod位示图中字长
  * 然后把这一位的“1”清成“0”，表示该块成为空闲块了
  * 最后回收页表所占用空间
* 屏幕显示
  * 主存使用情况示意图，哪些主存已经分配，哪些主存未分配，以不同的颜色表示（例如，红色表示已分配，蓝色表示未分配）。
  * 正在运行的进程对应指令存放的位置以特殊颜色显示。

## 设备管理

* 设备管理主要包括设备的分配和回收。
* 设备的模拟
  * 模拟系统中有A、B、C三种独占型设备，A设备3个，B设备2个，C设备1个。
* 数据结构
  * 因为模拟系统比较小，因此只要设备表设计合理既可。
* 设备分配
  * 采用先来先服务策略。
* 设备回收
  * 回收设备后，要注意唤醒等待设备的进程。
* 屏幕显示
  * 屏幕显示要求包括：每个设备是否被使用，哪个进程在使用该设备，哪些进程在等待使用该设备。

## 进程管理

* 进程管理主要包括进程调度，进程的创建和撤销、进程的阻塞和唤醒，中断作用的实现。
* 硬件工作的模拟
  * 中央处理器的模拟
    * 用CPU类模拟中央处理器。
    * 该函数主要负责解释“可执行文件”中的命令。
      * x=?;
      * x++;
      * x--;
      * !??;
      * end.
    * 注意：CPU只能解释指令寄存器IR中的指令。一个进程的运行时要根据进程执行的位置，将对应的指令存放到指令寄存器中。
  * 主要寄存器的模拟
    * EAX
    * 程序状态寄存器PSW
    * 指令寄存器IR
    * 程序计数器PC
    * 数据缓冲寄存器DR等。
  * 中断的模拟
    * 中断的发现应该是硬件的工作，这里在函数CPU中加检测PSW的方式来模拟
      * 在CPU函数中，每执行一条指令之前，先检查PSW，判断有无中断，若有进行中断处理，然后再运行解释指令。
      * CPU函数应该不断循环执行的。
  * 模拟中断的种类和中断处理方式
    * 程序结束（执行指令end形成的中断，软中断）：将结果写入文件out，其中包括文件路径名和x的值，调用进程撤销原语撤销进程，然后进行进程调度；
    * I/O中断（设备完成输入输出）：将输入输出完成的进程唤醒，将等待该设备的一个进程同时唤醒。
    * 时钟中断：进程时间片用完，转为就绪，重新进程调度
  * 时钟的模拟
    * 系统中的绝对时钟和相对时钟用全局变量模拟。系统时钟用来记录开机以后的时间
* 进程控制块
  * 进程标识符
  * 主要寄存器内容
  * 进程状态
  * 阻塞原因
  * 本模拟系统最多容纳10个进程块
  * PCB区域用数组模拟
* 进程控制块根据内容的不同组成不同的队列
  * 空白进程控制块链
  * 就绪队列
  * 阻塞队列
  * 正在运行的进程只有一个，系统初始时只有空白进程控制块链。
* 进程调度
  * 采用时间片轮转调度算法，采用时间片轮转调度算法时间片为5。
  * 进程调度函数的主要工作
    * 将正在运行的进程保存在该进程对应进程控制块中
    * 从就绪队列中选择一个进程
    * 将这个进程中进程控制块中记录的各寄存器内容恢复到CPU各个寄存器内。
* 进程控制
  * 进程创建create
    1. 申请空白进程控制块
    2. 申请主存空间，申请成功，装入主存
    3. 初始化进程控制块
    4. 将进程链入就绪队列，根据情况决定是否转向进程调度。
  * 进程撤销destory
    1. 回收进程所占内存资源
    2. 回收进程控制块
    3. 在屏幕上显示进程执行结果，进程撤销
  * 进程阻塞block
    1. 保存运行进程的CPU现场
    2. 修改进程状态
    3. 将进程链入对应的阻塞队列，然后转向进程调度
    4. 阻塞之后一定要调用进程调度函数
  * 进程的唤醒wake
    1. 将进程由阻塞队列中摘下
    2. 修改进程状态为就绪
    3. 链入就绪队列，根据情况决定是否转向进程调度
* 屏幕显示
  * 显示系统时钟
  * 显示正在运行的 进程的进程名、运行的指令、中间结果、相对时钟寄存器内容
  * 显示就绪队列中进程名
  * 显示阻塞队列中进程名
