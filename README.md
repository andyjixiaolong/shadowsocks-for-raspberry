# shadowsocks-for-raspberry
新到手了树莓派4,没有ss总感觉怎么弄都不太舒服，所以折腾了半天。不保证一定正确和最优
## 安装shadowsocks
其实可以使用sudo apt-get install shadowsocks直接解决，奈何我申请的vps使用aes-256-gcm加密的，apt下来的好像没有这种加密方式，所以还是自己下载吧。
- 到https://github.com/shadowsocks/shadowsocks git clone 这个项目
- 进入目录，python setup.py --build 编译
- python setup.py --install 安装，应该可以使用--prefix指定安装目录
- suod vim /etc/shadowsocks/config.json 编辑服务器的信息
- 使用sudo sslocal -c /etc/shadowsocks/config.json -d -start 就可以运行ss了
- 设置开机自动启动，新建一个sh脚本，输入下列代码，加入执行权限，sudo chmod 755 shadowsocks.sh， 然后编辑开机启动脚本sudo vim /etc/rc.local，在exit 0 之前加入/home/pi/Documents/shadowsocks.sh
```
#!/bin/bash

sudo sslocal -c /etc/shadowsocks/config.json -d start
```
OK，shadowsocks就安装好了，但是socks5不支持http和https的协议，所以还需要安装代理，首先给chromimum安装switchyomega

## 安装SwitchyOmega
- 由于chromimum没有商店，所以到https://github.com/FelisCatus/SwitchyOmega/releases 下载.crx格式的最新插件
- 现在的chromimum不支持拖入安装，所以需要更改.crx为.zip或者.rar解压到文件夹之后，在chromimum插件页面导入整个文件夹
- 进入SwitchyOmega，在proxy页面配置ss，协议选socks，代理服务器127.0.0.1，端口1080
- 可以配置auto switch， 默认直连，代理规则选刚配置完的proxy，规则列表格式选AutoProxy，网址写入https://raw.githubusercontent.com/gfwlist/gfwlist/master/gfwlist.txt
好的，现在chromimum已经能上谷歌了，如果想要shell也能使用ss，就需要安装privoxy

## 安装privoxy
- 安装privoxy， sudo apt-get install privoxy
- 配置privoxy，sudo vim /etc/privoxy/config，找到并修改为以下代码
```
listen-address  127.0.0.1:8118
forward-socks5   /               127.0.0.1:1080 .
# 访问局域网不走ss
forward         192.168.*.*/     .
forward            10.*.*.*/     .
forward           127.*.*.*/     .
```
- 启动privoxy，systemctl start privoxy
- 现在进行测试，curl google.com --proxy 127.0.0.1:8118，如果有结果那么配置成功了，现在可以通过privoxy代理任意程序了
- 局部代理
- 局部代理 sudo vim /etc/local/bin/proxy，输入以下代码，需要使用时可以通过proxy加命令执行,例如proxy curl google.com
```
#!/bin/bash
http_proxy=http://127.0.0.1:8118 https_proxy=http://127.0.0.1:8118 ftp_proxy=ftp://127.0.0.1:8118 $*
```
大功告成，上个谷歌不容易

~~可能有人需要gfwlist设置网络代理，可以通过sudo pip install genpac安装genpac，然后sudo genpac --pac-proxy="SOCKS5 127.0.0.1:1080" -o autoproxy.pac --gfwlist-url="https://raw.githubusercontent.com/gfwlist/gfwlist/master/gfwlist.txt"获得~~

