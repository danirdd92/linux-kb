# Linux KB #1 - Basics

## find help
`man`  
`man -k` w/ grep for relavant man kind (1-9)  
`tldr`  

## fs
man heir (fs heir)

* regex:
cmd a* starts with a and any other charactrs
cmd a? starts with a and any other single char
cmd a[nm]* n | m + any
cmd a[a-e]* a..e + any

`ln` - symbolic/ hard links
every file has an inode (need to check with manw)
symbolic/symlink - plain shortcut used as alias (e.g. /bin -> /usr/bin)
hardlink - actual ref copy of same file (points to same inode)

`find` (just tldr it)
-exec iterator per result
can use multiple execs 
`xargs` to pipe result 

`tar/ zip` - archive and compression


## working with textfiles
`vimtutor`     - vim learning utility great for learning vim  
`less/more`    - file pager, use less, more is defunced   
`tail/head -#` - read last/first # lines of text  
`cat/tac`      - cat and resverse cat  
`grep`         - find text using regex  
`cut`          - `-d` for delimiter; `-f` - field (slice number X)  
`sort`         - sorts by characters, -n for numeric sort  
`tr`           - translates char sets (used to lower/upper case ) `tr from to`  
`awk`          - very powerful programing language focused on string manipulation  
`sed`          - stream editor for string manipulation on iterables  
`diff`         - show diff between two files  

## RegEx
* always inside single quotes
* docs at `man 7 regex`
* extended regex only with -E (grep -E)

- ^x starts with (whole line)
- x$ ends with (whole line)
- x\b end of word
- x* zero or more times
- x+ one or more times (extended)
- x? zero or one time (extended)
- x\{n\} - \ for escaping the braces. repeats x - n times
- . match any one char

## root

- root user exist in kernel space (memory space allocated in RAM when system boots)
- root privillage is usually allevated temporary with sudo (15min by default, but can be configured otherwise)
`su`     - switch user opens a shell as specific user, use `su -` to login with root and jump to its home directory
`visudo` - edits sudoers file with default text editor, makes sure file is valid, needs root premissions of course, and changes done are immediately implemented.

## bash 

### keyboard shortcuts
ctr-l - clear screen
ctr-u - clear current command line
ctr-a - move to start of line
ctr-e - move to end of line
ctr-c - SIGINT current process
ctr-d - SIGKILL current process 
ctrl+shift+c - copy highlited text
ctrl+shift+v - paste text from clipboard (temp memory in RAM for saving highlited data)

### redirection and pipe
```
<     - Standard input (stdin)
>     - Standard output (stdout)
2>    - Standard error (stderr)
>>    - Append stdout to a file
2>>   - Append stderr to a file
&>    - Redirect both stdout and stderr to a file
1>&2  - Redirect stdout to stderr (POSIX compliant)
2>&1  - Redirect stderr to stdout (POSIX compliant)
|     - Pipe output to another command
|&    - Pipe both stdout and stderr to another command (bash)
```

### history
- `history`  
  - -c clear
  -  -w write-to
  -  -d delete specific line
  -  Ctr-r reverse-i-search  

### variables & configurations
* `env` - all env variables
- to create variables with VARNAME=value
- variables exist by default in the current terminal session
- in order to use them in sub-shells use `export`
    - `export VARNAME=value`

* alias - use alias name for a commend to make it easy `pn=pnpm`
usually stored in ~/.bashrc dotfile for presistence

* startup files 
/etc/enviorments contains list of variables initiated when starting bash (can be different in different linux distributions)

/etc/profile is executed while users login
(!) /etc/profile.d is used as a snapin dir that contains additional configs
(!) ~/.bash_profile can be used as user-specific version of profile.d
(!) ~/.bash_logout is run when user logs out (can be used for cleanup scripts)

/etc/bashrc runs everytime a new sub shell is initiated 
(!) ~/.bashrc is a user-specific version of /etc/bashrc

## users, groups and file ownership

* when a user creates a file on linuxm that user becomes the file owner
* every user must belong to at least one group (by default to group under the same name as username)
* most distros are using private groups which means that when a user is created, a new group is created with the same name and the user belongs to it
* on file creation a user owner and a group owner is assigned to the file:
```
PREMISSIONS | LNK |   OWNR       |  GRP  | SIZE |      DT     | NAME   
-rw-rw-r--    1      dani          dani      0    Apr 14 09:36 test
```

`id` - show user information (uid, gid groups)  
`getent`  - -"- w/ passwd | group  
`useradd` - create new user (in ubuntu add `-m` tp add home dir for the new user)  
`usermod` - modify user `-aG <group> <user>` to add user go group (as secondary group) `-g <group> <name>` (as primary group)  
`userdel` - deletes user  
* default settings are listed in `/etc/login.defs` 
* user information is listed in `/etc/passwd` and `/etc/shadow`
* user skelton folder which is a template to how users should be created can be found at `/etc/skel`
  
`groupadd` - create a new group  
`groupmod` - modify group  
`groupdel` - deletes a group  

`vipw` - edit user info with default text editor  
`vigr` - edit group info with default text editor  
* some users do not have access to a shell, instead they just run a cammand from /bin | /sbin 
* when user is created a configs for his home dir is copied from /etc/skel, you can edit it to change defaults

`passwd`   - manage and change password  
`chage`    - passwd related properties settings    
`chpasswd` - can be used to pipeline new crediantials to change password without prompt (for scripting)  
* password hashes stored in /etc/shadow

