LPIC102-500 EXAM NOTES - David Galera, February 2025

- [USERFUL COMMANDS](#userful-commands)
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

## USERFUL COMMANDS
- `sudo find /etc -name "ntp.conf" -exec ls -l {} ";"`
- `echo $PATH | tr : "\n"`
- `systemctl list-units --state active --type service`
- `ntpdate pool.ntp.org` Refresh system clock
- `shutdown -h now` Halt the system

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