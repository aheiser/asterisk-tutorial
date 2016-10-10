# asterisk-tutorial
Based on pascom asterisk tutorial (short text version)


## 01-03. Preparing environment
Switch to superuser or use _sudo_.
First of all, you need to install required packages for asterisk compilation:
```bash
user-$ apt-get install -y build-essential wget libssl-dev libncurses5-dev libnewt-dev libxml2-dev \
linux-headers-$(uname -r) libsqlite3-dev uuid-dev libjansson-dev
```
build-essential package contains tool for compilation - gcc, make etc.
Now go to [www.asterisk.org](http://www.asterisk.org/downloads) and download latest stable version of asterisk:
```bash
user-$ wget http://downloads.asterisk.org/pub/telephony/asterisk/asterisk-13-current.tar.gz
```
Unpack, configure, compile and install asterisk on your system:
```bash
user-$ tar zxvf asterisk-13-current.tar.gz
user-$ cd asterisk-13/
user-$ ./configure			# check environment
user-$ make menuconfig   	# optional
user-$ make	 				# compile the asterisk server
user-$ make install	 		# just copy compiled files to the linux system
user-$ make samples 			# create sample config files
```
Start asterisk console:
```bash
user-$ asterisk -cvvv     
asterisk> sip show peers
asterisk> exit
```
c - start asterisk and open CLI,v - verbose

Configure startup script:
```bash
user-$ cd ~/asterisk-13.11.2/contrib/init.d/
user-$ cp rc.debian.asterisk /etc/init.d/asterisk
user-$ vim /etc/init.d/asterisk
...
DAEMON=/usr/sbin/asterisk
ASTVARRUNDIR=/var/run/asterisk
ASTETCDIR=/etc/asterisk		
...
user-$ type asterisk			# check destination of the binary file
user-$ /etc/init.d/asterisk start
```
Create asterix user, set up his permisions and restart service:
```bash
user-$ /etc/init.d/asterisk stop
user-$ useradd -d /var/lib/asterisk asterisk
user-$ chown -R asterisk /var/spool/asterisk /var/lib/asterisk /var/run/asterisk /etc/asterisk/
user-$ cp ~/asterisk-13.11.2/contrib/init.d/etc_default_asterisk /etc/default/asterisk
user-$ vim /etc/default/asterisk
...
AST_USER="asterisk"
AST_GROUP="asterisk"	
...
user-$ update-rc.d asterisk defaults
user-$ /etc/init.d/asterisk start
user-$ ps aux | grep asterisk		
```
Optional changes:
```bash
user-$ vim /etc/asterisk/asterisk.config
...
live_dangerously = no	
...
```
## 04. Install and configure udhcp and ntp
Configure network interface with static ip:
```bash
user-$ vim /etc/network/interfaces
...
iface etho0 inet dhcp
iface etho0 inet static
	address 192.168.33.10
	netmask 255.255.255.0
	gateway 192.168.33.1
...
user-$ vim /etc/resolv.con
...
nameserver 8.8.8.8
...
user-$ service networking restart

```
Install udhcp and npt packages:
```bash
user-$ apt-get install -y dhcpd ntp
user-$ vim /etc/default/udhcp
---
DHCPD_ENABLED="yes"
---
```

Configure dhcpd:
```bash
user-$ vim /etc/udhcp.conf
--- set up proper start/end addresses
start		192.168.33.100
end		192.168.33.150

opt	dns 	8.8.8.8
opt	router  192.168.33.1
#pt	wins
#opt    2nd dns
---
user-$ vim /etc/init.d/udhspd start
```
Update time using ntp:
```bash
user-$ date
user-$ /etc/init.d/ntp stop
user-$ ntpdate pool.ntp.org	# update the time
user-$ /etc/init.d/ntp start
```

## 05. SIP phone peers
All sip peers configuration done in /etc/asterisk/sip.conf
(Open required ports on host/server machine!)
```bash
user-$ cd /etc/asterisk
user-$ cp sip.conf sip.conf.orig
user-$ vi /etc/asterisk/sip.conf
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
user-$ asterisk -rvvv
asterisk> sip show peers
asterisk> sip reload
asterisk> sip show peers
```



