# Administration And Networking

## ACL 

`getfacl` - get current acl settings
`setfacl` - set acl settings
* regular ACL takes care of all currently existing files
* default ACL will take caare of all new files
* use ACLs as an infra solution, they should be configured on dirs before you start to work with files in the dirs
`visudo` - edit /etc/sudoers, can be used to extend token validity for sudo among other sudo connfigurations
* can add `Defaults     timestamp_type=global,timestamp_timeout=<minutes>` in /etc/sudoers via visudo to extends token validity.
* instead of writing directly to /etc/sudoers, drop in files can be added to `/etc/sudoers.d/<user>` to configure sudo for individual users/groups.
```
//     ALL (Allow)         ! (Except for) 
<user> ALL=/usr/bin/passwd,! /usr/bin/passwd root
<user> ALL=(ALL) NOPASSWD: ALL (DO NOT USE)
```
* you can research `sudoedit`


### attributes

POISX details file attributes to complement premmisions, most commonly used it the immutable attribute

`chattr` - change attributes, `+i` for adding immutable - won't be able to write or delete even as root.

## service managment

`systemctl` - systemd system and service manager

* systemd provides 3 main functions
    1. system and service managment
    2. software platform as base to develop more linux software
    3. interface between the OS and the karnel

* systemd provides files to manage servicees, mounts, paths. sockets and more (units) aside from other functionliy
    * `systemd-journald` - logging
    * `systemd-udevd` - hardware initialization
    * `systemd-logind` - session managment
    * `systemd-networkd` - network configuration
    * `systemd-nspawn` - offers containerization functionality

* default units are located in `/etc/lib/systemd/system`
* custom units are located in `/etc/systemd/system`
* `systemctl cat` - show current unit content
* `systemctl edit` - edit a unit and create drop-in files in `/etc/systemd/system/<unit_name>.d`
* `man systemd.directives` - for edit options for services manual

### unit targets

* target is a group of units that can be managed as one, using `systemctl start/stop etc...`
* some targets are "isolatable" which means they can be used to define the state in which the system should be started, kind of like safe mode in windows:
    * `emergency.target`
    * `rescure.target`
    * `multi-user.target`
    * `graphical.target`
* `AllowIsolate` option in the [Unit] descriptor for a service defines an isolatable target
* `systemctl get-default` to get the default target in which the system will boot
* `systemctl set-default` to set the default target in which the system will boot

### unit dependencies

* dependencies can be defined in unit files
* `systemctl list-dependencies` - show current dependencies

#### cheatsheet
```
Requires - if the unit loads, units listed here will also load. if one of the required units is deactivated, this unit will also be deactivated
Requisite - if the units listed here are not already loaded, thiss unit will fail
Wants - this unit wants to load the units listed here, but won't fail if any of these units fail 
Before - will start this unit before the units mentioned with "Before="
After -  will start this unit after the units mentioned with "After="
```

### systemd timers

* modern alternative to cron jobs
* currently both cron and timers can be seen in the wild, so knowing cron is still important
* every timer has a matching service file which must have the same name
    - for instance: `fstrim.timer` works with `fstrim.service`
* `systemctl list-unit -t timer` to list active timers
* to use timers, enable the tmer, not the service
    - `systemctl enable fstrim.timer` - fstrim.timer will run the cron job for fstrim.service
* consult `man systemd.timer` for more details

### systemd sockets

* systemd sockets are used to listen for incoming traffic on a socket and when that occurs start the matching service
* like with timers, the names of sockets must match the names of services they are listening
* `systemctl list-files -t socket` for a list of sockets that are available

### systemd mounting filesystem

* `/etc/fstab` on modern linux distros is an input file for systemd
* based on fstab content, mount files are generated in `/run/systemd/generator`
* the names of mount unit files reflect the names of the directoreis on which they are mounting
    - `data.mount` if mounting on /data
    - `mnt-nfs.mount` if mounting on /mnt/nfs
* we can create mount unit files in `/etc/systemd/system and bypass fstab




# Networking 

`ip link show` - get all interfaces, for more information `lshw -class network` (list hardware of network kind)


* NetworkManager is used for persistent interface configuration on Red Hat linux.
    * use nmtui or nmcli to manage
    * config is stored in /etc/sysconfig/network-scripts

* On ubuntu, Netplan is used for persistent interface configuration
    * on server, systemd-networkd as the backend to netplan
    * on desktop, NetworkManager is used


### systemd-networkd

* the standard for managing networks on ubuntu server  
* it uses one or more YAML files, provided by netplan for the netwoek configuration: `/etc/netplan/*.yaml`  
    * on the netplan configuration, `render:networkd` is used to connect to systemd-networkd  
    * the netplan config files need to be created manually for persistent network config  
    * all netplan config files are loaded by alphabet sort, and can be used to override rules.
* the front-end for querying network status is `networkctl`  
    * `networkctl status` - get current status 
    * `networkctl up|down` - bring a link up | down


### static routes notes

* every node in a network is configured wit a default gateway for external communication
* `ip route show` to check default gateway (ifconfig deprecated)
* static routes can be added to define route to a network that is not behind the default gateway
* `ip route add <addr/mask> via <addr>` is used to add a custom route

### DNS clients notes

* /etc/resolve.conf is the old standard for configuring DNS name servers
* on NetworkManager systems, the file can be written directly or managed by NetworkManger
* on systemd-networkd systems we manage it with systemd-resolved (/etc/resolv.conf is symbolic link to /run/systemd/resove/stub-resolve.conf)

### bridge notes

* a bridge is used to introduce a software defined bridge, whoch behaves as a virtualized switch
* bridges are common in virtualiztion and containers
* using a bridge allows for creation of an internal network, which is strictly separated from the outside using NAT
    * nodes on a bridge can address external nodes
    * external nodes can only access internal nodes using port forwarding
* `brctl` - control bridges

### network sockets

* a network socket is a connection EP, <ip_addr>:<port_num>  
* there's also UNIX sockets which are EPs for UNIX/Linux communications
    * `SOCK_STREAM` - compared to TCP
    * `SOCK_DGRAM` - compared to UDP
    * `SOCK_SEQPACKET` - comapred to SCTP
* `ss` is the current standard for showing socket information, replacing `netstat`

### diagnostics

`tcpdump` & `nmap` for network diagnostics  


## troubleshooting 


`ip`                     - check machine's network configurations  
`traceroute`/`tracepath` - analyze path between source and destination  
`ping`                   - ICMP request to destination, used to test connectivity  
`mrt`                    - combine both `traceroute` and `ping`  
`arp`                    - show ip and MAC address resolution tables  
`dig`                    - current DNS lookup tool  
`nslookup`               - legacy DNS lookup tool  
`whois`                  - query DNS domain and request advanced information  