

# windows中的文件映射到linux虚拟机中同步程序文件

为什么将windows中的文件映射到linux虚拟机中呢？很简单，你总不能在虚拟机vim写代码吧？如果能将windows中代码映射到linux虚拟机中，就可以在windows中用IDE騒代码，而又能在linux环境运行调试程序，爽得一批。当然，你会问为什么要将在linux环境运行代码呢？因为有些环境需要啊。

## 环境
win7 pro，为host
virtualbox 5.2.8
centos 7.2，为client

> 期望在ubuntu中可以挂载win10中的某个目录（如d:\data），且ubuntu拥有读写权限，系统启动时自动挂载。

## 方案

**方案有三个，我简单的谈一下前面两种，重点是最后一种nfs。如果对前面两种谈兴趣，可能这个教程不太合适。**

原因是：最后一种nfs的性能最优，还没这么麻烦。第一种以前用过，的确性能比较慢。第二种我没试过。

### 一、 使用vbox自带共享文件夹功能-vboxsf

- vbox设置共享文件夹，拥有完全控制权限，如设置别名为dwww
- client中的centos安装VBoxLinuxAddtions（增强功能）
- sudo vi /etc/fstab，增加如下一行：
`dwww /opt/www vboxsf rw,gid=username,uid=groupname,auto 0 0`

> 其中shared是vbox中设置的共享目录别名，/opt/www 是ubuntu中的挂载点，提前建好目录并chown给username，vboxsf是文件系统类型，参数rw是读写权限，后面用户及组名。重启centos，即可使用 /opt/www 来读写共享目录中的文件，修改同时同步到win7。

### 二、 使用win7自带共享文件夹功能-cifs

-  在win7开启文件夹共享，如设置别名为shared
-  centos中sudo vi /etc/fstab
`//192.168.1.9/dwww /data cifs _netdev,username=administrator,password=123456,uid=root,gid=root,auto 0 0`

> 其中ip地址和访问共享文件夹的win7用户名密码根据实际更换。

### 三、使用nfs方式 

windows7端中：

- 下载 haneWIN NFS
- 打开 haneWIN 中的 Exports 标签添加 【d:\www -name:dwww -alldirs】。然后Restart Sever。会在列表上看到/dwww。

> 记得启动haneWIN服务，安装后的系统菜单有一个【Start NFS Server】脚本。

virtualbox端中：

- linux上安装nfs客户端
`yum -y install nfs-utils`

- 显示NFS服务是否存在
`showmount -e 192.168.1.9`

- 手动挂载共享目录到centos7中
`mount -t nfs 192.168.1.9:/dwww /opt/dwww`

- 系统自启挂载共享目录到centos7中
	 `vim /etc/fstab`
	 在fstab文本尾行添加一行:
	 `192.168.1.9:/dwww       /opt/dwww               nfs     defaults        0 0`

