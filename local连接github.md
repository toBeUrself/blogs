# local 连接 github 的一些操作

>笔记本在重装了系统后，今天再次向 github 提交代码，所以需要重新建立连接.这里做一些笔记，未免忘记后又需要找教程费时间。

1. 在机器上创建ssh

`ssh-keygen -t rsa -C "ToBeUrself@163.com"`

**后面的`-C`加邮箱其实是一种注释，最好加上好了。后面可以设置生成路径和密码，全部 Enter 默认好了。然后找到id_rsa.pub文件，一般默认在`C:\Users\my\.ssh`目录下。把文件里面的内容拷贝下来，放到github里面。github->setting->ssh->add ssh**

2. 设置git name 和 email

`git config git config --global user.name 'Tim'`

`git config --global user.email "ToBeUrself@163.com"`

3. 可能仓库到本地

`git clone 仓库地址`

**之后在本地的一些update，就可以正常push到github，也可以fetch code。**
