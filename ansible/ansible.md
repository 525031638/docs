# Ansible

Ansible是一种IT自动化工具。它可以配置系统，部署软件以及协调更高级的IT任务，例如持续部署或零停机滚动更新。Ansible基于 Python 语言实现，由 Paramiko 、PyYAML、Jinja2 三个关键模块构建。

**Ansible 特点：**

- 部署简单，agentless，默认使用ssh协议进行管理
- 配置简单、功能强大、扩展性强，支持API及自定义模块
- 通过强大的Playbooks定制配置、状态管理
- 对云平台、大数据都有很好的支持
- 幂等性

**Ansible架构图：**

![IMG_2064](IMG_2064.PNG)



### Ansible常用模块

根据官方的分类，将模块按功能分类为：云模块、命令模块、数据库模块、文件模块、资产模块、消息模块、监控模块、网络模块、通知模块、包管理模块、源码控制模块、系统模块、单元模块、web设施模块、windows模块等 。

### 一、ping模块

> ###### 测试主机是否是通的，用法很简单，不涉及参数：

```
ansible test -m ping
```

### 二、setup模块

> ###### setup模块，主要用于获取主机信息，在playbooks里经常会用到的一个参数gather_facts就与该模块相关。setup模块下经常使用的一个参数是filter参数，具体使用示例如下：

```
ansible 10.212.52.252 -m setup -a 'filter=ansible_*_mb'   //查看主机内存信息
```
```
ansible 10.212.52.252 -m setup -a 'filter=ansible_eth[0-2]'   //查看地接口为eth0-2的网卡信息
```
```
ansible all -m setup --tree /tmp/facts   //将所有主机的信息输入到/tmp/facts目录下，每台主机的信息输入到主机名文件中（/etc/ansible/hosts里的主机名）
```

### 三、file模块

> ###### file模块主要用于远程主机上的文件操作，file模块包含如下选项：

#### 常用参数

- force：需要在两种情况下强制创建软链接，一种是源文件不存在但之后会建立的情况下；另一种是目标软链接已存在,需要先取消之前的软链，然后创建新的软链，有两个选项：yes|no
- group：定义文件/目录的属组
- mode：定义文件/目录的权限
- owner：定义文件/目录的属主
- path：必选项，定义文件/目录的路径, required
- recurse：递归的设置文件的属性，只对目录有效
- src：要被链接的源文件的路径，只应用于state=link的情况
- dest：被链接到的路径，只应用于state=link的情况
- state：
  - directory：如果目录不存在，创建目录
  - file：即使文件不存在，也不会被创建
  - link：创建软链接
  - hard：创建硬链接
  - touch：如果文件不存在，则会创建一个新的文件，如果文件或目录已存在，则更新其最后修改时间
  - absent：删除目录、文件或者取消链接文件

#### 使用示例：

```
ansible v8 -m file -a "src=/run/resolvconf/resolv.conf dest=/etc/resolv.conf state=link"   //创建软连接
```
```
ansible test -m file -a “path=/tmp/fstab state=absent”   //删除文件
```
```
ansible test -m file -a “path=/tmp/test state=touch”   //创建文件
```
```
ansible all -m file -a "dest=~/t.py mode=777 owner=ashin group=ashin"   //修改文件权限
```
### 四、copy模块

> ###### 复制文件到远程主机

#### 常用参数

- backup：在覆盖之前将原文件备份，备份文件包含时间信息。有两个选项：yes|no
- content：用于替代”src”,可以直接设定指定文件的值
- dest：必选项。要将源文件复制到的远程主机的绝对路径，如果源文件是一个目录，那么该路径也必须是个目录, required.
- directory_mode：递归的设定目录的权限，默认为系统默认权限
- force：如果目标主机包含该文件，但内容不同，如果设置为yes，则强制覆盖，如果为no，则只有当目标主机的目标位置不存在该文件时，才复制。默认为yes
- others：所有的file模块里的选项都可以在这里使用
- src：要复制到远程主机的文件在本地的地址，可以是绝对路径，也可以是相对路径。如果路径是一个目录，它将递归复制。在这种情况下，如果路径使用”/“来结尾，则只复制目录里的内容，如果没有使用”/“来结尾，则包含目录在内的整个内容全部复制，类似于rsync。
- validate ：The validation command to run before copying into place. The path to the file to validate is passed in via ‘%s’ which must be present as in the visudo example below.

