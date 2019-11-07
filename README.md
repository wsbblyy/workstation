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
## Vagrant的部署教程:   
1. 下载安装 vagrant 和 virtualbox 安装没什么说的都选默认就行   
2. 安装好这两个东西后 virtualbox 就不用管了因为它会随着 vagrant 而启动     
3. 打开命令行 输入命令 `vagrant -v` 可以看到vgrant版本   
4. 添加vagrant box, 其实就是就是虚拟系统, `vagrant box add {title} {url}`   
{title} 是你给你的 vagrant box 取得名字, {url} 是vagrant box 的地址, 可以是线上地址, 也可以是本地的   
vagrant box 没有国内源, https://pan.baidu.com/s/1Xn8Io-E9eA9Cyyly6kO5vQ 密码：r224, 这个是我上传的 ubuntu18 box   
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

现在我们`ls`一下`/`目录就可以看到`workspace`这个挂载目录, 它和你的主机上的`D:/workspace`是同一个目录并且数据是同步的    

7. (可选)如果在国内的话需要改下apt的源要不然安装东西会非常慢
    在Linux下运行   
    `cd /workspace`   
    `sudo touch sources.list`   
    在windows下用编辑器打开sources.list这个文件, 然后将下面的源复制进去, 这些是中科大的源   

    deb http://cn.archive.ubuntu.com/ubuntu/ bionic main restricted universe multiverse
    deb http://cn.archive.ubuntu.com/ubuntu/ bionic-security main restricted universe multiverse
    deb http://cn.archive.ubuntu.com/ubuntu/ bionic-updates main restricted universe multiverse
    deb http://cn.archive.ubuntu.com/ubuntu/ bionic-proposed main restricted universe multiverse
    deb http://cn.archive.ubuntu.com/ubuntu/ bionic-backports main restricted universe multiverse
    deb-src http://cn.archive.ubuntu.com/ubuntu/ bionic main restricted universe multiverse
    deb-src http://cn.archive.ubuntu.com/ubuntu/ bionic-security main restricted universe multiverse
    deb-src http://cn.archive.ubuntu.com/ubuntu/ bionic-updates main restricted universe multiverse
    deb-src http://cn.archive.ubuntu.com/ubuntu/ bionic-proposed main restricted universe multiverse
    deb-src http://cn.archive.ubuntu.com/ubuntu/ bionic-backports main restricted universe multiverse

    复制完后继续在Linux下运行   
    `sudo cp /etc/apt/sources.list /etc/apt/sources.list.bak` 备份sources.list以防万一   
    `sudo rm /etc/apt/sources.list` 删除原有list文件   
    `sudo mv /workspace/sources.list /etc/apt/sources.list` 把自建sources.list文件放入apt文件夹   
    `sudo apt-get update`   
    `sudo apt-get upgrade`   

8. 安装vim   
    `sudo apt-get install vim`    
    如果安装报错 说 需要 vim-common   
    `sudo apt-get remove vim-common`   
    再尝试   
    `sudo apt-get install vim`   
********************************************************** 
## Docker部署php+nginx教程:   
1. `sudo apt install docker.io` 安装Docker   
2. 建立Docker组, 这样在执行docker命令时就不用加sudo   
    `sudo groupadd docker` 创建docker组   
    `sudo gpasswd -a ${USER} docker` 把当前用户添加进去   
重启服务器就行了(`logout` 然后再 `vagrant ssh`)   
3. 如果再国内的话需要改Docker镜像   
    `sudo vim /etc/docker/daemon.json`   
    进入vim后复制下面的json(阿里云的docker镜像)   
    ```
    {    
        "registry-mirrors": ["https://jzngeu7d.mirror.aliyuncs.com"]   
    }  
    ``` 
    粘贴, 然后 `:wq` 保存退出   
4. 下载php和nginx的docker image   
    先看下`docker images` 应该是什么都没有   
    `docker pull nginx:alpine` 下载nginx image   
    `docker pull php:7.2-fpm-alpine` 下载php-fpm image   
    注意apline可以理解为纯净版, 建议所有服务都使用apline版本, 除非有特殊需求   
    现在再看`docker images` 应该就有php和nginx了       
