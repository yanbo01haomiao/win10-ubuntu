## 	[保姆级教程]win10下ubuntu双系统安装和环境配置(基础环境与ROS环境)

因为疫情的原因没办法回学校去，而且实验室已经关闭，没有办法跑代码，我手边只有一个能上网课的surface，所以紧急请老师帮我把之前的老电脑邮寄回来勉强使用，电脑是联想G40,大学时候的工具人，现在使用起来还是没什么问题的，所以就当成临时替代品了。

目的：win10下ubuntu双系统安装和环境配置(基础环境与ROS环境)

梅开二度（不是）。下面就是在实际操作过程中的步骤和遇到的问题的流水帐啦，记一下存档。

说在前面：如果有小伙伴需要学习linux（比如操作系统）的话，可以用双系统的选择，比VM安装更好。而且如果有独立显卡的话强烈推荐双系统，因为VM模拟下跑深度学习代码没办法使用GPU加速，硬件被虚拟了找不到对应的source。

### 双系统安装

#### 准备

1. 需要安装的系统的镜像文件(iso)。我这里因为ROS平台的原因强制选择ubuntu16.04的版本。官网下载或者百度云盘下载（因为我有超级会员就后者比较快）。

2. 大小大于或等于4G的U盘，因为需要格式化，所以不是空U盘的话需要先备份文件啦`[勘误@vgg]`这里软碟通可以快速写入——便捷写入，不需要备份。启动盘的关键在于引导和基本文件，格式化只是匹配格式。

3. UltraISO(软碟通)软件，这里给一个注册版本的下载链接：https://m.ddooo.com/softdown/153565.htm 。这个软件是用来写入镜像文件的，也就是将镜像写成安装盘的意思，这样就可以用U盘进行安装了。

#### 制作系统盘

1. 安装并打开UltraISO，插上 U 盘；

2. 进入UltraISO，选择文件，浏览到ubuntu镜像所在的目录，双击打开ubuntu镜像文件；

3. 在界面菜单栏选择"启动"，选择"写入硬盘映像"；
4. 默认设置即可，不需要额外调整，写入方式默认是USB-HDD+。

#### 准备空白分区

这步就是为linux系统腾出空间，这里只给出建议是50G或以上，当然是越大越好，这个根据你的实际情况决定，最少50G是为了保证系统的正常使用。

> 下面都是在windows10环境下的操作

1. "此电脑"点击右键，点击"管理"，点击存储选项下的"磁盘管理"，或者直接左下角搜索"磁盘管理"；