#### 示例如下
```
ansible test -m copy -a “src=/srv/foo.conf dest=/etc/foo.conf owner=foo group=foo mode=0644”   //往服务器拷贝文件并修改所属权限
```
```
ansible test -m copy -a “src=/mine/sudoers dest=/etc/sudoers validate=’visudo -cf %s’”   //往服务器拷贝文件，并保留之前权限
```
```
ansible v6 -m copy -a "src=/home/log/killlog.sh dest=/root/mon"   //拷贝文件
```
### 五、service/systemd模块

> ###### 用于管理服务

#### 常用参数

- arguments：给命令行提供一些选项

- enabled：是否开机启动 yes|no

- name：必选项，服务名称

- pattern：定义一个模式，如果通过status指令来查看服务的状态时，没有响应，就会通过ps指令在进程中根据该模式进行查找，如果匹配到，则认为该服务依然在运行

- runlevel：运行级别

- sleep：如果执行了restarted，在则stop和start之间沉睡几秒钟

- state：对当前服务执行启动，停止、重启、重新加载等操作（started, stopped, restarted, reloaded）

  #### 使用示例
```
ansible test -m service -a “name=httpd state=started enabled=yes”   //启动httpd服务
```
```
asnible test -m service -a “name=foo pattern=/usr/bin/foo state=started”   //启动foo服务
```
```
ansible test -m service -a “name=network state=restarted args=eth0”   //重启网卡
```
### 六、cron模块

> ###### 用于管理计划任务

#### 常用参数

- backup：对远程主机上的原任务计划内容修改之前做备份
- cron_file：如果指定该选项，则用该文件替换远程主机上的cron.d目录下的用户的任务计划
- day：日（1-31，，/2,……）
- hour：小时（0-23，，/2，……）
- minute：分钟（0-59，，/2，……）
- month：月（1-12，，/2，……）
- weekday：周（0-7，*，……）
- job：要执行的任务，依赖于state=present
- name：该任务的描述
- special_time：指定什么时候执行，参数：reboot,yearly,annually,monthly,weekly,daily,hourly
- state：确认该任务计划是创建还是删除
- user：以哪个用户的身份执行

#### 使用示例
```
ansible test -m cron -a ‘name=”a job for reboot” special_time=reboot job=”/some/job.sh”‘   //在系统重启的时候执行job.sh
```
```
ansible test -m cron  -a ‘backup=”True” name=”test” minute=”0” hour=”5,2” job=”ls -alh > /dev/null”‘  //添加任务计划
```
```
ansilbe test -m cron -a ‘cron_file=ansible_yum-autoupdate state=absent’   //删除任务计划
```
```
ansible 10.23.67.85 -m cron -a "name=logcheck minute=*/5 job='/bin/bash /root/log/log.sh >/dev/null 2>&1' state=present"   //添加任务计划
```
### 七、yum/apt模块

> ###### 使用yum/apt包管理器来管理软件包

#### 常用参数

- config_file：yum的配置文件
- disable_gpg_check：关闭gpg_check
- disablerepo：不启用某个源
- enablerepo：启用某个源
  *　name：要进行操作的软件包的名字，也可以传递一个url或者一个本地的rpm包的路径
- state：状态（present，absent，latest）

#### 使用示例
```
ansible test -m yum -a ‘name=httpd state=latest’   //确保安装的软件是最新版
```
```
ansible test -m apt -a "name=python-pip state=present" --sudo -K //使用apt安装软件，并使用sudo权限
```
```
ansible test -m yum -a ‘name=”@Development tools” state=present’   //安装一个软件包组
```
```
ansible test -m yum -a ‘name=http://nginx.org/packages/centos/6/noarch/RPMS/nginx-release-centos-6-0.el6.ngx.noarch.rpm state=present’   //指定安装包安装软件
```

