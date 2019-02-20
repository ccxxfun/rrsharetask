# 快速安装人人影视、自动任务、清理任务。

# 1、国外 KVM或者杜甫 先安装BBR
wget -N --no-check-certificate "https://raw.githubusercontent.com/chiakge/Linux-NetSpeed/master/tcp.sh"
chmod +x tcp.sh
./tcp.sh

#选择安装bbr或者bbrplus或者其他 安装完内核会要求重启。重启完成后 运行
./tcp.sh


# 2、加入在线监控

wget -N --no-check-certificate https://raw.githubusercontent.com/ToyoDAdoubiBackup/doubi/master/status.sh && chmod +x status.sh

#显示客户端管理菜单
bash status.sh c
 
#显示服务端管理菜单
bash status.sh s


# 3、安装人人影视

#CentOS 7系统
yum install make wget crontabs -y

#下载
wget --no-check-certificate https://dl.ilvruan.com/rrshareweb_centos7.tar.gz

#解压CentOS 7压缩包，这里测试的Debian、Ubuntu都可以使用该包，CentOS 6的没试过
tar -zxvf rrshareweb_centos7.tar.gz
#删除无用文件
rm -rf rrshareweb*.tar.gz rrshareweb_linux.rar WEB*.png

#更改人人影视配置文件 修改默认端口
vim /root/rrshareweb/

{
    "port" : 18888,

    "logpath" : "",
    "logqueit" : false,
    "loglevel" : 1,
    "logpersistday" : 2,
    "defaultsavepath" : "/root/file"
}

#CentOS 7系统 防火墙
#安装防火墙
yum install firewalld
#启动防火墙
systemctl start firewalld
#将端口加入防火墙
firewall-cmd --zone=public --add-port=18888/tcp --permanent
#重新加载防火墙
firewall-cmd --reload

# 4、将 人人加入 开机自启
#以下是一整条命令，一起复制到SSH客户端运行
cat > /etc/systemd/system/renren.service <<EOF
[Unit]
Description=RenRen server
After=network.target
Wants=network.target

[Service]
Type=simple
PIDFile=/var/run/renren.pid
ExecStart=/root/rrshareweb/rrshareweb
RestartPreventExitStatus=23
Restart=always
User=root

[Install]
WantedBy=multi-user.target
EOF


# 5、人人影视冷门自动任务
yum install git -y
git clone https://github.com/rushiwo/rrysautotask.git
curl -o- https://raw.githubusercontent.com/creationix/nvm/v0.34.0/install.sh | bash
source .bashrc
nvm install node
node -v //查看版本是否v11
cd rrysautotask
npm install -g /root/rrysautotask
crontab -e
*/1 * * * * /root/.nvm/versions/node/v11.10.0/bin/autotask -c /root/rrysautotask/config.json>/root/rrysautotask/rrysautotask.log 2>&1&
按CTRL+O保存，按CTRL+X退出。

vim /root/rrysautotask/config.json

[{
    "ip":"54.39.**.**",
    "port":**,
    "username":"**",
    "passwd":"***",
    "unlock_passwd":"**"
}]

ln -s /root/.nvm/versions/node/v11.10.0/bin/node /usr/bin/node

# 6、开始启动人人：
systemctl start renren

#从新加载任务
systemctl daemon-reload

#查看状态：
systemctl status renren
如果显示active(running)即开启成功。

#设置开机自启：
systemctl enable renren