2. 为linux分配空间：这里细心的为大家区分两种情况，大家看自己的情况选择：

   2.1  单硬盘（不管是SSD还是机械）：

   注意：这里指的是你的硬盘数量而不是盘符数量。在磁盘管理的页面能比较清晰看到自己的存储情况。磁盘0，磁盘1这些就是硬盘数量，后面的一系列反映盘符分配情况的信息。

   选择最后一个盘（比如 CD两个盘的最后一个是D盘，CDE盘的最后一个是E盘，CDEF盘的最后一个是F盘，以此类推），在该盘点击右键，下拉选项中选择“压缩卷”，输入压缩空间量，单位为M（1G=1024M），这里我选择输入的是51200M（50×1024M），然后压缩成未分配空间就可以了。

   注：如果你的最后一个盘容量太小，还不够分出50G，那需要从其他盘匀一些过来，需要用到DiskGenius这个工具进行盘符分割等操作，具体操作不再赘述，小伙伴们请自行解决。

   2.2  多硬盘（SSD+）：这里我假设是1×SSD+1×机械的默认配置，这里需要进行部分和2.1单硬盘唯一的不同就是需要先在SSD所在的C盘（按理来说就是电脑的第一块硬盘，部分电脑系统装得比较奇怪那就不是我们本次讨论的问题范围啦）分出200M的空白分区用来安装ubuntu的EFI启动项，然后再选择另一块机械硬盘的最后一个盘符，在该盘点击右键，选择压缩卷，输入压缩空间量压缩，步骤与2.1完全一致。

   注：

   （1）这里需要解释一下2.2的这个操作有什么意义。双硬盘在C盘分出了200M用来安装EFI启动项。电脑在开机的时候，会自动在C盘所在的那个硬盘搜索启动项以启动系统，开机时会搜索到windows和ubuntu两个启动项，可以手动选择进入哪个系统。当然这里的"启动项需要安装在C盘所在的硬盘"并不是绝对的，只是为了开机能够自动搜索到启动项，安装在其他硬盘也可以，只是每次开机都需要进boot manager才能找到ubuntu启动项，比较麻烦。一般现在对于多EFI启动项的电脑都会给一个选择页面，如果不是自动的话需要进入boot manager中选择。

   （2）为什么要选择最后一个盘符进行压缩卷：windows和ubuntu的文件存储格式是不一样的，分区的操作只是将磁盘分了一部分在逻辑地址上给了ubuntu，事实上两个系统还是在共用一块物理磁盘，为了防止存储格式不同两个系统可能相互影响（不确定是否会，但是我这里就这样假设了），通过从最后一个盘压缩将ubuntu的分区分到了磁盘最后一段，也就是一块磁盘的前部分是windows的分区，后部分是ubuntu的分区（因为磁盘的填充和擦除是线性逻辑的）。

   #### 安装Ubuntu系统

   注：因为各个厂商的计算机bios和boot manager启动的快捷键不相同，所以请自行百度如何进bios和boot manager。联想G40是默认关闭按键快捷进入bios的，需要按在充电接口旁边的按钮自动进入bios选项。

   1. windows设置预处理：在控制面板-硬件与声音-电源选项中："选择电源按钮的功能"，在定义电源按钮并启用密码保护中进入"更改当前不可用的设置"，取消选择"启用快速启动(推荐)"。
   
   2. 插入系统盘，重启电脑，开机进bios，在Security页面，关掉secure boot（不同电脑secure boot可能在不同位置），然后到Boot页面，如果有Fast Boot这一项（联想电脑有），也把它关掉，没有忽略；然后保存更改，在Boot页面下方启动项选择 USB启动，回车，如果顺利进入安装页面，继续往下做；如果点击USB启动项无法进入，保存并退出，电脑会重启，根据自己电脑按相应的键进boot manager，找到USB启动项，回车即可进入。 
      注：这里我遇到了一个极为恶心的问题，卡了我久，就是无论如何我都没有办法在bios中找到USB的启动项，选项直接消失，只有一个window的启动项。解决方案：在windows下，设置，进入更新与安全，选择恢复，在恢复页面“高级启动”中选择“立即重新启动”，疑难解答，高级选项，启动设置，执行重启。这样理论上是可以找到U盘启动项的，这个地方我忘记截图存档了，但是应该是可以找到一个从U盘中启动的选项，这样直接就会转到U盘启动，进入ubuntu的安装了。
      
   3. 进入安装步骤。
      安装过程不详细说明，但是在这一步骤时注意：选择安装类型：`清除整个磁盘并按照`；`其他选项`。这里选择`其他选项`，从而进行手动的分区。
      找到我们之前的windows下准备的50G的空白分区，选择，在下面选择+号或者新建分区表分别对每一项进行分区设置，设置细节如下：
      `efi`：引导分区，分区类型为逻辑分区，大小200M，分区格式ext4;
      `swap`：充当虚拟内存，分区类型为逻辑分区，大小内存大小的1-2倍，分区格式swap;
      `挂载点/`：安装系统和软件的主分区，分区类型为主分区，大小20G（可根据实际情况增加），分区格式ext4;
      `挂载点/home`：其他分区，相当于windows的其他盘符，分区类型为逻辑分区，大小为剩下的空间，分区格式ext4。
   
      如果设置错误，可以点击change进行修改。最后在这个页面的下面`Device for boot loader installation`，下拉选择设置的`efi`所对应的盘符编号，可能是`sdb2,3`等，根据自己的实际编号选择即可。
   
   4. 确认无误后进行下一步安装。然后就是漫长的等待环节了，你可以去玩玩手游，看看书等它装好再回来就OK啦。
   
   5. 全部安装完成提示重启。拔出U盘，重启电脑。在启动选项中选择`Ubuntu`就可以进入新安装的ubuntu系统啦。原本的windows系统只要选择`Windows Boot Manager`就可以进入windows啦。
   
   至此，双系统的安装过程全部结束。恭喜。

### Ubuntu环境配置

#### 基础环境配置

1. 配置国内镜像源：
   
   备份默认的更新源文件，操作前养成备份的好习惯:

```bash
sudo cp /etc/apt/sources.list /etc/apt/sources.list.bk
```

把国内镜像指令写到 `source.list` 中：一般情况下，将 `/etc/apt/sources.list` 文件中 Ubuntu 默认的源地址 `http://archive.ubuntu.com/` 替换为 `http://mirrors.ustc.edu.cn` （这里以科大源为例子，也可以是其他比如清华，阿里等）即可。

可以使用如下命令：

```bash
sudo sed -i 's/archive.ubuntu.com/mirrors.ustc.edu.cn/g' /etc/apt/sources.list
```

选择速度较快的更新源更改完 `sources.list` 文件后请运行 `sudo apt-get update` 更新索引以生效：