`loginctl` - systemd login manager, used to manage login sessions
`who`      - show who is logged and some related data  
`w`        - show who is logged and what they are doing  

`chown` - change owner (user)  
`chgrp` - change owner (group)  
`chmod` - change premissions; 2 modes abosulte & relative overrides the need for the mentioned above
 - absolute `chmod 750 filename`
 - relative `chmod +x myscript`
* premissions are bitwise values for read, write, exectute grouped by User.Group.Other
* anything in a user's home dir can be rwx by that user

### advanced premissions - WIP

* set user id (suid) (4) - on files - run the file as the user-owner of that file
* set group id (suig) (2) - on files - run the file as the group-owner of that file, on dirs - sets ownership on newly created items as the group owner of that dir
* sticky bit (1) - on dirs - allows a user to delete files if the user is a file owner or the user is a dir owner

`umask` - provides a value to subtract from the current premissions  and sets those permissins recoursively to the folder strcuture

## storage managment essentials

* iSCSI is a block protocol for storage networking used to provision storage from a SAN (storage area network) to the server

```
/dev/hd[a,b,c..] IDE hard disks (old but still can be found in various places)
/dev/sd[a,b,c..] SCSI hard disks
/dev/vd[a,b,c..] Virtual hard disks
/dev/nvme0n - nvme hard disks
/dev/sr - optical drive
```
`blkid` - list block devices with unique id
`lsblk` - list block devices  
`fdisk` - format disk, used to create MBR disk partitions  
`gdisk` - format disk util, used to create GPT partitions  

* to use a partition a filesystem must be created on top of it
* files systems are mainly used to store filesm but there aree also special purpose file systems that can be created on a partition
* Swapfs os a swap file system
* initramfs is a file system that is written to the Init RAM FS, which is used while booting
* generic file systems are XFS and Ext4

`mkfs.<fs> <part>` - create a file system of fs type (`mkfs.ext4`) on top of the partition

`mount`   - mounts a device `mount /dev/sdb1 <target-location>`
`unmount` - unmounts a device
`df -h`   - presents mounted devices, including available disk space
`findmnt` - shows all mounts nicely 
* mount to /mnt for removeable mounts
* `/media` is used to dynamically mount **usb** devices on most Debian based linux distributions
* `/run/media` is used to dynamically mount **usb** devices on most RedHat based linux distributions
* create a mounting point before mounting, a location must exist to be mounted

## networking

`ifconfig` - is deprecated on most linux distributions (might be found on MacOS but not recommended to use due to security issue - enables peremission escallation)  
`ip` - current way to manage temporary network configuration
    - other ways are:
      - `NetworkManager`
      - `netplan`


### BiosDevName
```
Biosdevname naming convensions for naming network devices, generated by systemd-udevd 
* em<port-num> - Ethernet Motherbord Portnumber
* p<port>p<slot> - PCI, PCI port
* eno123 - EtherNet Onboard
* if no sufficient info on the driver, generic eth<num> is used 
```

`hostnamectl` - get/set hostname  
* hostnames are important for proper communication and local hostname resolution
* configure /etc/hosts file with approproate host name lookup settings
* DNS are configured in /etc/resolv.conf
* order of host name resultion is specified in /etc/nsswitch.conf

`ping`     - verify host reachability  
`ss`       - utility to investigate sockets (ports)  
`dig`      - DNS lookup utility  
`nmap`     - powerful tool for network analysis  
`netstat`  - depricated; use ss  
`nslookup` - deprecated; use dig  

* NetworkManager is the common service that takes care of persistent networking
* to configure NetworkManager use `nmtui` or `nmcli`
* configurations is written to configuration files

## systemd

* system daemon is the first thing started by the linux carenl
* it starts processes and can do it in parallel
* manages mounts, timers, paths, and much more
* event driven, which means that it can react to specific events
* the items that are managed by systemd are called units
* default units are in /usr/lib/systemd/system; custom units are in /etc/systemd/system

`systemclt` - controls systemd and service manager 

## ssh

`ssh`         - secure shell  
`scp`         - file transfer via ssh  
`ssh-keygen`  - generate secure keys  
`ssh-copy-id` - copy key to manged machine    
`ssh-agent`   - used to cache catchphrase answers for ssh connections  
`ssh-add`     - used to add catchephrase to ssh-agent


## process managment

`top`     - dynamic real-time information about running processes   
`ps`      - information about running processes   
`jobs`    - status of jobs in the current session  
`fg`      - run job in foreground  
`bg`      - resumes job that have been suspanded (e.g. Ctr + z) keeps them running in the background   
`nice`    - run process with custom priority  
`renice`  - alter priority of a running process  
`kill`    - kills a process by pid   
`killall` - kills all processes with same name  


## task scheduling

`cron` - classic scheduler for reoccuring tasks  
    * uses the `crond` daemon  
    * use `crontab -e` to edit tasks  
`at`   - for tasks that need to run once only  
    * uses the `atd` daemon  
    * uses at to schedule the tasks  
* systemd `timer` - the new alternative to cron jobs  
  - create a timer unit and run it using systemctl  

## logs

`journalctl` - query systemd logs  
`rsyslogd`   - modern implemnetation of the old logger `Syslog`  
`logger`     - write to syslog
* some distros do not use rsyslogd
