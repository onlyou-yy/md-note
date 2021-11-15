## CentOS搭建Git服务器及权限管理

## 1. 系统环境

系统： Linux：`CentOS 7.2 64位`

由于CentOS已经内置了`OpenSSH`,如果您的系统没有，请自行安装。

查看ssh版本

```shell
$ ssh -V

# 输出以下表示没问题，可以继续。 版本可能不一致，能用即可。
OpenSSH_6.6.1p1, OpenSSL 1.0.1e-fips 11 Feb 2013
```

> 避免系统环境和其他的不一致，请核对您系统的版本，其他发行版请对应修改。



## 2. 安装git

建议以下操作都切换到root

```shell
# 请确保您切换到了root账户
$ su root
$ yum install -y git

# 验证是否安装成功
$ git --version
# 输出如下内容表示成功：
git version x.x.x.x
```



## 3. 添加git的管理的账户和设置密码

设置专门管理git的账号非必须，但是建议这么操作。

```shell
# 添加git账户
$ adduser git

# 修改git的密码
$ passwd git
# 然后两次输入git的密码确认后。

# 查看git是否安装成功
$ cd /home && ls -al
# 如果已经有了git，那么表示成，参考如下：
drwxr-xr-x.  5 root root 4096 Apr  4 15:03 .
dr-xr-xr-x. 19 root root 4096 Apr  4 15:05 ..
drwx------  10 git  git  4096 Apr  4 00:26 git

# 默认还给我们分配一个名字叫git的组。
```



## 4. git的权限管理

