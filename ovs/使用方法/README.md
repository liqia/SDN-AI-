# 如何安装OVS

在openvswitch-2.11.1目录下执行如下命令进行安装

```
sudo ./install.sh
```

执行命令过程中机器可能会重启. **如果机器重启了, 那么请再次执行一次上面的命令**

上述命令安装完成后如果存在文件`/var/log/gary.log`.

并且`dmeg`命令输出含有类似如下的内容:

```
[  604.878727] openvswitch: Open vSwitch switching datapath upcall delay mingliang!!! 8 8 16 2.11.1
[  604.878828] openvswitch: LISP tunneling driver
[  604.878829] openvswitch: STT tunneling driver
```

那么说明已经安装成功.

如果没有显示上面的信息，运行一下`start.sh`脚本。

# 如何启动OVS并开始数据

实际上,执行安装命令之后OVS已经启动并开始收集数据了,

但是如果安装成功之后,你的机器关过了,那么在openvswitch-2.11.1目录下执行如下命令启动ovs

```
sudo ./start.sh
```

该命令会删除和重新生成用户态的数据文件gary.log.

# 如何关闭OVS

执行如下命令关闭ovs:

```
sudo ./stop.sh
```

数据收集完成后,建议关闭ovs.

# 数据输出到何处

首先，因为OVS包括内核态和用户态，因此特征数据有些是在内核态下，另一些是在用户态下。
这导致了内核态特征数据和用户态特征数据是使用不同方法来取。

内核态的数据会输出到/var/log/kern.log中。

用户态的数据会输出到/var/log/gary.log中.

# 数据格式以及数据处理

获得实验数据kern.log和gary.log分别保存着内核态数据和用户态数据，

使用当前文件夹下的data_process.py对数据进行处理，生成文件
 ```datapath_Gary,datapath_upcall,user_table_time,user_upcall和userspace```

具体数据含义入下：

  ```
datapath_Gary：
    每个数据包经过都会产生一行。每行里面分别存放了：
    时间戳、 源IP、目的IP、源端口、目的端口、
    命中内核态流表的数据包数目(累加统计)、命中内核流表失败次数（累加统计）、
    查找内核态流表使用的毫秒数、
    ovs_flow_cmd_set()执行次数(累加统计)、ovs_flow_cmd_get()执行次数(累加统计)、ovs_flow_cmd_del()执行次数(累加统计)、ovs_execute_actions()执行次数(累加统计)、
    命中mask_catch数目(累加统计)、查询mask cache的次数（累加统计）、
    命中mask&flow_key hash表的次数（累加统计）、查询mask&flow_key hash表的次数（累加统计）、
    ovs_packet_cmd_execute（）执行失败次数（累加统计）.

datapath_upcall：
    内核每产生一个upcall都会产生一条。里面分别存放了：
    时间戳、upcall数量（累加统计）、当前upcall长度（每个包的长度）

user_upcall：
    用户态每收到一个upcall都会产生一行。每行里面存放了：
    时间戳、upcall时延

userspace ：
    定时每秒输出一行，每行分别为：
    时间戳、接收到控制器数据包数（累加统计）、发送给控制器的数据包数（累加统计）、
    udpif_upcall_handler执行次数（累加统计）、
    命中用户态流表次数（累加统计）、用户态流表项数目、
    main函数循环执行次数（累加统计）。

user_table_time：
    每查询一次用户态流表输出一次，每行分别为：
    时间戳、查找用户态流表时间
  ```

括号表示数据项的说明。

累加统计表示从ovs启动到当前时间戳为止发生的次数。

# OVS安装脚本源码的一些说明

- 用户态的数据使用rsysylog来记录，执行下面命令在文件/etc/rsyslog.conf中末尾追加一行(保存文件位置自行修改)：

  ```
  echo "local4.*    /var/log/gary.log" >>/etc/rsyslog.conf
  ```

  然后重启日志系统，在shell中运行如下命令

  ```
  /etc/init.d/rsyslog restart 
  ```

- 安装之前，确保你的内核版本不超过4.18，超过4.18的内核将无法安装，请更换内核。内核版本最好就是4.18，我尝试过4.15版本的内核，但是出现了问题。


- 为了防止安装过程中安装了linux自带的openvswitch模块，我们需要把目录`/lib/modules/(内核版本)/kernel/net/openvswitch/`删除。每个内核版本都会有这个目录，要把所有内核版本的都删掉。


- 安装基本的编译工具 `sudo apt-get install gcc make autoconf libtool  libelf-dev`


- 修改ovs源码目录下所有文件权限，在ovs源码目录中执行：`chmod -R 777 ./` 以及为脚本添加执行权限`chmod +x ./boot.sh`.
- 接下来运行如下编译安装命令(在执行这些安装命令之前建议执行下面的删除ovs命令)
  编译安装命令：

```
	./boot.sh  #不能执行，可能是没有执行权限，修改执行权限：sudo chmod +x ./boot.sh
    ./configure --with-linux=/lib/modules/$(uname -r)/build
    make -j4
    sudo make install
    sudo make modules_install
    sudo modprobe openvswitch
    lsmod | grep openvswitch #可以看到ovs相关的内核模块输出
    sudo mkdir -p /usr/local/etc/openvswitch
    sudo ovsdb-tool create /usr/local/etc/openvswitch/conf.db vswitchd/vswitch.ovsschema #可能报错说已经存在，这种情况不用管，继续
    sudo mkdir -p /usr/local/var/run/openvswitch
    sudo ovsdb-server --remote=punix:/usr/local/var/run/openvswitch/db.sock \
        --remote=db:Open_vSwitch,Open_vSwitch,manager_options \
        --private-key=db:Open_vSwitch,SSL,private_key \
        --certificate=db:Open_vSwitch,SSL,certificate \
        --bootstrap-ca-cert=db:Open_vSwitch,SSL,ca_cert \
        --pidfile --detach
    sudo ovs-vsctl --no-wait init
    sudo ovs-vswitchd --pidfile --detach --log-file
    ps -ea | grep ovs #有相关ovs进程输出
```


- 如果要删除OVS运行如下命令(如果需要删除的话)：

```
    sudo killall ovsdb-server
    sudo killall ovs-vswitchd
    sudo apt-get remove openvswitch-common openvswitch-datapath-dkms openvswitch-controller openvswitch-pki openvswitch-switch
    sudo ovs-dpctl del-dp ovs-system
    sudo modprobe -r openvswitch 
    sudo rmmod openvswitch
```

- 该ovs版本支持的linux内核为4.18，内核过高会出现许多问题，请使用4.18版本的内核！