5. 运行php实例(他们叫容器 container)   
    执行 `docker create --name myphp -p 9000:9000  -v    /workspace/php/:/www/var:rw -v     /workspace/config/php/conf.d:/usr/local/ect/php/conf.d:rw -v   /workspace/config/php/php-fpm.conf:/usr/local/etc/php-fpm.conf:rw php:7.2-fpm-alpine`   
    然后一定删除`/workspace/config/php/php-fpm.conf/`这个文件夹   
    然后执行`docker cp myphp:/usr/local/etc/php-fpm.conf   /workspace/config/php`   
    最后执行`docker start myphp`   
    解释一下这几个操作    
    1. `docker create` 创建一个实例   
    2. `--name myphp` 给这个实例取个名字myphp, 以后方便操作   
    3. `-p 9000:9000` 把实例中的9000端口映射给Linux系统的9000端口   
    4. `-v /workspace/php/:/www/var:rw`  把本机 `/workspace/php`(如果使用vagrant的话这个文件夹和D:/workspace/php也是同步的) 这个文件夹挂载到Docker实例中的`/www/var`这个文件夹中实现文件夹同步, `:rw`是赋予读写权限, 这个文件夹是放php项目的   
    `-v /workspace/config/php/conf.d:/usr/local/ect/php/conf.d:rw` 同上挂载目录, 这个文件夹是放php额外配置的   
    5. **重要** `-v    /workspace/config/php/php-fpm.conf:/usr/local/etc/php-fpm.conf:rw` 挂载文件`php-fpm.conf`文件, **注意**, 这里同步的文件而不是文件夹, docker会默认为php-fpm.conf为文件夹并创建一个, 所以执行create命令后要删除`/workspace/config/php/php-fpm.conf`文件夹, 不然php运行不了    
    6. `php:7.2-fpm-alpine` 该docker实例所需的image, 这个就是我们在前面第4步pull下来的   
    7. `docker cp`复制命令, 可以把实例中的文件/文件夹复制到本地, `myphp:`是实例名称, 接下来第一个参数为docker中的文件/文件夹, 第二个参数是目标文件/文件夹, **注意**, 如果目标是目录的话应该选择上级目录不然docker会给你多建个目录   
    8. `docker start` 启动docker实例(容器), 后面为实例(容器)名   
6. 运行nginx实例:   
    `docker create --name mynginx -p 80:80 -v /workspace/php:/www/var -v   /workspace/log/nginx:/var/log/nginx  -v    /workspace/config/nginx/nginx.conf:/etc/nginx/nginx.conf -v    /workspace/config/nginx/conf.d:/etc/nginx/conf.d --link myphp:php    nginx:alpine`   
    然后删除`/workspace/config/nginx/nginx.conf`文件夹    
    然后执行`docker cp mynginx:/etc/nginx/nginx.conf /workspace/config/nginx`   
    最后`dcoker start mynginx`   
    这行命令和上面启动php的命令类似, 有两个地方需要注意    
    1. `nginx.conf`也是个文件   
    2. `--link myphp:php`, 这行命令的意思是把上面运行的docke实例myphp给连接到mynginx实例中, 使用方法在nginx配置中会用到   
    3. nginx配置文件, 在`/worksapce/config/nginx/conf.d/`下面建立一个以`.conf`为结尾的nginx配置文件, 下面是我的配置文件    
        ```
        server {
            listen 80;
            listen [::]:80;

            server_name vm.phpinfo;
            
            root /www/var/phpinfo;
            index index.php index.html;

            location / {
                try_files $uri $uri/ /index.php?$query_string;
            }

            location ~ \.php$ {
                fastcgi_pass   php:9000;
                fastcgi_index  index.php;
                include        fastcgi_params;
                fastcgi_param  SCRIPT_FILENAME $document_root$fastcgi_script_name  ;
            }
        }
        ```
    nginx配置需要注意的有这几个地方   
    `server_name vm.phpinfo`, vm.phpinfo 是我在windows系统中的hosts文件添加的域名, ip是给vagrant分配的private_network ip, 就是vm.phpinfo就可以访问该项目了
    `root /www/var/phpinfo`, 这个是项目地址, 因为我把`/workspace/php`挂载到了docker实例中的`/www/var`文件夹中, 所以`/workspace/php/phpinfo`就等于`/www/var/phpinfo`    
    `fastcgi_pass php:9000`, 这里的`php`和运行nginx的docker实例命令`--link myphp:php`中的`php`是对应的     
    4. `docker restart mynginx`, 重启nginx, 我们该nginx配置后都要运行该命令重启   
7. 准备好了之后就可以在`/workspace/php/phpinfo`中建立一个`index.php`文件, 里面写上     
    ```
    <?php
    phpinfo();
    ```
    然后浏览器输入vm.phpinfo打开, 应该就可以看到phpinfo了

**重要**   
如果`docker run` 然后 或者 `docker start ` , `docker restart` 后, 查看`docker ps` 并没有看到启动的实例说明该实例出错了, 很有可能是配置文件写错了, 看看日志     
