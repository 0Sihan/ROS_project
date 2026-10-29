# ROS2无人系统课程 Lab01 & Lab02 学习记录
> 环境：Ubuntu 24.04 + ROS 2 Jazzy
> 学习目标：搭建ROS2开发环境，掌握Gazebo仿真基础，理解ROS2三大通信机制（话题/服务/动作）

## 目录
1. [Lab01 环境搭建与仿真验证](#lab01-环境搭建与仿真验证)
2. [Lab02 ROS2通信基础：话题、服务、动作](#lab02-ros2通信基础话题服务动作)
3. [知识点总结](#知识点总结)
4. [遇到的问题与解决方案](#遇到的问题与解决方案)
5. [实验成果](#实验成果)

---

## Lab01 环境搭建与仿真验证
### 实验目的
1. 验证ROS2 Jazzy安装有效性，配置课程工作空间
2. 使用`colcon`编译ROS2功能包，熟悉工作空间环境刷新
3. 启动Gazebo仿真环境，加载TurtleBot3 Waffle Pi机器人模型，验证仿真与机器人基础话题
4. 理解无人系统分层架构：感知、决策、规划、控制四层

### 实验步骤简述
1. 验证ROS2发行版：`printenv ROS_DISTRO`，确认输出`jazzy`
2. 进入课程工作目录，执行`colcon build --symlink-install`编译课程功能包`course_ugv`
3. 加载工作空间环境脚本`source install/setup.bash`
4. 设置环境变量`TURTLEBOT3_MODEL=waffle_pi`，通过launch文件一键启动Gazebo仿真与RViz可视化
5. 使用`ros2 topic list`查看机器人发布的底层话题（`/cmd_vel`、`/odom`、`/scan`、`/tf`等），确认机器人传感器与控制接口正常。

### 实验结果
成功启动Gazebo仿真环境，场景内加载TurtleBot3机器人，环境内包含障碍物圆柱与围墙，仿真正常运行无崩溃。
![Lab01 Gazebo仿真界面](./images/lab01_gazebo.png)

> 关键收获：
> - ROS2每次新开终端都需要执行`source`加载环境；可以写入`.bashrc`实现自动加载
> - `colcon build`是ROS2编译工具，`--symlink-install`软链接模式方便代码修改，无需重复编译Python脚本
> - 无人巡检系统四层架构对应：
>   - 感知层：激光雷达`/scan`、相机图像`/camera/image_raw`
>   - 决策层：任务调度、状态机
>   - 规划层：全局路径、局部避障路径规划
>   - 控制层：速度指令`/cmd_vel`下发，控制机器人运动

---

## Lab02 ROS2通信基础：话题、服务、动作
### 实验目的
1. 理解ROS2计算图模型，掌握三种核心通信范式：**话题(Topic)、服务(Service)、动作(Action)**，并区分适用场景
2. 自主编写话题发布/订阅节点，实现自定义消息发布与监听
3. 掌握服务的客户端调用，实现订阅消息后异步调用加法服务的链路
4. 实现Action动作服务端，验证长时任务的**过程反馈+最终结果**特性
5. 熟悉ROS2命令行调试工具：`ros2 topic`、`ros2 service`、`ros2 action`、`ros2 node`

### 实验步骤简述
1. 体验官方示例`talker/listener`，理解话题发布订阅模型
2. 编写Python发布节点`hello_ros.py`，定义`/greet`话题，周期性发布字符串消息；使用`ros2 topic echo /greet`监听话题消息
3. 扩展节点：增加订阅器，接收`/greet`消息，统计订阅计数，异步调用`/add_two_ints`加法服务，打印计数与求和结果
4. 实现Fibonacci动作服务端，支持发送目标任务，在任务执行过程中持续推送过程反馈，任务结束返回最终结果；通过`ros2 action send_goal`从命令行发送目标
5. 选做：使用launch文件一键启动多个节点，实现多节点编排

### 实验结果
1. 话题通信：`/greet`话题正常周期性发布字符串消息，终端监听可连续捕获消息
![Lab02 topic echo /greet](./images/lab02_topic_echo_greet.png)

2. 服务通信：成功调用`/add_two_ints`加法服务，输入a=3,b=4，返回sum=7；扩展节点实现订阅消息后自动调用服务，打印订阅计数与求和结果
![Lab02 service call结果](./images/lab02_service_call.png)
![Lab02 hello_publisher节点日志](./images/lab02_publisher_log.png)

3. Action动作通信：Fibonacci动作服务正常接收目标，执行过程持续输出中间序列反馈，任务完成返回完整斐波那契数列
![Lab02 action反馈输出](./images/lab02_action_feedback.png)

> 三种通信机制对比（巡检系统场景举例）
> |通信方式|通信模型|特点|巡检系统场景|
> | ---- | ---- | ---- | ---- |
> |Topic话题|发布-订阅，单向流式|持续周期性推送数据，无应答，一对多|激光雷达扫描数据、相机图像、机器人里程`/odom`|
> |Service服务|客户端-服务端，同步请求应答|短任务，一问一答，等待返回结果|查询地图上某个航点坐标、查询设备状态|
> |Action动作|目标-反馈-结果，异步长任务|任务耗时久，执行中持续回传进度反馈，支持取消任务|机器人自主导航前往巡检点、长时路径规划任务|

---

## 知识点总结
1. **ROS2环境管理**
    - ROS2环境由两层组成：系统ROS安装包环境（`/opt/ros/jazzy`） + 本地工作空间环境；每次新开终端都需要`source`加载环境
    - `colcon build`用于编译功能包，`--symlink-install`适合Python开发，修改源码不需要重复编译
    - Launch文件可以一键批量启动多个节点，统一配置参数，简化多节点系统启动。
2. **ROS2通信模型**
    - Topic：流式、持续数据，适合传感器、状态流，异步单向；
    - Service：短请求、即时应答，适合查询类短任务；
    - Action：长时任务，带实时进度反馈，支持任务中断，适合导航、规划等耗时任务。
3. **仿真基础**
    - Gazebo作为物理仿真器，可以加载机器人模型、搭建障碍物环境，输出传感器仿真数据；
    - RViz用于可视化机器人状态、传感器数据、TF坐标变换。

## 遇到的问题与解决方案
1. **`ros2: command not found`**
    - 原因：未加载ROS2 Jazzy环境
    - 解决：执行`source /opt/ros/jazzy/setup.bash`，也可以写入`~/.bashrc`自动加载
2. **新开终端，Gazebo/ROS命令找不到工作空间包**
    - 原因：没有执行`source install/setup.bash`加载本地工作空间
    - 解决：每次新开终端进入工作目录后执行环境source脚本
3. **节点运行时提示服务暂时不可用（WARN）**
    - 原因：发布订阅节点启动早于服务端，客户端发起调用时服务还未就绪
    - 解决：代码中增加服务可用性判断，等待服务就绪再发送请求，跳过服务未就绪时刻的调用
4. **WSL图形界面Gazebo黑屏**
    - 解决：配置`DISPLAY`环境变量，启用WSLg，设置`export DISPLAY=:0`

## 实验成果
1. ✅ Lab01：成功搭建Ubuntu24.04 + ROS2 Jazzy环境，编译课程功能包，Gazebo仿真TurtleBot3机器人正常启动，机器人传感器话题全部可用。
2. ✅ Lab02：
    - 自定义话题发布订阅节点`hello_ros.py`，实现消息发布、监听；
    - 节点扩展：订阅消息计数 + 异步调用加法服务；
    - Action动作服务端，实现带实时反馈的斐波那契长任务；
    - 掌握全套ROS2命令行调试工具，能观察节点、话题、服务、动作状态。