```bash
sudo apt-get update
```

执行更新

```bash
sudo apt-get upgrade
```
2. 安装必要工作软件：google浏览器，typora编辑器，其他使用的IDE或软件

#### ROS环境配置

```bash
sudo apt-get install ros-kinetic-desktop-full
```

安装ROS出现问题：

> 下列软件包有未满足的依赖关系：
>  ros-kinetic-desktop-full : 依赖: ros-kinetic-desktop 但是它将不会被安装
>                             依赖: ros-kinetic-perception 但是它将不会被安装
>                             依赖: ros-kinetic-simulators 但是它将不会被安装
>                             依赖: ros-kinetic-urdf-tutorial 但是它将不会被安装

>  apt-get install ros-kinetic-desktop ros-kinetic-perception ros-kinetic-simulators ros-kinetic-urdf-tutorial

这个命令看似能解决问题，但是又会陷入无限依赖的循环之中，是不能实际解决问题的，参考一下网上的解答，需要进行安装前准备：参考[链接](https://blog.csdn.net/u014430021/article/details/89036122)

1.终端输入命令：下载aptitude

```bash
sudo apt install aptitude
```

再输入命令：

```bash
sudo  aptitude install ros-kinetic-desktop-full 
```

2.在“软件和更新“里面，选中“更新”，将“从下列地点安装更新：”下面的选项都勾选中

3.再次安装：

```bash
sudo apt-get install ros-kinetic-desktop-full
```

这样就可以安装成功。其中第一步耗时很长，需要保证网络连接和电脑开启。

初始化ROS

```bash
sudo rosdep init
```

如果出现

```bash
sudo rosdep init
ERROR: cannot download default sources list from:
https://raw.githubusercontent.com/ros/rosdistro/master/rosdep/sources.list.d/20-default.list
Website may be down.
```

尝试使用：
*打开hosts文件* 

```bash
sudo gedit /etc/hosts 
```

*在文件末尾添加* 

```bash
151.101.84.133  raw.githubusercontent.com 
```

*保存后退出再尝试即可成功。*

```bash
	Wrote /etc/ros/rosdep/sources.list.d/20-default.list
	Recommended: please run
```

```bash
	rosdep update
```

加载环境设置文件

```bash
	echo "source /opt/ros/kinetic/setup.bash" >> ~/.bashrc
	source ~/.bashrc
```

安装rosinstall

```bash
	apt-get install python-rosinstall
```
测试

	roscore 

如果出现对应的说明和端口号就说明安装成功。

安装roboware——一款好用的IDE，在github上找到最新的版本，下载deb文件。[参考文章](https://blog.csdn.net/JIEJINQUANIL/article/details/102572722)



创建并初始化工作目录

```bash
mkdir -p ~/catkin_ws/src #建立工作空间
cd ~/catkin_ws #进入工作空间
catkin_make    #开始编译，生成build（cmake&catkin缓存和中间文件）  devel  src（源代码）
cd ~/catkin_ws #回到工作空间,catkin_make必须在工作空间下执行
catkin_make    #开始编译
source ~/catkin_ws/devel/setup.bash #刷新坏境
```

ROS环境变量配置

打开`~/.bashrc`

```bash
gedit ~/.bashrc
```

在文件末尾添加：

```bash
# Set ROS Kinetic
source /opt/ros/kinetic/setup.bash
source ~/catkin_ws/devel/setup.bash
# Set ROS Network
export ROS_HOSTNAME=192.168.0.103
export ROS_MASTER_URI=http://${ROS_HOSTNAME}:11311
# Set ROS alias command
alias cw='cd ~/catkin_ws'
alias cs='cd ~/catkin_ws/src'
alias cm='cd ~/catkin_ws && catkin_make'
```

这里的ip也可以写成localhost，在一台pc上运行所有的ROS功能包。

最后测试一下，有个直观的把握：遥控小乌龟

```bash
# terminal 1
zhangyuan@zhangyuan-Lenovo:~$ roscore
# terminal 2 
zhangyuan@zhangyuan-Lenovo:~$ rosrun turtlesim turtlesim_node
[ INFO] [1589460797.225174968]: Starting turtlesim with node name /turtlesim
[ INFO] [1589460797.232611693]: Spawning turtle [turtle1] at x=[5.544445], y=[5.544445], theta=[0.000000]
# terminal 3 键盘遥控
zhangyuan@zhangyuan-Lenovo:~$ rosrun turtlesim turtle_teleop_key
Reading from keyboard
# terminal 4 当前正在运行的节点信息图
zhangyuan@zhangyuan-Lenovo:~$ rqt_graph
---------------------------
Use arrow keys to move the turtle.
```
