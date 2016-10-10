# asterisk-tutorial
Based on pascom asterisk tutorial (short text version)

## 01-03. Preparing environment.
Switch to superuser or use _sudo_.
First of all, you need to install required packages for asterisk compilation:
```bash
$ apt-get install -y build-essential wget libssl-dev libncurses5-dev libnewt-dev libxml2-dev \
linux-headers-$(uname -r) libsqlite3-dev uuid-dev libjansson-dev
```
build-essential package contains tool for compilation - gcc, make etc.
Now go to [www.asterisk.org](http://www.asterisk.org/downloads) and download latest stable version of asterisk:
```bash
$ wget http://downloads.asterisk.org/pub/telephony/asterisk/asterisk-13-current.tar.gz
```
Unpack, configure, compile and install asterisk on your system:
```bash
$ tar zxvf asterisk-13-current.tar.gz
$ cd asterisk-13/
$ ./configure			# check environment
$ make menuconfig   	# optional
$ make	 				# compile the asterisk server
$ make install	 		# just copy compiled files to the linux system
$ make samples 			# create sample config files
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
## 04. Install and configure udhcp and ntp.
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

## 05. SIP phone peers.
All sip peers configuration done in /etc/asterisk/sip.conf
(Open required ports on host/server machine!)
```bash
$ cd /etc/asterisk
$ cp sip.conf sip.conf.sample
$ vi /etc/asterisk/sip.conf
---
# delete all comments and then blank lines
:g/^\s*;/d
:g/^\s*$/d

# add new peers
[general]
qualify=yes				# connection's quality
[james]
	type=friend			# friend means sending and receiving calls
	context=phones		# dialplan context
	allow=ulaw,alow		# allow some codecs - method in which asterisk convert a speech
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

## 06-08. Dialplan intro.
asteris orginized in contexes.
Every call going inside/outside manages by dialplan.
Dialplan is a configuration file. asterisk reads lines one by one.
Dialplan defined in /etc/extensions.conf file. Let's make some simple changes to it:
```bash
$ cd /etc/asterisk
$ cp extensions.conf extensions.conf.sample
$ echo "" > extensions.conf
$ vim extensions.conf
----
[phones]				# defining a context 
exten => 100,1,NoOp(First Line)		# print the first line
exten => 100,2,NoOp(Second Line)	# print the second line
exten => 100,3,Dial(SIP/james)		# Dial(Technology/resouce), dial james 
exten => 100,4,Hangup			
# NoOp(), Dial(), Hangup - applications

$ asrerisk -r
asterisk> dialplan reload		

# we can get app info via asterisk cli
asterisk> core show applications			# show all apps
asterisk> core show applications like dial	
asterisk> core show applications describing dial
asterisk> core show application dial			# get manual for dial app
```
Now, we dial to james and got error - unable to re-open file /var/log/asterisk//cdr-csv//Master.cs .
We have create log file to fix it:
```bash
$ touch /var/log/astefrisk//cdr-csv//Master.cs
$ chown asterisk /var/log/asterisk//cdr-csv//Master.cs
```

There're some trick to make our life simpler:
```bash
exten => 100,1,NoOp(First Line)	  -->   exten => 100,1,NoOp(First Line)
exten => 100,2,NoOp(Second Line)  -->   exten => 100,n,NoOp(Second Line)	
exten => 100,3,Dial(SIP/james)	  -->   exten => 100,n,Dial(SIP/james)	
exten => 100,4,Hangup		  -->   exten => 100,n,Hangup
```
n - next.
And more... You can say "same", if you are in the same extension:

exten => 100,1,NoOp(First Line)   -->   exten => 100,1,NoOp(First Line)
exten => 100,n,NoOp(Second Line)  -->   same => n,NoOp(Second Line)
exten => 100,n,Dial(SIP/james)    -->   same => n,Dial(SIP/james)
exten => 100,n,Hangup		  -->   same => n,Hangup
```
Finaly our dialplan looks pretty..
```bash
[phones]
exten => 100,1,NoOp(First Line)
same => n,NoOp(Second Line)
same => n,Dial(SIP/james)		
same => n,Hangup

exten => 200,1,NoOp(First Line)
same => n,NoOp(Second Line)
same => n,Dial(SIP/mathias)		
same => n,Hangup
```
When we try call to james, get another error - exited non-zero on 'SIP/james-00000011'.
(not imported module yet)

## 09. File playback.
Test playback files stored in /var/lib/asterisk/sounds/ folder. Let's check it:
```bash
$ ls /var/lib/asterisk/sounds/en/tt_* 
```
To set up custom sound that caller will hear while ringing, asterisk use Playback(_filename_) app. Don't try to provide file extention, aster choose it automatically.
```bash
exten => 200,1,NoOp(First Line)
same => n,NoOp(Second Line)
+same => n, Playback(tt-monkeys)
same => n,Dial(SIP/mathias)		
same => n,Hangup
```

## 10. Incoming calls simulation.
Modify our dialplan (extensions.conf):
```bash
[phones]
exten => 100,1,NoOp(Call for James)
same => n,Dial(SIP/james)		
same => n,Hangup

exten => 200,1,NoOp(Call for Mathias)
same => n,Dial(SIP/mathias)		
same => n,Hangup
```
To simulate outside peer we have to create new peer in /etc/asterisk/sip.conf:
```bash
[outside]
	type=friend
	context=incoming
	allow=ulaw,alaw
	secret=12345678
	host=dynamic
```
Reload dialplan and try to call from outside.
To route call from outsite to local peer we have to make some changes in dialplan.
/etc/asterisk/extensions.conf
```bash
[incoming]

exten => 991123123,1,Goto(phones,100,1)
```
External call goes to context incoming, searches for 91123100 and than go to context phones and dial exten 100 with priority 1.



