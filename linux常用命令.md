1、开放端口

**firewall-cmd --zone=public --add-port=5672/tcp --permanent**  # 开放5672端口

**firewall-cmd --zone=public --remove-port=5672/tcp --permanent** #关闭5672端口

**firewall-cmd --reload**  # 配置立即生效

 

2、查看防火墙所有开放的端口

**firewall-cmd --zone=public --list-ports**

 

3.、关闭防火墙

如果要开放的端口太多，嫌麻烦，可以关闭防火墙，安全性自行评估

**systemctl stop firewalld.service**

 

4、查看防火墙状态

 **systemctl status firewalld.service**

 

5、查看监听的端口

**netstat -lnpt**

6、检查端口被哪个进程占用

**netstat -lnpt |grep 5672**

查看状态

systemctl status firewalld.service

打开防火墙

systemctl start firewalld.service

关闭防火墙

systemctl stop firewalld.service

开启防火墙

systemctl enable firewalld.service

禁用防火墙

systemctl disable firewalld.service
