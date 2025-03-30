# vsftpd 设置

安装步骤如下：

1.将此命令复制到终端

sudo apt-get install vsftpd

2.安装完成后启动VSFTPD服务：

service vsftpd start

3.新建目录/home/uftp作为用户主目录

sudo mkdir /home/uftp

4.制定用户并设置该用户密码

sudo useradd -d /home/uftp -s /bin/bash uftp

5.在将此命令复制终端

sudo chown uftp:uftp /home/uftp

6.新建文件/etc/vsftpd.user_list，用于存放允许访问ftp的用户：

sudo vi /etc/vsftpd.user_list

在其中添加用户uftp，并且保存退出：


7.编辑VSFTPD配置文件

sudo vi /etc/vsftpd.conf

修改：
　　打开注释 write_enable=YES
　　添加信息 userlist_file=/etc/vsftpd.user_list
　　添加信息 userlist_enable=YES
　　添加信息 userlist_deny=NO
　　修改完成后保存退出。

8.测试是否成功

