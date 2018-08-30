# 终极开发工作站 PC + Vagrant + Docker

## WVD 或者 MVD 解决的问题

Windows/Mac + Vagrant 解决了用win/mac开发没有精准的linux命令行
而用Linux系统开发却不没有开发者习惯的用户界面和软件

首先我们说下这三个东西的关系:
Windows/Mac 显然是住在你的设备里
Vagrant是Windows/Mac中的一个软件, 可以提供一个Linux虚拟主机, 其实也就是命令行
Docker是Linux下的一个软件, 所以Docker是住在Vagrant提供的Linux虚拟系统里
最重要的一点是 **它们三个东西的文件夹和文件是可以共同的**
********************************************************
Vagrant 可以实现: 用非Linux系统却能单独使用Linux命令行, 并且(也是最重要的), Linux命令行和你的本机可以共享文件夹
以及文件夹里的文件

![](./imgs/1.png 'Windwos中的Linux命令行 工具是cmder')
![](./imgs/2.png 'Windwos文件浏览')

可以看到上面的Linux命令行中的workspace文件夹和windows中的workspace文件夹是**共用**的
********************************************************
Docker 可以实现: Docker的话大家应该都熟悉了, 把服务都Docker化, 比如php, nginx, redis 这些服务都不单独部署, 而是通过Docker来部署, 那么就可以百分百确保环境的统一而避免各种不必要的调试
********************************************************
那我们为啥要用这种一环套一环的开发环境呢, 有以下3个原因
1. 开发方便
2. 让个人电脑变得干净整洁, 因为所有的服务都被打包到Docker和Vagrant里了
3. 部署方便, 环境统一, 这其实就是Docker的特性

********************************************************
##Vagrant的部署教程:
1. 下载安装 vagrant 和 virtualbox 安装没什么说的都选默认就行
2. 安装好这两个东西后 virtualbox 就不用管了因为它会随着 vagrant 而启动
3. 打开命令行 输入命令 `vagrant -v` 可以看到vgrant版本
4. 添加vagrant box, 其实就是就是虚拟系统, `vagrant box add {title} {url}`
{title} 是你给你的 vagrant box 取得名字, {url} 是vagrant box 的地址, 可以是线上地址, 也可以是本地的
vagrant box 没有国内源, https://c4ys.com/archives/1230 这个我网搜的百度云的box, 
或者用vpn到 https://app.vagrantup.com/boxes/search 和 http://www.vagrantbox.es/ 找合适的box
我的是下载好的box, cd 到下载文件夹文件夹, 然后运行命令`vagrant box add centos7 Vagrant-CentOS-7.box`
`centos7`是我给这个实例取的名字, `Vagrant-CentOS-7.box`是box文件
运行`vagrant box list`就能看见创建好的实例了
5. cd 到一个新建的文件夹中执行`vagrant init centos7`(这里因为我实例取的名字教centos7), 在该文件夹中会自动生成`Vagrantfile`这个文件
打开`Vagrantfile`
    1. 取消`config.vm.network "private_network"`的注释, ip可以自定义但是建议不要以1结尾, 有未知错误, 我的是'192.1.1.0'
    通过这个IP就可以访问你的Linux主机了
    2. 取消`config.vm.synced_folder`的注释, 第一个参数是本机想要挂载的目录, 第二参数是Linux目标目录
    我是这样配置的 **config.vm.synced_folder "D:/workspace", "/workspace"**
6. 现在运行`vagrant up`启动virtualbox, 第一次运行需要选择虚拟机, 选virtualbox就行, 然后运行`vagrant ssh`就可以进入Linux服务器了