### 八、user模块

> ###### 用户管理

#### 常用参数

- home：指定用户的家目录，需要与createhome配合使用
- groups：指定用户的属组
- uid：指定用的uid
- password：指定用户的密码
- name：指定用户名
- createhome：是否创建家目录 yes|no
- system：是否为系统用户
- remove：当state=absent时，remove=yes则表示连同家目录一起删除，等价于userdel -r
- state：是创建还是删除
- shell：指定用户的shell环境

#### 使用示例：
```
ansible test -m user -a 'name=johnd comment=”John Doe” uid=1040 group=admin'   //添加用户指定组
```
```
ansible test -m user -a 'name=james shell=/bin/bash groups=admins,developers append=yes user: name=johnd state=absent remove=yes'   //删除用户并删除家目录
```
```
ansible test -m user -a 'name=james18 shell=/bin/zsh groups=developers expires=1422403387'
```
```
ansible test -m user -a 'name=test generate_ssh_key=yes ssh_key_bits=2048 ssh_key_file=.ssh/id_rsa'   //生成密钥时，只会生成公钥文件和私钥文件，和直接使用ssh-keygen指令效果相同，不会生成authorized_keys文件。
```
> ###### 注：指定password参数时，不能使用明文密码，因为后面这一串密码会被直接传送到被管理主机的/etc/shadow文件中，所以需要先将密码字符串进行加密处理。然后将得到的字符串放到password中即可。
>
> ```
> echo "123456" | openssl passwd -1 -salt $(< /dev/urandom tr -dc '[:alnum:]' | head -c 32) -stdin
> $1$4P4PlFuE$ur9ObJiT5iHNrb9QnjaIB0
> #使用上面的密码创建用户
> ansible all -m user -a 'name=foo password="$1$4P4PlFuE$ur9ObJiT5iHNrb9QnjaIB0"'
> 不同的发行版默认使用的加密方式可能会有区别，具体可以查看/etc/login.defs文件确认，centos 6.5版本使用的是SHA512加密算法。
> ```

### 九、group模块

> ###### 组管理模块

#### 常用参数

- gid ：指定gid
- name : 指定组名称
- state ：操作状态，present,absent
- system : 是否为系统组

#### 使用示例
```
ansible all -m group -a 'name=somegroup state=present'    //创建组
```
### 十、synchronize模块

> ###### 使用rsync同步文件

#### 常用参数

- archive: 归档，相当于同时开启recursive(递归)、links、perms、times、owner、group、-D选项都为yes ，默认该项为开启
- checksum: 跳过检测sum值，默认关闭
- compress:是否开启压缩
- copy_links：复制链接文件，默认为no ，注意后面还有一个links参数
- delete: 删除不存在的文件，默认no
- dest：目录路径
- dest_port：默认目录主机上的端口 ，默认是22，走的ssh协议
- dirs：传速目录不进行递归，默认为no，即进行目录递归
- rsync_opts：rsync参数部分
- set_remote_user：主要用于/etc/ansible/hosts中定义或默认使用的用户与rsync使用的用户不同的情况
- mode: push或pull 模块，push模的话，一般用于从本机向远程主机上传文件，pull 模式用于从远程主机上取文件

#### 使用示例
```
ansible all -m synchronize -a "src=/home/ubuntu/newserver20161008.tar.gz dest=/home/ubuntu mode=push"    //往服务器传同步文件
```
```
ansible dns -m synchronize -a "src=some/relative/path dest=/some/absolute/path archive=no links=yes"    //同步文件开启雅俗
```
```
ansible dns -m synchronize -a "src=some/relative/path dest=/some/absolute/path checksum=yes times=no'    //同步文件，检查sum
```
```
ansible dns -m synchronize -a "src=/tmp/helloworld dest=/var/www/helloword rsync_opts=—no-motd,—exclude=.git mode=pull"    //同步文件，排除.git
```
### 十一、filesystem模块

