# Security

## firewalls

### network ports
* firewalls work on ip addresses and ports
* services usually communicate via a dedicated port
* ports from 0 - 1023 are considered reserved or privileged
* ports 1024 - 65536 are used for dynamic purposes and can be used by services that dont have a priviledged port
* static port definitions can be found in `/etc/services`
* `ss -tunap`/`netstat -tulpen` for an overview of ports and services listening on these ports

### ufw

UFW - Uncomplicated Firewall, simple tool to manage firewall  

* `sudo ufw enable`                                        - enables the firewall and automatically loads its rules at system startup.
* `sudo ufw allow <port>`                                  - allows incoming traffic on the specified port.
* `sudo ufw reject <direction> <port>`                     - rejects traffic of the specified protocol and port in the specified direction.
* `sudo ufw status`                                        - displays the current status of the firewall and its rules.
* `sudo ufw delete <rule>`                                 - deletes the specified rule.
* `sudo ufw deny <protocol> from <IP> to <IP> port <port>` - denies incoming traffic of the specified protocol and port from the specified IP address to the specified IP address.
* `sudo ufw reset`                                         - disables the firewall and deletes all its rules.
* `sudo ufw app list`                                      - lists the available application profiles that can be used to create firewall rules.
* `sudo ufw app info <app>`                                - displays information about the specified application profile.
* `sudo ufw logging <on/off>`                              - enables or disables logging of the firewall's actions.
* `man ufw`                                                - displays the manual page for the UFW command.


### firewalld

A more advanced firewwall solution found in RH systems that contains the following parts  


***Zone          -***  collection of networks that is facing a specific direction and to which rules can be assigned  
***Interfaces    -***  induvidual NICs assigned to zones  
***Services      -***  XML configuration that specifies ports to be opened and modules that should be used  
***Forward Ports -***  used to send incoming traffic on a specific port to another port which may be on another machine  
***Masquerading  -***  provides NAT capabilities on a router  
***Rich Rules    -***  extention of the firewalld syntax to make more complex configurations possible  

`firewall-cmd` - `firewalld` cli tool  
many elements have `get`,`set`,`list` options which makes working with them intuative

* `firewall-cmd --list-services`
* `firewall-cmd --get-services`
* `firewall-cmd --add-services=<service>`
* `firewall-cmd --remove-services=<service>`

