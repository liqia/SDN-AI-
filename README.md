# SDN-AI-
ODL+OVS+Tensorflow AI



楼上服务器中的虚拟机(只有左三,左五服务器的虚拟机)安装了openssh用于远程登录.
在win10电脑的cmd里面输入下面相应的命令就能连接上相应的主机(**需要连上之江work后缀wifi或者连上之江有线，如果想要在寝室控制，可以看使用下面控制被穿透左三的命令**).苹果电脑可能要自己装个ssh,win10自带ssh.

|主机名字   |    命令| 密码|
|  ----  | ----  | ----  |
|s1     | ssh s1@10.1.118.183 | 1|
|s2   |  ssh s1@10.1.118.217  | 1|
|s3   |  ssh s1@10.1.118.13  | 1|
|s4   | ssh s1@10.1.119.103  | 1|
|s5    |  ssh s1@10.1.119.92  | 1|
|s6                |ssh ovs@10.1.119.173|  1|
|s7    | ssh s1@10.1.119.188  | 1|
|s8                |ssh s8@10.1.118.77| 1|
|s9                |ssh s8@10.1.118.26|  1|
|s10               |ssh s8@10.1.118.131|  1|
|s11               |ssh s8@10.1.118.180| 1|
|s12               |ssh ovs@10.1.118.66|  1|
|s13               |ssh ovs@10.1.119.120|  1|
|s14               |ssh ovs@10.1.118.16|  1|
|s15               |ssh ovs@10.1.119.104|  1|
|s16               |ssh ovs@10.1.119.126|  1|
|s17               |ssh s8@10.1.118.9|  1|
|右 1    | ssh zju@10.1.118.224  | 1|
|右 2   | ssh zju@10.1.118.109  | 1 |
| 左3   | ssh zju@10.1.118.216 |  1|
|左5    | ssh zjusec-003@10.1.119.51  | zjusec|
|左3（穿透） | ssh -oPort=6666 zju@150.158.157.122 | 1 |


​    

为了方便在寝室连上上面的这些主机, 我通过有公网ip的服务器搭建了内网ssh穿透到左3,可以通过如下命令连接到左3:   
`ssh -oPort=6666 zju@150.158.157.122` 密码为  1   
也就是可以在寝室通过这个命令控制左3, 至于想控制其他服务器可以通过套娃来完成(通过在左3输入上面表格的命令进而控制其他主机)

**具体穿透方法再NAT_traverse文件夹下有说明**。