> ###### 在块设备上创建文件系统

#### 常用参数

- dev：目标块设备
- force：在一个已有文件系统 的设备上强制创建
- fstype：文件系统的类型
- opts：传递给mkfs命令的选项

#### 使用示例
```
ansible test -m filesystem -a ‘fstype=ext2 dev=/dev/sdb1 force=yes’    //创建ext2文件系统
```
```
ansible test -m filesystem -a ‘fstype=ext4 dev=/dev/sdb1 opts=”-cc”‘    //创建ext4文件系统
```
### 十二、mount模块

> ###### 配置挂载点

#### 常用参数

- fstype：必选项，挂载文件的类型
- name：必选项，挂载点
- opts：传递给mount命令的参数
- src：必选项，要挂载的文件
- state：必选项
- present：只处理fstab中的配置
- absent：删除挂载点
- mounted：自动创建挂载点并挂载之
- umounted：卸载

#### 使用示例：

```
name=/mnt/dvd src=/dev/sr0 fstype=iso9660 opts=ro state=present
name=/srv/disk src='LABEL=SOME_LABEL' state=present
name=/home src='UUID=b3e48f45-f933-4c8e-a700-22a159ec9077' opts=noatime state=present
ansible test -a 'dd if=/dev/zero of=/disk.img bs=4k count=1024'
ansible test -a 'losetup /dev/loop0 /disk.img'
ansible test -m filesystem 'fstype=ext4 force=yes opts=-F dev=/dev/loop0'
ansible test -m mount -a 'name=/mnt src=/dev/loop0 fstype=ext4 state=mounted opts=rw'
ansible $ip -m mount -a 'name=/data/images src=10.23.117.31:/data/images fstype=nfs state=mounted opts=rw'
```

### 十三、get_url 模块

> ###### 该模块主要用于从http、ftp、https服务器上下载文件（类似于wget）

#### 常用参数

- sha256sum：下载完成后进行sha256 check；
- timeout：下载超时时间，默认10s
- url：下载的URL
- url_password、url_username：主要用于需要用户名密码进行验证的情况
- use_proxy：是事使用代理，代理需事先在环境变更中定义

#### 使用示例：
```
ansible test -m get_url -a 'url=http://example.com/path/file.conf dest=/etc/foo.conf mode=0440'  
```
```
ansible test -m get_url -a 'url=http://example.com/path/file.conf dest=/etc/foo.conf sha256sum=b5bb9d8014a0f9b1d61e21e796d78dccdf1352f23cd32812f4850b878ae4944c'
```
### 十四、unarchive模块

> ###### 用于解压文件，模块包含如下选项：

#### 常用参数

- copy：在解压文件之前，是否先将文件复制到远程主机，默认为yes。若为no，则要求目标主机上压缩包必须存在。
- creates：指定一个文件名，当该文件存在时，则解压指令不执行
- dest：远程主机上的一个路径，即文件解压的路径
- grop：解压后的目录或文件的属组
- list_files：如果为yes，则会列出压缩包里的文件，默认为no，2.0版本新增的选项
- mode：解决后文件的权限
- src：如果copy为yes，则需要指定压缩文件的源路径
- owner：解压后文件或目录的属主

#### 使用示例
```
ansible test -m unarchive -a 'src=foo.tgz dest=/var/lib/foo'
```
```
ansible test -m unarchive -a 'src=/tmp/foo.zip dest=/usr/local/bin copy=no'
```
```
ansible test -m unarchive -a 'src=https://example.com/example.zip dest=/usr/local/bin'
```

### 十五、Git模块

> ###### 用于解压文件，模块包含如下选项：

#### 常用参数

- copy：在解压文件之前，是否先将文件复制到远程主机，默认为yes。若为no，则要求目标主机上压缩包必须存在。
- creates：指定一个文件名，当该文件存在时，则解压指令不执行
- dest：远程主机上的一个路径，即文件解压的路径
- grop：解压后的目录或文件的属组
- list_files：如果为yes，则会列出压缩包里的文件，默认为no，2.0版本新增的选项
- mode：解决后文件的权限
- src：如果copy为yes，则需要指定压缩文件的源路径
- owner：解压后文件或目录的属主

