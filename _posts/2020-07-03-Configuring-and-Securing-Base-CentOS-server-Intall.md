---
title: Configure and Secure A Minimal CentOS 7 Install
published: true
---

* * *
This is a quick reference text based guide in place of the video I created on my youtube [here](). I won't go over as many of the details, this is more if you want the information quick and dirty without all the long winded talking. If you do have questions though, feel free to ask on one of my socials or on the comment section of the video. This will be centered around CentOS 7.

## Index

1. [Turn on SELinux](#selinux)
2. [Disable ICMP echo requests](#icmp)
3. [Setup Networking](#network)
4. [Update](#update)
5. [Users and SSH Keys](#user)
6. [Configure SSH](#ssh)
7. [Configure fail2ban](#fail2ban)


* * *

### Turn on SELinux<a name="selinux"><a/>

You can check if SELinux is enabled with the following command:

```
getenforce
```

If it is not enabled, you can enable it by editing the file `/etc/selinux/config`. When you are editing the file, change `SELINUX=disabled` to `SELINUX=enforcing`. You will then reboot and check again with `getenforce`

`reboot`

* * *

### Disable ICMP echo requests<a name="icmp"><a/>

Quite simple, add the following line to the bottom of `/etc/sysctl.conf`. This will prevent `ping` requests from returning a reply. 

```
net.ipv4.icmp_echo_ignore_all = 1
```

Load the changes with sysctl:

```
sysctl -p
```

* * *

### Setup Networking<a name="network"><a/>

Unless you are using DHCP you will need to configure/setup networking. You can do so as follows (changing IPs as needed for your own needs):

```
nmcli con mod ens192 ipv4.addresses '10.10.5.14/24' ipv4.gateway '10.10.5.1' connection.autoconnect yes
nmcli con up ens192
```

If it is not on, turn on the firewall:

```
systemctl enable --now firewalld
```

* * *

### Update<a name="update"><a/>

To update the system is rather simple, respond `y` when it prompts to isntall and a second time when it wants to add the GPG keys. 

```
yum update
```

I would reccomend installing the following packages. You will need fail2ban later. fail2ban is installed from the epel repository, so we will need to add that as well. Since we had updated, and there were (in this case) kernel updates, we'll need to reboot. 

```
yum install epel-release
yum install bash-completion setroubleshoot fail2ban
reboot
```


* * *

### Users and SSH Keys<a name="user"><a/>

Create a new user and set a decent password

```
useradd myuser
passwd myuser
```

If you need sudo access, either add the user to the wheel group, or configure a rule for your user. Be certain to use visudo as it will check your syntax.

```
usermod -aG wheel myuser
visudo 
```

On your client machine or desktop, we cam generate the keys. Hit <Enter> at each prompt. 

```
ssh-keygen
```

Copy the public key to your server (replace IP as needed). Reply yes, and then test and ssh login to verify it uses keys (shouldn't prompt you for password)

```
ssh-copy-id myuser@10.10.5.14
ssh myuser@10.10.5.14
```

* * *

### Configure SSH<a name="ssh"><a/>

We will change the port, remove ssh root login, and disable password auth (so only keys work). Edit the following lines in `/etc/ssh/sshd_config`. The port number can be anything. 

```
Port 4242
PermitRootLogin no
PasswordAuthentication no
```

Next we need to inform SELinux of the change:

```
sudo semanage port -a -t ssh_port_t -p tcp 4242
sudo semanage port -l | grep ssh
```

Don't worry about there being 2 ports configured for ssh (22 and the port you chose). A service can only run under 22 if they have the same context, i.e. something like mysql might have mysql_t which would get blocked. To get around this you would have to write your own policy for SELinux, which I won't go over today. 

Configure a new firewall rule and remove the original for the new port:

```
sudo firewall-cmd --permanent --remove-service=ssh 
sudo firewall-cmd --permanent --add-port=4242/tcp
sudo firewall-cmd --reload
```

Restart sshd 

```
systemctl restart sshd
```

* * *

### Configure fail2ban<a name="fail2ban"><a/>

Create the following jail `/etc/fail2ban/jail.local`. This will ban for 10 days after 5 failed attempts.

```
[sshd]
enabled = true
port = 4242
bantime = 864000
banaction = iptables-multiport 
maxretry = 5
```

Start and enable the service

```
systemctl enable --now fail2ban
```

* * *

If you made it this far, thank you for reading!
