# asterisk-tutorial
Based on pascom asterisk tutorial (short text version)


## 01-03. Preparing environment
Swith to the superuser or use _sudo_.
First of all, you need install required packages for asterisk installation:
```
apt-get install build-essential wget libssl-dev libncurses5-dev libnewt-dev libxml2-dev linux-headers-$(uname -r) libsqlite3-dev uuid-dev libjansson-dev
```
build-essential package contains tool for compilation - gcc, make etc.
Now go to [www.asterisk.org](http://www.asterisk.org/downloads)and download latest stable version of asterisk
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


