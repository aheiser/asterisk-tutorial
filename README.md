# asterisk-tutorial
Based on pascom asterisk tutorial (short text version)


## 01-03. Preparing environment
Switch to superuser or use _sudo_.
First of all, you need to install required packages for asterisk compilation:
```bash
apt-get install -y build-essential wget libssl-dev libncurses5-dev libnewt-dev libxml2-dev \
linux-headers-$(uname -r) libsqlite3-dev uuid-dev libjansson-dev
```
build-essential package contains tool for compilation - gcc, make etc.
Now go to [www.asterisk.org](http://www.asterisk.org/downloads) and download latest stable version of asterisk:
```bash
wget http://downloads.asterisk.org/pub/telephony/asterisk/asterisk-13-current.tar.gz
```
Unpack, configure, compile and install asterisk on your system:
```bash
$ tar zxvf asterisk-13-current.tar.gz
$ cd asterisk-13/
$ ./configure			# check environment
$ make menuconfig   	# optional
$ make 				# compile the asterisk server
$ make install 		# just copy compiled files to the linux system
$ make samples 		# create sample config files
```
Start asterisk console:
```bash
$ asterisk -cvvv     
asterisk> sip show peers
asterisk> exit
```
c - start asterisk and open CLI,v - verbose

Configure startup script:
```bash
$ cd ~/asterisk-13.11.2/contrib/init.d/
$ cp rc.debian.asterisk /etc/init.d/asterisk
$ vim /etc/init.d/asterisk
...
DAEMON=/usr/sbin/asterisk
ASTVARRUNDIR=/var/run/asterisk
ASTETCDIR=/etc/asterisk		
...
$ type asterisk			# check destination of the binary file
$ /etc/init.d/asterisk start
```
Create asterix user, set up his permisions and restart service:
```bash
$ /etc/init.d/asterisk stop
$ useradd -d /var/lib/asterisk asterisk
$ chown -R asterisk /var/spool/asterisk /var/lib/asterisk /var/run/asterisk /etc/asterisk/
$ cp ~/asterisk-13.11.2/contrib/init.d/etc_default_asterisk /etc/default/asterisk
$ vim /etc/default/asterisk
...
AST_USER="asterisk"
AST_GROUP="asterisk"	
...
$ update-rc.d asterisk defaults
$ /etc/init.d/asterisk start
$ ps aux | grep asterisk		
```
Optional changes:
```bash
$ vim /etc/asterisk/asterisk.config
...
live_dangerously = no	
...
```
## 04. Install and configure udhcp and ntp
Configure network interface with static ip:
```bash
$ vim /etc/network/interfaces
...
iface etho0 inet dhcp
iface etho0 inet static
	address 192.168.33.10
	netmask 255.255.255.0
	gateway 192.168.33.1
...
$ vim /etc/resolv.con
...
nameserver 8.8.8.8
...
$ service networking restart

```
Install udhcp and npt packages:
```bash
$ apt-get install -y dhcpd ntp
$ vim /etc/default/udhcp
---
DHCPD_ENABLED="yes"
---
```

Configure dhcpd:
```bash
$ vim /etc/udhcp.conf
--- set up proper start/end addresses
start		192.168.33.100
end		192.168.33.150

opt	dns 	8.8.8.8
opt	router  192.168.33.1
#pt	wins
#opt    2nd dns
---
$ vim /etc/init.d/udhspd start
```
Update time using ntp:
```bash
$ date
$ /etc/init.d/ntp stop
$ ntpdate pool.ntp.org	# update the time
$ /etc/init.d/ntp start
```

## 05. SIP phone peers
All sip peers configuration done in /etc/asterisk/sip.conf
(Open required ports on host/server machine!)
```bash
$ cd /etc/asterisk
$ cp sip.conf sip.conf.orig
$ vi /etc/asterisk/sip.conf
---
# delete all comments and then blank lines
:g/^\s*;/d
:g/^\s*$/d

# add new peers
[general]
qualify=yes		# connection's quality
[james]
	type=friend	# friend means sending and receiving calls
	context=phones	# dialplan context
	allow=ulaw,alow	# allow some codecs - method in which asterisk convert a speech
	secret=12345678
	host=dynamic
[mathias]
	type=friend	
	context=phones	
	allow=ulaw,alow	
	secret=12345678
	host=dynamic
---

# go to asterisk CLI, update config and check status
$ asterisk -rvvv
asterisk> sip show peers
asterisk> sip reload
asterisk> sip show peers
```



