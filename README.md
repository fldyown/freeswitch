## FreeSWITCH 1.6
### 系统安装
	选择debian 8
	将/etc/ssh/sshd_config 中的"PasswordAuthentication"参数值为"no",修改回"yes"，
	重启sshd服务即可
	#RHEL/CentOS系统
	$ service sshd restart
	#ubuntu系统
	$ service ssh restart
	#debian系统
	$ /etc/init.d/ssh restart
	
### 阿里云服务器安装FreeSWITCH 1.6
	1.添加源和更新源
	wget -O - https://files.freeswitch.org/repo/deb/debian/freeswitch_archive_g0.pub | apt-key add -
 
	echo "deb http://files.freeswitch.org/repo/deb/freeswitch-1.6/ jessie main" > /etc/apt/sources.list.d/freeswitch.list
 
	# you may want to populate /etc/freeswitch at this point.
	# if /etc/freeswitch does not exist, the standard vanilla configuration is deployed
	
	2.更换阿里美国源（国内服务器可以不用换）
	vim /etc/apt/sources.list.d/freeswitch.list
	更改如下：
	deb http://files.freeswitch.org/repo/deb/freeswitch-1.6/ jessie main
	# See http://www.debian.org/releases/stable/i386/release-notes/ch-upgrading.html
	# for how to upgrade to newer versions of the distribution.
	# united states
	deb http://ftp.us.debian.org/debian jessie main contrib non-free
	deb-src http://ftp.us.debian.org/debian jessie main contrib non-free
	备份并删除原来的阿里源
	
	3.安装
	apt-get update && apt-get install -y freeswitch-meta-all
	
	4.FreeSWITCH™ is now installed and can be accessed with
	fs_cli -rRS
	
	5.如果上面一步无法进入控制界面可能是设备不支持IPv6
	找到文件
	event_socket.conf.xml
	/etc/freeswitch/autoload_configs/event_socket.conf.xml
	修改listen-ip内容如下
	<configuration name="event_socket.conf" description="Socket Client">
	  <settings>
	    <param name="nat-map" value="false"/>
	    <param name="listen-ip" value="0.0.0.0"/>
	    <param name="listen-port" value="8021"/>
	    <param name="password" value="ClueCon"/>
	    <!--<param name="apply-inbound-acl" value="loopback.auto"/>-->
	    <!--<param name="stop-on-bind-error" value="true"/>-->
	  </settings>
	</configuration>
	修改完成之后重启即可
	systemctl restart freeswitch.service
	
### 状态查看和启动
	ps -Af | grep freeswitch
	# 立即启动
	systemctl start freeswitch.service
	# 立即停止
	systemctl stop freeswitch.service
	# 重启
	systemctl restart freeswitch.service
	# 可以通过以下命令可以查看日志
	tail -f /var/log/freeswitch/freeswitch.log

### 用户管理
	使用1000.xml的默认用户作为模版创建用户即可
	
	0007-freeswitch_create_and_delete_users.patch


### 错误解决
	1.[ERR] sofia.c:3156 Error Creating SIP UA for profile: 	internal-ipv6 (sip:mod_sofia@[::1]:5060;transport=udp,tcp)

	/etc/freeswitch/sip_profiles# ls
	external  external-ipv6  external-ipv6.xml  external.xml  internal-ipv6.xml  internal.xml
	修改后：
	/etc/freeswitch/sip_profiles# ls
	external  external-ipv6.disable  external-ipv6.xml.disable  external.xml  internal-ipv6.xml.disable  internal.xml
	
	2.默认密码问题
	FS默认认证密码为：1234，可以根据需要进行修改。编辑配置文件：
	/etc/freeswitch/vars.xml
	找到：<X-PRE-PROCESS cmd="set" data="default_password=666666"/> ，将默认的“1234”修改为666666。
	
	3.错误日志查看
	进入控制台
	fs_cli -rRS
	更改日志级别
	sofia loglevel all 9
	
	tail -f /var/log/freeswitch/freeswitch.log
	
	端口使用 netstat -anp|grep 5060
	
### 详细相关配置请查阅以下补丁
	0001-freeswitch_disable_ipv6.patch
	0002-freeswitch_change_default_password.patch
	0003-freeswitch_config_nat.patch
	0004-freeswitch_fs_cli_rRS_Connection_Error.patch
	0005-freeswitch_aliyun_nat_config.patch
	0006-freeswitch_add_H264_VP9_And_other_codes.patch
	0007-freeswitch_create_and_delete_users.patch
