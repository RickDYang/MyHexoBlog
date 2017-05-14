---
title: VPS迁移备忘
mathjax: true
categories: 备忘
tags: [备忘]
date: 2017-05-14
toc: true
---
前几天把VPS的服务从Vultr迁移到了Google Cloud。Google Cloud的asia-east1-a服务器国内访问快得飞起。主要的任务是迁移shadowsocks和Hexo Blog。这里记录一下迁移的步骤以备忘。
<!--more-->
## 安装Shadowsocks ##
只需要下面的几条指令
```bash
sudo apt-get update
sudo apt-get install python-pip
sudo pip install shadowsocks
```
### 配置Config ###
在etc下创建编辑shadowsocks.json
```json
{
    "server":"0.0.0.0",
    "server_port":8388,
    "local_address":"127.0.0.1",
    "local_port":1080,
    "password":"******",
    "timeout":600,
    "method":"aes-256-cfb",
    "fast_open": false
}
```
由于Google Cloud上创建的是VM，即使有外部IP，shadowsocks只能绑定到其内部IP上，所以简单起见，"server"指定成"0.0.0.0"即可。
config编辑完成后，可以尝试调用下面的命令，看看ssserver能够正常启动运行。
```bash
sudo ssserver -c /etc/shadowsocks.json
ps -A | grep ssserver
```
### 自动运行shadowsocks ###
编辑rc.local，设置开机启动
```bash
sudo vim /etc/rc.local
```
加入一行
```bash
sudo ssserver -c /etc/shadowsocks.json -d start
```
### 打开防火墙端口 ###
上面的配置中"server_port"设成的是8388,所以我们需要在云服务上的防火墙设置中打开tcp:8388。
不同的云服务商的方式有所不同。Vultr是默认都打开的，所以这步不是必须的。
## 迁移Hexo博客 ##
### 安装Hexo ###
安装git
```bash
sudo apt-get install git-core
```
安装 Node.js
cURL:
```bash
curl https://raw.github.com/creationix/nvm/master/install.sh | sh
```
Wget:
```bash
wget -qO- https://raw.github.com/creationix/nvm/master/install.sh | sh
```
安装完成后，重启终端并执行下列命令即可安装 Node.js
```bash
nvm install stable
```
安装Hexo
需要cd到当前用户权限的目录，比如$HOME。Hexo会在当前目录创建hexo目录，并在hexo目录下安装。
```bash
npm install -g hexo-cli
```
调用
```bash
hexo g
```
即可生成测试的hello world页面
### 安装NexT Theme ###
cd到Hexo目录，比如$HOME/hexo
```bash
git clone https://github.com/iissnan/hexo-theme-next themes/next
```
### 安装nginx ###
```bash
sudo apt-get install nginx
```
启动ngnix
```bash
sudo /etc/init.d/nginx start
```
这时在浏览器中输入主机ip就能访问nginx的Welcome页面了
### 集成Hexo ###
这时我们需要把nginx的主页指向hexo的输出页面。
修改Nginx的配置
```bash
vim /etc/nginx/sites-available/default
```
修改root指向hexo输出的public目录
```
root /home/***/hexo/public;
```
重启nginx
```bash
sudo service nginx restart
```
这时在浏览器中输入主机ip，就打开的是hexo blog的页面了
### 从GitHub同步 ###
我的hexo博客的source数据和站点/主题的配置文件都放在我的github专门的repo上了。
从github上checkout下来即可。不过需要把站点/主题的配置文件（\_config.yml  & themes/next/\_config.yml 都暂时删掉，避免冲突。
```bash
git init
git remote add origin PATH/TO/REPO
git fetch
git checkout -t origin/master
```
Blog数据同步完成之后，运行
```bash
hexo g
```
就能看到所有的博客数据完全生成了。
### js特效 ###
我的博客加了一个js特效
编辑
```bash
vim themes/next/layout/_layout.swig
```
加入下面的js代码
```html
<script type="text/javascript" color="0,255,255" opacity='0.7' zIndex="-2" count="99" src="//cdn.bootcss.com/canvas-nest.js/1.0.1/canvas-nest.min.js"></script>
</body>
```
### 配置SSH Key ###
配置SSH Key作用是让hexo能够把blog同步到你的.github.io上
生成当前VM的SSH密钥
```bash
ls ~/.ssh
```
如果没有id_rsa & id_rsa.pub代表没有生成过SSH密钥
用你的email生成ssh密钥
```bash
ssh-keygen -t rsa -C "your_email@example.com"
```
提示都直接回车即可。
复制id_rsa.pub里的内容
```bash
vim ~/.ssh/id_rsa.pub
```
在.github.io repo的settings里，添加id_rsa.pub里的内容到Deploy Keys中。
这时调用
```bash
hexo d
```
hexo就会把生成的blog自动同步到github上了。
## 域名指向 ##
登陆GoDaddy，把域名指向新的VPS IP。大功告成。