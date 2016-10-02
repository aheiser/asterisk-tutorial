# asterisk-tutorial
Based on pascom asterisk tutorial (short text version)


## 01-03. Preparing environment
Swith to the superuser or use _sudo_
First of all, you need install required packages for asterisk installation:
```
apt-get install -y build-essential wget libssl-dev libncurses5-dev libnewt-dev libxml2-dev \
linux-headers-$(uname -r) libsqlite3-dev uuid-dev libjansson-dev
```
build-essential package contains tool for compilation - gcc, make etc.
Now go to [www.asterisk.org](http://www.asterisk.org/downloads) and download latest stable version of asterisk
```
wget http://downloads.asterisk.org/pub/telephony/asterisk/asterisk-13-current.tar.gz
```
Unpack, configure, compile and install asterisk on your system:
```bash
tar zxvf asterisk-13-current.tar.gz
cd asterisk-13/
# less README - read README :)
./configure		# check environment
# make menuconfig   - optional
make 			# compile the asterisk server
make install 		# just copy compiled files to the linux system
make samples 		# create sample config files
```
Start asterisk console
```
asterisk> asterisk -cvvv     
asterisk> sip show peers
asterisk> exit
```
c - start asterisk and open CLI,v - verbose

Configure startup script:
```
cd ~/asterisk-13.11.2/contrib/init.d/
cp rc.debian.asterisk /etc/init.d/asterisk
vim /etc/init.d/asterisk
...
DAEMON=/usr/sbin/asterisk
ASTVARRUNDIR=/var/run/asterisk
ASTETCDIR=/etc/asterisk		
...
type asterisk			# check destination of the binary file
/etc/init.d/asterisk start
```
Create asterix user, configure his permisions and restart service
```
/etc/init.d/asterisk stop
useradd -d /var/lib/asterisk asterisk
chown -R asterisk /var/spool/asterisk /var/lib/asterisk /var/run/asterisk /etc/asterisk/
cp ~/asterisk-13.11.2/contrib/init.d/etc_default_asterisk /etc/default/asterisk
vim /etc/default/asterisk
...
AST_USER="asterisk"
AST_GROUP="asterisk"	
...
update-rc.d asterisk defaults
/etc/init.d/asterisk start
ps aux | grep asterisk		
```
Optional changes:
```
/etc/asterisk/asterisk.config
...
live_dangerously = no	
...
```
## 04. Install and configure udhcp and ntp
Configure network interface with static ip:
```
vi /etc/network/interfaces
...
iface etho0 inet dhcp
iface etho0 inet static
	address 192.168.33.10
	netmask 255.255.255.0
	gateway 192.168.33.1
...
vim /etc/resolv.con
...
nameserver 8.8.8.8
...
service networking restart

```
Install udhcp and npt packages
```
apt-get install -y dhcpd ntp
vi /etc/default/udhcp
---
-DHCPD_ENABLED="no"
+DHCPD_ENABLED="yes"
---

# configure dhcpd
vi /etc/udhcp.conf
--- set up proper start/end addresses
start		192.168.33.100
end		192.168.33.150

opt	dns 	8.8.8.8
opt	router  192.168.33.1
#pt	wins
#opt    2nd dns
---

/etc/init.d/udhspd start

## set up time
date
/etc/init.d/ntp stop
ntpdate pool.ntp.org	# update the time
/etc/init.d/ntp start
```
