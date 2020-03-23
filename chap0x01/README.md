# chapt0x01
## 软件环境
Windows

VirtualBox

Ubuntu 18.04 Server 64bit
## 如何配置无人值守安装iso并在Virtualbox中完成自动化安装。
### 1.手动安装一个Ubuntu虚拟机

### 2.配置双网卡
#### ifconfig 查看配置网卡：只显示NAT网络网卡

#### 按照老师视频修改01-netcfg.yaml文件

root权限：

        vi /etc/netplan/01-netcfg.yaml
        netplan apply

![修改yaml文件](img/3.png)

#### ifconfig显示双网卡，enp0s8处于DOWN状态

#### ifconfig enp0s8 up，开启UP状态

ip a显示如下

![查看网卡ip](/chap0x01/img/1.jpg)

### 3.putty连接虚拟机
#### 打开putty，输入host-only网卡ip，连接虚拟机
![putty连接虚拟机](img/2.jpg)
### 4.使用psftp将用于ubuntu18.04.4镜像文件复制进虚拟机
#### psftp连接虚拟机

open+'hostname'

输入用户名密码就可以连接

![pstfp连接](img/5.jpg)

#### 将镜像文件传送到虚拟机 ：

cd /home/lyl

put [本地文件地址+文件名] [虚拟机文件地址+文件名]

![传送镜像文件](img/5.png)

### 5.挂载iso文件，克隆光盘内容
#### 在当前用户目录下（/home/lyl）创建一个用于挂载iso镜像文件的目录
mkdir loopdir
#### 挂载iso镜像文件到该目录
root权限：mount  /home/lyl/ 1.iso loopdir

![挂载镜像文件](img/6.png)

#### 创建一个工作目录用于克隆光盘内容
mkdir cd
#### 同步光盘内容到目标工作目录
rsync -av loopdir/ cd

![克隆光盘内容](img/7.png)

#### 卸载iso镜像
umount loopdir
#### 进入目标工作目录
cd cd/
### 6.剩下步骤
#### 编辑Ubuntu安装引导界面增加一个新菜单项入口
vim isolinux/txt.cfg

添加以下内容到该文件

label autoinstall
  
  menu label ^Auto Install Ubuntu Server
  
  kernel /install/vmlinuz
  
  append  file=/cdrom/preseed/ubuntu-server-autoinstall.seed debian-installer/locale=en_US console-setup/layoutcode=us keyboard-configuration/layoutcode=us console-setup/ask_detect=false localechooser/translation/warn-light=true localechooser/translation/warn-severe=true
 initrd=/install/initrd.gz root=/dev/ram rw quiet---

 强制保存退出：
 Esc+wq！
 #### 用pstfp将老师提供ubuntu-server-autoinstall.seed传输到工作目录/home/cuc/cd/preseed/
 ![传输seed文件](img/8.png)

 #### 重新生成md5sum.txt
cd /home/lyl/cd && find . -type f -print0 | xargs -0 md5sum > md5sum.txt
#### 封闭改动后的目录到.iso
IMAGE=custom.iso

BUILD=/home/lyl/cd/
 
 apt-get install genisoimage

mkisofs -r -V "Custom Ubuntu Install CD" \
            -cache-inodes \
            -J -l -b isolinux/isolinux.bin \
            -c isolinux/boot.cat -no-emul-boot \
            -boot-load-size 4 -boot-info-table \
            -o $IMAGE $BUILD
#### 将custom.iso文件移到根目录，用psftp复制到本机
mv custom.iso ../

打开psftp

get custom.iso [本机地址]
#### 新建虚拟机，设置光驱为custom.iso，启动虚拟机，实现自动安装
自动安装视频已上传b站，还在审核。主页链接：https://space.bilibili.com/32520022
## Virtualbox安装完Ubuntu之后新添加的网卡如何实现系统开机自动启用和自动获取IP？
### 1.使用chkconfig命令让网络服务在系统启动级别是2345时默认启动。
　　chkconfig --level 2345 network on
### 2.修改网卡文件ifcfg-eth0，设置ONBOOT的值为yes，让网络服务启动时使用该网卡。设置BOOTPROTO的值为dhcp，让网卡从DHCP服务器自动获取IP地址。
vi /etc/sysconfig/network-scripts/ifcfg-eth0

　　ONBOOT=yes

　　BOOTPROTO=dhcp

　　service network start


## 如何使用sftp在虚拟机和宿主机之间传输文件？
### 1.端口映射

打开虚拟网络编辑器，打开NAT设置，给虚拟机ip添加端口映射 
### 2.打开ssh服务
sudo /etc/init.d/ssh start
### 3.查看端口是否打开
netstat -ano | findstr "22"
### 4.关闭宿主机防火墙
### 进行连接
sudo ssh user@remoteserveripaddr -p portnum
### 5.sftp传送文件
#### 首先登录上远程主机 ,再将本地文件传给远程主机
sftp user@remoteserveripaddr 
sftp> put /local.html /remote/
## 出现的问题
### 1.host-only网卡的ip地址不显示
第一次尝试的时候出现这个问题，向老师提问后，发现是关闭了DHCP服务器，开启后解决
第二次尝试出现此问题，看完老师在b站上的视频，发现是01-netcfg.yaml没有改，改后解决。
### 2.pstfp传送文件的时候总是出现找不到cd目录
![找不到路径](img/9.png)

![找不到路径2](img/10.png)

解决方法：修改路径格式。暂时还没有找到规律
### 或者是访问cd目录被拒绝
![拒绝](img/11.png)

解决方法：先传入根目录，再在虚拟机上用mv移到指定文件。没有找到原因。
### 在启动新的虚拟机的时候，出现下面的情况
![failed to load](img/12.png)

不知道之前那些步骤出了差错，只好从头再来一遍，又出现下面的情况
![seed failed](img/13.png)

咨询其他同学，说是txt.cfg文件写错，但是我的是复制的，不应该会错。

其他同学又说虚拟机必须加载iso镜像文件，尝试失败了两次（每次进入登陆页面，光驱就会消失）。

又只好再来一遍，最后一遍成功。应该是中间操作失误，不是镜像文件的问题。


## 参考文献 
https://c4pr1c3.github.io/LinuxSysAdmin/chap0x01.exp.md.html#/title-slide

https://blog.csdn.net/qq_31989521/article/details/58600426?depth_1-utm_source=distribute.pc_relevant.none-task&utm_source=distribute.pc_relevant.none-task

http://www.pgygho.com/help/fwq/16111.html

https://blog.csdn.net/banzhihuanyu/article/details/79169498

