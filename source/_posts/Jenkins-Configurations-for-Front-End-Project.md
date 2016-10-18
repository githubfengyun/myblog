---
title: Jenkins(基本篇)  - web前端项目自动化部署实例(基于linux系统)
date: 2016-10-16 20:26:45
tags: [Javascript, Angile develop]
---

## 部署的步骤

1. 新建一个项目(类型为自由风格).
2. 配置源码管理(现在主流的源码管理已经从SVN转到GIT, 所以本文介绍GIT配置方式)
  a. 配置有两种方式,一种为https, 一种为ssh. 我们这里使用ssh方式.
     在仓库url: git@github.xxx.com:GPMO/portal.git(请根据实际的服务器参考地址填写)
     这个时候一般会遇到ssh认证失败的问题. 这时通过以下步骤解决,原理是本地生成ssh私钥,把公钥加入到git服务器的部署key里面.
        1. sudo su - jenkins
        2. cd ~
        3. ssh-keygen -t dsa
        4. cd .ssh
        5. cat id_dsa.pub
        6. 把内容复制.
        7. 打开浏览器, 输入该项目的github地址,githu地址,找到settings并进入.
        8. 点击 Deploy keys
        9. 把复制的内容粘贴进来.
        10. 回到jenkins,复制出错的信息 git ls-remote -h git@xxx:GPMO/portal.git HEAD 
        11. 打开linux shell命令窗口,粘贴上并执行
        12. 这个时候千万要注意一下,如果是刚加入的ssh key, 需要打开linux窗口输入 git clone git@github.xxx.com:GPMO/portal.git
            当提示需要输入yes的时候输入yes,这个时候ssh key通道就打开了.
        13. 回到jenkins, 在仓库url地方把刚才地址剪切掉,点一下其他地方,再重新把该地址粘贴到仓库url这里.这样就激活这个仓库了.
  b. 分支填*/master就可以了, 如果是开发环境可以填*/develop分支.
3. 构建环境:
  a. 我一般在构建之前,初始化一个干净的工作空间,这个时候可以勾选在构建之前删除工作空间,请注意的是,请点击❓, 
    在pattens这里加上两个exclude, 我们不需要每次都删掉Node modules, 第二个不要删除.git目录.
    格式: Exclude: \**/node_modules/** 和 Exclude: \**/.git/**
  b. 环境变量,这个非常重要,如果不加入环境变量到jenkins的上下文,很多shell命令jenkins识别不了,会报错.
   勾选Inject environment variables to the build process.
   Properties Content填入:PATH=$PATH:$WORKSPACE:/usr/local/bin, 请根据自己的实际环境填写.
  c. Provide Node & npm bin/ folder to PATH
   请安装好nodejs, 在installation地方 自动会选上nodejs
4. 构建
    这里选择Execute shell(这个地方主要是当代码已经从github拉到工作空间以后的构建步骤)
    一般来说我们会是先安装相应的包, npm install, 安装完了以后,需要gulp或者webpack进行打包.
    清理实例的目录,拷贝工作目录到实例的目录,重启相关的服务.
    这里稍微要注意一下,如果后台用forever来管理nodejs进程,需要先定义一个新的BUILD_ID在jenkins环境,否则,forever进程会被jenkins杀掉.
    下面是一个列子,大家根据情况写.
    ```
    npm config set strict-ssl false
    npm config set proxy http://web-proxy.corp.hp.com:8080
    npm config set https-proxy https://web-proxy.corp.hp.com:8080
    npm config set registry "http://registry.npmjs.org/"
    npm i
    gulp watch-changed-without-liveload
    npm run build
    cd /usr/apps/GPMO/bin || true
    forever stop www || true
    cd /usr/apps/GPMO && rm -rf * || true
    cp -rf /home/sunp/jenkins_slave/workspace/gpmo/* /usr/apps/GPMO
    mkdir logs
    cd /usr/apps/GPMO/bin
    export BUILD_ID=dontKillMe
    export NODE_ENV=production && forever start www

    ```

5. 构建后步骤
    可以根据情况,配上email,当构建有问题的或者构建完成时通知相关的人员.这里不做详述.

## 下一篇我会介绍一下Jenkins集群的工作方法.

