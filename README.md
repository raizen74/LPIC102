LPIC102-500 EXAM NOTES - David Galera, February 2025

- [USEFUL COMMANDS](#useful-commands)
- [105.1](#1051)
- [MANAGING USERS](#managing-users)
- [SCHEDULING JOBS](#scheduling-jobs)
- [SYSTEMD TIMERS](#systemd-timers)
- [TIME](#time)
- [NTP](#ntp)
- [CHRONY 5-steps](#chrony-5-steps)
- [LOCALE](#locale)
- [EMAIL](#email)
- [PRINTERS](#printers)
- [SECURITY](#security)
- [PORTS](#ports)
- [ENCRYPTION](#encryption)
- [OPEN SSH](#open-ssh)
- [WINDOWS](#windows)
- [NETWORKING](#networking)
- [IP COMMAND](#ip-command)
- [NETWORK MANAGER](#network-manager)
- [DNS Client](#dns-client)
- [LEGACY NET-TOOLS (DEPRECATED by iproute2)](#legacy-net-tools-deprecated-by-iproute2)
- [EXAM](#exam)

## USEFUL COMMANDS
- `sudo find /etc -name "ntp.conf" -exec ls -l {} ";"`
- `echo $PATH | tr : "\n"`
- `systemctl list-units --state active --type service`
- `ntpdate pool.ntp.org` Refresh system clock
- `shutdown -h now` Halt the system
- `gpasswd -a user devs` adds user to devs group

## 105.1
`set` displays all environment variables, user variables and functions

`printenv` displays environment vars, not user vars

`env` same as printenv

`$-` Shell options

`set -a` activates a

`set +a` deactivates a

## MANAGING USERS
`newgrp sudo` starts a subshell with sudo as primary group

`/etc/default/useradd` config file that determines account items
`/etc/login.defs`

`/etc/skel/` new users skeleton

`/bin/nologin` or `/bin/false` in passwd file -> user is not allowed to login

`usermod -c "GECOS" David` modifies the GECOS field in passwd

`userdel -r <user>` deletes user and $HOME dir

`sudo passwd -S root` see account info

`sudo passwd -l david` lock the account

`sudo passwd -u David` unlock

`sudo chage -l David` see account info in human format

`sudo chage David` change account password settings

`passwd David` set a password for David

`getent group 1000` see group from /etc/group

`getent group David`

`groupmod -m newname oldname` change group name	

`id -gn` view primary group of a user

## SCHEDULING JOBS
`atq` Jobs scheduled, same as at-l

`atrm` job remove job

`/etc/at.allow` if a user is here, `/etc/at.deny` is not checked. Same for `/etc/cron.allow`

`/var/spool/cron` contains cron and at jobs

`crontab -r` remove my crontab table

`crontab -e` edit crontab table

`/etc/crontab` system crontab conf 

**System crontab table** contains an extra field before the command: Account

`/etc/cron.d` files in this dir are treated as individual crontab entry

anacron assumes that the system DOES NOT run continuosly `/etc/anacrontab` `/var/spool/anacron` format -> Period, Delay, Job Identifier, Command

While anacron can handle daily, monthly and yearly jobs, **it does not handle jobs that need to run hourly**

## SYSTEMD TIMERS
`systemctl list-timers --no-pager` list timer units

`systemctl cat <unit>` view file, if [Service] section is not in it, the **service unit file** has the same name + .service i.e. <unit>.service -> Location `/etc/systemd/system`

Output from transient timers go to the journal

`sudo systemd-run --on-calendar="*-*-* 15:00:00" COMMAND` # Schedule a transient timer job

`systemd daemon-reload`

The `OnBootSec` setting is used in **monotonic timer unit files (timers)** to designate the time to run the job after the system boots. The other setting used is `OnUnitActiveSec`, which designates the time to **run the job after the last time it ran**

## TIME
`sudo hwclock -r` display hw clock

`sudo hwclock --adjust` tune hwclock manually

`sudo hwclock --systohc` set the **hardware (real time) clock's time** to the software clock's time. You can also use the hwclock -w command `sudo hwclock -w`

`sudo hwclock --hctosys` sets the **software clock's time** to the hardware clock's time (modifies system clock) or -s option

`timedatectl set-time` 

`/etc/localtime` /usr/share/zoneinfo/Europe/Madrid (binary)

`/etc/timezone` plaintext file

`tzselect` utility for selecting timezone

## NTP
`/etc/ntp.conf` 

`pool 0.ubuntu.pool.ntp.org iburst` iburst causes initial time corrections to occur more quickly

`server ntp.ubuntu.com`

`ntpdate pool.ntp.org` update system clock

`ntpstat` display synchronization status

`ntpq -p` display ntp pool time servers you have chosen in ntp.conf against yours

## CHRONY 5-steps
`/etc/chrony/chrony.conf`

`systemctl status chrony`

`chronyc tracking` displays stratus, offset and more info

`chronyc sourcestatus -v` system default pools info

## LOCALE
`locale -a` display all locales available in the system

`/etc/default/locale` or `/etc/locale.conf`

`locale -m` list all system character sets

to modify any locale variable, override it with environment var

special locale argument: C or POSIX

## EMAIL
add mydomain = ASUS-David to `/etc/postfix/main.cf`

`sendmail -bp`, `mailq`, `postqueue` -p -> View email message queues

Automatically forward messages to another user -> create `~/.forward` file add all the users to forward

`/etc/aliases` -> file to add/remove aliases for email users

`newaliases` and `sendmail -I` with sudo to update the aliases.db

`mail -s "subject" username` send an email

## PRINTERS
CUPS -> Common Unix Printing System, supports Internet Printing Protocol (IPP) and Line Printer Daemon Protocol (LPD , Legacy)

`/etc/cups` dir contains `cupsd.conf` (Daemon config file) and `printers.conf` (printers configured in this system)

`lp -d <printername> <file>`, `lpr -P <printername> <file>` -> Send file to printer

`cancel`, `lprm` -> Remove print job

To configure a printer you can Access a web browser on IP:631 or use `lpadmin` command or edit printers.conf

`lpstat`, `lpq -P` -> View printer queue, current jobs

`lpstat -a <printer name>` show **printer queue** status to see if it is accepting jobs

`lpstat -p <printer name>` show **printer** status (disabled | enabled)

`cupsenable <printer name>` enables a physical printer fed by a CUPS printer queue; not the printer queue.

`cupsdisable`

`cupsaccept <printer name>` enables a CUPS printer queue,
/etc/printcap is a file kept for backward compatibility with LPD

`lpadmin -x UdemyQueue` command will **remove the UdemyQueue printer queue's configuration**

`lprm -P` older command

`cupsctl` utility allows you to check (and modify) the CUPS daemon's configuration

## SECURITY
Memorize the facility numbers for each service

`lastb` # login failures

`sudo find /usr/bin -perm -6000` SUID AND GUID

`sudo find /usr/bin -perm -u=s,g=s` equivalent

`sudo find /usr/bin -perm /6000` SUID OR GUID, -type f to search only files

`%sudo   ALL=(ALL:ALL) NOPASSWD:ALL` all hosts (ALL=) belonging to sudo group (%sudo) can execute all commands (NOPASSWD:ALL) without typing password, as any user or group (ALL:ALL)

`ulimit -a` account soft limits

`ulimit -ah` account hard limits

## PORTS
`nmap -sT <IP>` Scan TCP open ports

`nmap -sU <IP>` Scan UDP open ports

`nmap -sT -6 ::1` Scan IPv6 open TCP ports on host

`netstat -lptn` l displays programs listening on ports, p provides program name(sudo), t for only tcp, n numeric port

`lsof -i4UDP` same

`fuser -v 22/tcp`

`fuser -vn tcp 22` ports opened

`ss -lt`

`systemctl list-unit-files --type=service` services can be enabled, disabled or static (cannot be disabled)

`chkconfig --level 0123456 <service name> off` In SysVinit turns off a service at a specific runlevel

## ENCRYPTION
`gpg --gen-key` utility to create a key pair of asymmetric keys

`gpg --export -a <owner name> > filename.key` exports public key into a text file, GnuPG stores a user's keys, called a key ring.

`gpg --import <public.key>` imports a key from the public key file of another user

`gpg --list-keys`

`gpg --recipient <owner name> --out <fileout> --encrypt <filein>` encrypts filein with owner key, results in fileout

`gpg --out <response> --decrypt <fileout>`

## OPEN SSH
`scp <file> user@remotehost:<dir>` copy file to a remote host dir

`ssh user@remotehost 'cat helloworld >> .ssh/authorized_keys'` issue a command to a remote host

`ssh-keygen -t dsa -t <hostkeyname>` generate ssh keys

`ssh-copy-id -i <key path> user@remotehost` adds the key pair to ~/.ssh/authorized_keys file in the remote host, passphrase is not required

`ssh-agent /bin/bash` execute the agent

`ssh-add <key path>` adds key to the agent, passphrase key required. While the agent is running you can ssh into remote machine

`ssh -X user@remotehost` establish a remote tunnel for graphic application output. The remote system must have 'X11Forwarding yes' in its OpenSSH configuration file, which is typically the /etc/ssh/sshd_config file

## WINDOWS
`/etc/X11/xorg.conf.d`

After changing any of the configuration files you have to restart the service:
- `service <displaymanagername> restart` SysVinit
- `systemctl restart display-manager` Systemd

`echo $DISPLAY` localhost:10.0

`xhost +192.168.30.42` command will add the 192.168.30.42 address to an access control list (ACL) in order to allow this remote system's X11 display server to send its graphical utility output to the system on which the command was issued

**XDMCP** is a protocol for the desktop environment's graphical login window

**GDM** is the display manager software for the GNOME desktop environment

The **xsession-errors**, which is actually a hidden file in a user's home directory (~/.xsession-errors), contains display manager errors
xauth utility displays the authorization information used when connecting to a remote X11 server

**Modules section** in an X11 configuration handles which device drivers to load

## NETWORKING
- /etc/sysconfig/network-scripts/
- /etc/network/interfaces
- /etc/netplan/
- /etc/sysconfig/network
- **Broadcasting**: Send a packet to ALL devices in the network
- **Multicasting**: Send a packet to MULTIPLE devices in the network
- 10.0.0.0/8
- 172.16.0.0/12
- 192.168.0.0/16

169.254.164.23 address shows that the DHCP server could not be reached, and an APIPA address was assigned instead. APIPA addresses for IPv4 fall into the range of **169.254.0.0 to 169.254.255.255**

- `ifconfig eth0` Also shows packet info
- `ip a show eth0`

`/etc/services` -> well known ports documentation
- Port 20 # FTP data
- Port 21 # FTP command
- Port 23 # Telnet
- Port 25 # SMTP
- 110 # POP3
- 514 UDP # syslog

## IP COMMAND
ip command objects:
- **address** show or modifies network interface addresses
- **link** displays or controls network interfaces
- **route** shows or modifies the kernel routing tables (built at boot time)
  
- `ip -br addr show`
- `ip -br a show eth0` displays only eth0 interface
- `ip -s a show eth0` displays interface statistics
- `ip -br link show` displays MAC addr for interfaces
- `ip route` route table, default -> Gateway
- `ip a add <CIDR> dev eth0` add a new ip to the network interface
- `ip a del <CIDR> dev eth0` del a new ip in the network interface
- `sudo ip link set eth0 down` brings the interface down
- `sudo ip link set eth0 up` brings the interface up
- `sudo ip route delete default` delete default Gateway
- `sudo ip route add default via 172.30.208.1` adds a route to default gateway (172.30.208.1)

## NETWORK MANAGER
- `nmcli general` status
- `nmcli gen` same
- `nmcli -p conn show` show connection profiles
- `nmcli d show` NICs info
- `nmcli conn modify` add a new ip

## DNS Client
- `hostnamectl` displays hostname
- `hostname set-hostname <new>` change hostname
- `/etc/resolv.conf` contains the ip of the name server, you can have up to 3 ips in `/etc/resolv.conf`
- `/etc/nsswitch.conf` defines how queries (lookup order) are handled e.g. hosts:	files dns
- `systemd-resolved` -> dns client, etc/systemd/resolved.conf # daemon configuration file
- `systemd-resolve www.google.com` shows time resolution
- `systemd-resolve --statistics`

## LEGACY NET-TOOLS (DEPRECATED by iproute2)
- `ifconfig`
- `route` similar to `netstat -r`
- `netstat -r` displays routing tables, replaced by ip route
- `netstat -a` displays NIC ports and UNIX sockets opened
- `netstat -s` displays NIC statistics (packets send/received by protocol), replaced by nc
- `netstat -i` kernel interface table, packet info
- `arp` replaced by ip neigh
- `nc -l 8001` netcat, listen traffic (l) port8001
- `ss -s` socket statistics
- `traceroute -nI <IP>` dont display host
- `tracepath` main difference is that displays mtu (max transmisión unit)
- `hostname <newname>` changes hostname
- `getent hosts` displays `/etc/hosts`
- `host -a www.google.com` DNS resolution, queries dns servers to get IP. -a provides resolution time
- `dig @1.1.1.1 www.lpi.org` DNS resolution, querying 1.1.1.1 name server

## EXAM 
`sudo find /etc -name "locale" -print`

`useradd -m` # creates a new user's home dir and provisions with skel files

`SPICE` proto Access video card output of a VM

`NetworkManager` can be configured to use the distro NIC config and by default does not change config of NI already configured

display managers like `XDM` and `KMD` -> Handle user login and prepares desktop environment for the user

`XDM` is the default graphical login manager that comes with vanilla X11. Default wallpaper -> `/etc/X11/xdm/Xsetup` 

content in `xorg.conf` (X11 config file) in the section "SectionName" is placed between a line cointaining Section "SectionName" and a line containing EndSection

`xwininfo` shows current color Depth of the X Server, investigates properties for a particular window in X

`.bashrc` and `.bash_pro` affect the behavior of the Bash Shell

`env -u FOO./myscript` suppresses FOO environment var for the execution of myscript

`ssh-agent` preloads and manages keys

The default route is only used if there is not a more specific route to a destination host or network

`GOK` -> on-screen keyboard

`ssh -p 2222 example.com`, `ssh -o Port=2222 example.com` connect to a specific ssh port (e.g. 2222)

SPICE:
- Lets you connect local USB into a remote app
- Downloads and installs local apps from a remote machine

`export VARIABLE` make it available to subshells

/`etc/xinetd.conf` enable/disable network services

display NIC eth0 bytes transmitted -> `ifconfig eth0` and `ip -s link show eth0`

if `cron.allow` and `cron.deny` do not exist the behavior depends on the distro. On most distros only root user can create cron Jobs, in some distros all users can create cron Jobs.

- `test -z` # test if a variable is empty (true)
- `test -n` # test if there is content in the string or string variable
- `test -x` # test if file is executable by user

`hostnamectl set-hostname` modern systems

`hostname newname` old systems

nmcli network connectivity: none, portal, limited, full, unknown

`traceroute` sends **UDP** packets

`ssh -L 4672:www.example.com:80` localhost forward local port 4672

`lprm 5` removes job 5, the printer name is not required