#### 使用示例
```
ansible test -m git -a "repo=https://github.com/axiaoxin/json_line_format.vim.git dest=~/project-dir version=HEAD"   //dest目录必须是空文件夹或者还不存在的文件夹
```

### 十六、script模块

> ###### 用于在远程主机执行本地的脚本：

#### 常用参数

- copy：在解压文件之前，是否先将文件复制到远程主机，默认为yes。若为no，则要求目标主机上压缩包必须存在。
- creates：指定一个文件名，当该文件存在时，则解压指令不执行
- dest：远程主机上的一个路径，即文件解压的路径
- grop：解压后的目录或文件的属组
- list_files：如果为yes，则会列出压缩包里的文件，默认为no，2.0版本新增的选项
- mode：解决后文件的权限
- src：如果copy为yes，则需要指定压缩文件的源路径
- owner：解压后文件或目录的属主

#### 使用示例
```
ansible test -m script -a "/home/ubuntu/killlog.sh"   //dest目录必须是空文件夹或者还不存在的文件夹
```

### 十五、raw模块

> ###### 用于解压文件，模块包含如下选项：

#### 常用参数

- copy：在解压文件之前，是否先将文件复制到远程主机，默认为yes。若为no，则要求目标主机上压缩包必须存在。
- creates：指定一个文件名，当该文件存在时，则解压指令不执行
- dest：远程主机上的一个路径，即文件解压的路径
- grop：解压后的目录或文件的属组
- list_files：如果为yes，则会列出压缩包里的文件，默认为no，2.0版本新增的选项
- mode：解决后文件的权限
- src：如果copy为yes，则需要指定压缩文件的源路径
- owner：解压后文件或目录的属主

#### 使用示例
```
ansible test -m git -a "repo=https://github.com/axiaoxin/json_line_format.vim.git dest=~/project-dir version=HEAD"   //dest目录必须是空文件夹或者还不存在的文件夹
```

### 十五、fetch模块

> ###### 用于从远程服务器拉取文件，必须是file，不能是目录，模块包含如下选项：

#### 常用参数

- copy：在解压文件之前，是否先将文件复制到远程主机，默认为yes。若为no，则要求目标主机上压缩包必须存在。
- creates：指定一个文件名，当该文件存在时，则解压指令不执行
- dest：远程主机上的一个路径，即文件解压的路径
- grop：解压后的目录或文件的属组
- list_files：如果为yes，则会列出压缩包里的文件，默认为no，2.0版本新增的选项
- mode：解决后文件的权限
- src：如果copy为yes，则需要指定压缩文件的源路径
- owner：解压后文件或目录的属主

#### 使用示例
```
ansible 10.23.46.236 -m fetch -a "src=/data/tomcat.sh dest=/home/ubuntu/"
   //dest目录必须是空文件夹或者还不存在的文件夹
```

###十六、authorized_key模块

> ###### 用于为特定用户帐户添加或移除SSH授权密钥，模块包含如下选项：

#### 常用参数

- exclusive：是否从authorized_keys文件删除所有其他未指定的keys。
- key：指定ssh公钥
- key_options：添加ssh密钥选项
- manage_dir：是否管理授权密钥文件的目录，如果设置，将会创建目录，设置所有者和权限
- path：authorized_keys路径，默认在用户家目录下.ssh/authorized_keys
- state：present，absent
- user：远程主机的用户名，其授权的密钥文件将被修改
- validate_certs：这仅适用于使用HTTPSURL作为密钥的来源。如果设置为“否”，则SSL证书将不被验证

#### 使用示例

```
ansible all -m authorized_key -a "user=test key='{{ lookup('file', '/home/test/.ssh/id_rsa.pub') }}' path=/home/test/.ssh/authorized_keys manage_dir=yes"
```