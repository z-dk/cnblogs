|  命令    |  描述    | 示例     |
| ---- | ---- | ---- |
|  shutdown    |   关机   |      |
| passwd/chpasswd | 设置/修改密码  | passwd  jen   修改jen用户的密码 |
| mount | 挂载外接设备，例如u盘  |   |
| chown/chgrp | 用来改变文件的所属用户/组  | # 修改a目录的用户和组为 xjj，chown -R xjj:xjj a  |
| chmod |  用来改变文件的访问权限  | # 毁灭性的命令：chmod 000 -R \/  # 给a.sh文件增加执行权限（这个太常用了)：chmod a+x a.sh|
| yum | centos的包管理工具 |  |
| systemctl | 后台服务管理：systemctl兼容了service命令  | service mysql restart、systemctl restart  mysqld（推荐）   |
| su | 切换用户  | su - xjj  |
| useradd/userdel | 添加/删除用户 | useradd zhangsan；userdel -f zhang3：f参数会在其他用户使用系统时强制退出 |
| usermod | 修改用户 | usermod  -d  /data/jen  jen           修改jen的用户目录 |

### 文件权限
![](https://img2020.cnblogs.com/blog/1457262/202103/1457262-20210307121900836-1284337874.png)
**文件权限分为三部分**

* 所有者权限：缩写为u。文件的所有者所拥有的权限。
* 组用户权限，缩写为g。文件所属组内所有用户的权限。
* 其他用户权限，缩写为o。其他不相关用户的权限。
* 全部，缩写为a，表示对上面三类用户集体操作。
**权限分类**
* r  表示可读权限。4：read。
* w 表示可写权限。2：write。写权限针对文件内容，拥有w权限未必可删除文件（需要拥有该文件上级目录的写权限）；
* x 表示可执行权限。1：execute。
* \-  权限占位符，表示没有当前权限。

###目录权限
* r 表示允许读取目录中的文件名，但不能进入该目录
* w 表示允许用户修改目录，可以创建、迁移、删除、更名目录下的文件
* x 可以获得目录下文件的列表，以及进入目录，执行cd命令
* t 粘贴位，普通用户不能删除其他用户的文件