git仓库的权限管理，我们可以手动进行管理和配置，也可以通过其他辅助工具。如果小团队的话，直接通过ssh公钥进行管理即可，如果大点的团队，最好用[gitolite](https://link.jianshu.com/?t=https%3A%2F%2Fgithub.com%2Fsitaramc%2Fgitolite) 或者 [gitosis](https://link.jianshu.com/?t=https%3A%2F%2Fgithub.com%2Fres0nat0r%2Fgitosis)，两者都差不多，一个是Perl开发，一个是Python开发。

以下我分别介绍手动管理权限和使用`gitolite`管理的方式，注意两者不兼容，不能混用。



## 5. git的手动权限管理

经过以上步骤，其实服务器的基本已经配置好，但是需要设置权限和配置远程访问git仓库的方式。我们只介绍ssh的方式，https不做介绍。



### 5.1 配置服务端的ssh访问

切换到git账号,并创建ssh的默认目录和校验公钥的配置文件

```shell
# 1.切换到git账号
$ su git
# 2.进入 git账户的主目录
$ cd /home/git

# 3.创建.ssh的配置，如果此文件夹已经存在请忽略此步。
$ mkdir .ssh

# 4. 进入刚创建的.ssh目录并创建authorized_keys文件,此文件存放客户端远程访问的 ssh的公钥。
$ cd /home/git/.ssh
$ touch authorized_keys

# 5. 设置权限，此步骤不能省略，而且权限值也不要改，不然会报错。
$ chmod 700 /home/git/.ssh/
$ chmod 600 /home/git/.ssh/authorized_keys
```

此时，服务端的配置基本完成。接下需要把客户端的公钥拷贝到`authorized_keys`文件中。



### 5.2 配置客户端的ssh私钥并上传服务器

以下是客户端创建ssh私钥和拷贝的过程，如果您有私钥越过创建私钥的过程。请用您的客户端进入终端（如果只有一台电脑，可以用不同的账号模拟不同客户端）

**第一步： 创建客户端的ssh私钥和公钥**

> 检查是否已经拥有ssh公钥和私钥：进入用户的主目录。
>
> 用户主目录：
> Windows系统：`C:\Users\用户名`
> Linux系统：`/home/用户名`
> Mac系统：`/Users/用户名`

然后查看是否有`.ssh`文件夹，此文件夹下是否有如下几个文件。

```shell
# 用户主目录的.ssh文件夹下
.ssh
├── id_rsa
└── id_rsa.pub  # 我们要用的私钥
```

如果没有，那么用ssh-keygen创建ssh的私钥。

```shell
$ ssh-keygen -t rsa

# 接下来，三个回车默认即可。
```

> 如果报了`ssh-keygen`不是有效命令可以添加一个环境变量`git安装路径\usr\bin`

创建私钥成功后，在查看用户目录是否有已加有了公钥文件id_rsa.pub

**第二步： 拷贝私钥到git的服务器**

如何把客户端的文件拷贝到服务器端，我建议用`scp`命令进行拷贝。

以下以mac系统为例：

```shell
# 首先进入我的用户主目录的.ssh目录下，注意用户名xxx替换成自己的
$ cd /Users/xxx/.ssh

# 以下命令是：把本地的id_rsa.pub文件拷贝到 aicoder.com服务器，登录aicoder.com服务的账号是git。
# 冒号后面默认就是git账号的主目录，最后文件被保存成laoma.pub
# 注意：把域名换成你自己的或者ip，最后的文件名可以自己定，后面还有用。
$ scp ./id_rsa.pub git@aicoder.com:.ssh/laoma.pub
```

> 使用这个命令拷贝的时候需要输入密码，但是明明输入对却死活无法完成的话，可以使用远程终端连接工具将文件直接上传上去对应的目录下



### 5.3 服务器端添加客户端的SSH公钥

切换到服务器端，把刚才上传的laoma.pub文件的内容添加到 authorized_keys中，就可以允许客户端ssh访问了。

```shell
# 切换到git账户
$ su git
$ cd /home/git/.ssh

$ ls -al
# 查看一下.ssh目录是否有authorized_keys和laoma.pub文件
# .
# |-- authorized_keys
# `-- laoma.pub

# 如果有，那么进行下面的把laoma.pub文件中的内容添加到authorized_keys中.
$ cat laoma.pub >> authorized_keys
```

> `>> `是在文件后面追加的意思，`> `是直接写入（覆盖）意思，主要如果用其他编辑器，每个ssh的pub要单独一行，建议用cat命令方便简单。echo也有类似功能
>
> ```shell
> #cat 也可以创建文件也可以写入东西到文件中
> cat > a.txt #创建后可以写入内容 按ctrl + D退出
> cat "hello" > a.txt #直接创建并写入内容
> cat << EOF > a.txt #创建后可以写入内容 末尾输入 EOF 退出
> ```

到此为止，您配置的客户端应该可以ssh的方式直接用git账号登录服务器。(当然不安全，后面可以控制)

```shell
# 在客户端用ssh测试连接远程服务器,请将域名aicoder.com换成你的ip地址或者域名
$ ssh git@aicoder.com    

# 第一次连接有警告，输入yes继续即可。如果可以连接上，那么恭喜你的ssh配置已经可以了。 
```



### 5.4 服务器端创建测试git仓库

进入服务器的终端。

```shell
# 切换到git账号
$ su git
# 进入git账号的用户主目录。
$ cd /home/git
# 在用户主目录下创建 test.git仓库的文件夹
$ mkdir test.git  && cd test.git
# 在test.git目录下初始化git仓库
$ git init --bare
# 输出如下内容，表示成功
Initialized empty Git repository in /home/git/test.git/
```

> git init --bare 是在当前目录创建一个裸仓库，也就是说没有工作区的文件，直接把git仓库隐藏的文件放在当前目录下，此目录仅用于存储仓库的历史版本等数据。

此时，客户端就可以进行clone或者remote add此仓库了。



### 5.5 客户端测试连接git远程仓库

**克隆远程创库**

```shell
git clone git@aicoder.com:test.git
```

> `git@aicoder.com:test.git`中 `git——刚刚创建的云主机用户`、`aicoder.com:-云主机ip或域名`、`test.git——创库名`，整句话的意思就是***克隆云主机上git用户目录下的test.git创库***

**提交本地创库到远程创库**

客户端，可以新建一个文件夹，初始化一个仓库，然后跟远程服务器上的空仓库建立连接。

```shell
# 以下shell代码，纯手写没有验证，如果有错误请自行纠正。
$ mkdir demos && cd demos
$ git init
$ touch a.txt
$ echo 'aicoder.com' >> a.txt
$ git add .
$ git commit -m 'the first commit'
# 把当前仓库跟远程仓库添映射
$ git remote add origin git@aicoder.com:test.git
# 把当前仓库push到远程仓库。
$ git push -u origin master
```

到此为止，我们就可以尽情的享用git私服了，但是！但是！但是！客户端可以直接ssh登录啊，这是bug，也是不安全的隐患，且看下面怎么禁用git账号的shell登录。



### 5.6 禁止客户端shell登录

因为前面我们添加了客户端的ssh的公钥到远程服务器，所以客户端可以直接通过shell远程登录服务器，这不安全，也不是我们想要的。且看下面如何禁用shell登录：

**第一步：**
给 `/home/git` 下面创建`git-shell-commands`目录，并把目录的拥有者设置为git账户。可以直接用git账号登录服务器终端操作。

```shell
$ su git
$ mkdir /home/git/git-shell-commands
```

> 此文件夹是git-shell用到的目录，需要我们手动创建，不然报错：`fatal: Interactive git shell is not enabled. hint: ~/git-shell-commands should exist and have read and execute access.`

**第二步：修改`/etc/passwd`文件，修改**

```shell
$ vim /etc/passwd

# 可以通过 vim的正则搜索快速定位到这行，  命名模式下  :/git:x

# 找到这句, 注意1000可能是别的数字
git:x:1000:1000::/home/git:/bin/bash
# 改为：
git:x:1000:1000::/home/git:/bin/git-shell

# 最好不要直接改，可以先复制一行，然后注释掉一行，修改一行，保留原始的，这就是经验！！！
# vim快捷键： 命令模式下：y 复制行， p 粘贴  0光标到行首 $到行尾 x删除一个字符  i进入插入模式 
# 修改完后退出保存：  esc进入命令模式， 输入：:wq!   保存退出。
```

好了，此时我们就不用担心客户端通过shell登录，只允许使用git-shell进行管理git的仓库。如果有其他小伙伴要连接git服务器，仅需要把他的公钥也添加到authorized_keys即可(`cat other.pub >> authorized_keys`)。

> 禁止客户端shell登录之后，我们再次在客户端执行`ssh git@aicoder.com:`的时候就会这样的了
>
> ```shell
> git>xxxx
> git>xxxx
> ```
>
> 这样就禁止了客户端通过shell登录了。
>
> 但是同样在服务端使用`su git`时会发现也是上面这种情况，要知道我们上面都是使用git用户来创建和授权`/home/git`下面的目录和文件的，如果此时需要创建一个新的创库但是我们无法切换到git用户，而是只能使用root进行创建，那么你会发现在提交的时候会提示你没有提交上去的权限。（**用哪个用户创建了`/home/git`目录及子目录文件，就要用哪个用户创建创库，否则提交时会报错**）
>
> ```shell
> fatal: Unable to create temporary file '/home/username/git/myrepo.git/./objects/pack/tmp_pack_XXXXXX': Permission denied
> error: pack-objects died of signal 13
> error: failed to push some refs to 'ssh://username@server.com/home/username/git/myrepo.git'
> ```
>
> 可以使用`ls -al /home/git`查看文件和目录的使用用户，会发现使用root创建的创库是root
>
> ```shell
> drwxr-xr-x  7 root root  4096 11月  1 20:53 newTest.git
> ```
>
> 我们可以使用`chown -R 所有者:所属组 文件或目录`来修改目录的拥有者，
>
> ```shelll
> chown -R git:git /home/git/newTest.git
> ```
>
> 之后再提交就可以了。



**原文链接**：[CentOS搭建Git服务器及权限管理](https://www.cnblogs.com/fly_dragon/p/8718614.html)