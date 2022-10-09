# ROS仿真

## gazebo一直在“Preparing your world”

意思是gazebo正在下载模型文件，耐心等待下完即可



## gazebo加载不出机器人的xacro模型时的解决方法

如果xacro安装正确的话，运行

> roslaunch mbot_gazebo view_mbot_gazebo_empty_world.launch

会报错

> [ERROR] [1627529772.148763844]: No link elements found in urdf file

首先将使用mbot_gazebo_empty_world.xacro的文件中使用另一个xacro文件的

```xml
<mrobot_body/>
改为
<xacro:mrobot_body/>
```

然后将所使用的xacro文件内自定义的macro的引用都加上xacro:

```xml
<wheel prefix="left"  reflect="-1"/> 
改成
<xacro:wheel prefix="left"  reflect="-1"/>
```

因为xacro文件的语法改了

<mark>所有自定义的macro，要引用的话都要加上xacro:</mark>

文件中mrobot_body也是一个自定义的macro被mbot_gazebo_empty_world.xacro所引用，因此也要加上xacro:
