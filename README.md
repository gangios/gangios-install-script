gangios-install-script
======================

李阳  关于配置文件  下面的主要流程中  蓝色的为我已经掌握配置的步骤，红色的为尚未开始配置的步骤。希望你能把红色部分写具体详细些  具体怎么配。我会努力实现。
另外 在实现的蓝色部分 在最后有一个nrpe插件装不上 也没有RPM包 应该怎么解决？以下是init.sh脚本中的一段内容。
```bash
if [ "$distributor_id" != "Asianux" -a ! -x /etc/init.d/gmond ]; then
  echo -ne "  * Installing gmond service\t\t"
  yum install ganglia-gmond -y > /dev/null 2>&1
  yum install ganglia-gmond-modules-python -y > /dev/null 2>&1
  chkconfig --add gmond
  chkconfig gmond on
  if [ $? -eq 0 ]; then echo "[OK]"; else echo "[Failed]"; fi
fi

if [ "$distributor_id" != "Asianux" -a ! -x /etc/init.d/nrpe ]; then
  echo -ne "  * Installing nrpe service\t\t"
  yum install nrpe -y > /dev/null 2>&1
  chkconfig --add nrpe
  chkconfig nrpe on
  if [ $? -eq 0 ]; then echo "[OK]"; else echo "[Failed]"; fi
fi
```

已实现
```bash

#!/bin/sh
$1

setenforece 0
sed -i /etc/selinux/config -e s/SELINUX=.*/SELINUX=disabled/g
service iptables stop
chkconfig iptables off

mkdir -p /var/www/html/CentOS5.5
mount /dev/cdrom /var/www/html/CentOS5.5

#====================== yum
setenforece 0
sed -i /etc/selinux/config -e s/SELINUX=.*/SELINUX=disabled/g
service iptables stop
chkconfig iptables off

cd /root/yumserver
sh base_install.sh http
mount /dev/cdrom /var/www/html/yum/CentOS5.5
cp /root/RPMS /var/www/html/yum/gangios

#read -p "please input a ipadress" ipadress

vi /var/www/html/yum/bin/run-centos.repo

#====================== edit
#[CentOS5.5]
#name=CentOS 5.5 Repo
#baseurl=http://192.168.40.131/yum/CentOS5.5/
#gpgcheck=0
#enabled=1

#[gangios]
#name=Gangios
#baseurl=http://192.168.40.131/yum/gangios/
#gpgcheck=0
#enabled=1
#============================


sed -i '$a[gangios]\nname=Gangios\nbaseurl=http://192.168.40.131/yum/gangios/\ngpgcheck=0\nenabled=1' /var/www/html/yum/bin/run-centos.repo 
sed -i /var/www/html/yum/bin/run-centos.repo -e 's/server/'$1'/g'



#====================== init ganglia gmond
setenforece 0
sed -i /etc/selinux/config -e s/SELINUX=.*/SELINUX=disabled/g
service iptables stop
chkconfig iptables off

rm /etc/yum.repos.d/*
curl -s http://192.168.40.131/yum/bin/run-centos.repo > /etc/yum.repos.d/run-centos.repo
yum clean all

yum install -y ganglia-gmond
#====================== edit!!!!!!
vi /etc/ganglia/gmond.conf
#udp_send_channel {
  #bind_hostname = yes # Highly recommended, soon to be default.
                       # This option tells gmond to use a source address
                       # that resolves to the machine's hostname.  Without
                       # this, the metrics may appear to come from any
                       # interface and the DNS names associated with
                       # those IPs will be used to create the RRDs.
#  mcast_join = 239.2.11.71
#  host = 192.168.40.131
 # port = 8649
 # ttl = 1
#}

#udp_recv_channel {
#  mcast_join = 239.2.11.71
 # port = 8649
#  bind = 239.2.11.71
#  retry_bind = true
#}

sed -i '/bind_hostname/s/^/#/g' /etc/ganglia/gmond.conf
sed -i '/bind_hostname/i host = 1.1.1.1' /etc/ganglia/gmond.conf 
sed -i '/mcast_join/s/^/#/g' /etc/ganglia/gmond.conf
sed -i '/retry_bind/i port =8888' /etc/ganglia/gmond.conf
sed -i '/bind /s/^/#/' /etc/ganglia/gmond.conf
```

未实现
```bash
#syslog udp 发送日志 配置
vi /etc/syslog.conf

#===========================
service gmond restart
chkconfig --add gmond
chkconfig --level 2345 gmond on

#=================================== ganglia gmetad
yum install ganglia-gmetad ganglia-gweb

service gmetad restart
chkconfig --add gmetad
chkconfig --level 2345 gmetad on

#=================================== nagios
yum install nagios nagios-plugins
cd /root/gangios/bin/nagios
#ruby make_config.rb (??)


#=================================== log
yum install logsystem

#=================================== thin
yum install gangios
vi /etc/thin/gangios.yml

#===
chdir: /root/gangios
#===


service thin restart
chkconfig --add thin
chkconfig --level 2345 thin on
#===
```
