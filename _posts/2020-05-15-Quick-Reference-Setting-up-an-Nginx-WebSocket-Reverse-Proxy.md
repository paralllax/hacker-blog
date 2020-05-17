---
title: Nginx Websocket Reverse Proxy quick reference
published: true
---

* * *

This is a quick reference guide to configuring Nginx as a Websocket Reverse Proxy. If you don't have a websocket application, but would like to try setting one up, [this agario clone](https://github.com/huytd/agar.io-clone) is a good place to start. 

## Index

1. [Install nginx](#install)
2. [Configuring nginx](#config)
3. [SELinux/Firewall](#add)

* * *
### Install nginx<a name="install"><a/>

As this is aimed at being more of a quick reference, this guide will make a few assumptions, such as that you already have a server provisioned, and a generic familiarity with web servers. To install nginx: (note you can install nginx-mainline from some repositories for new features, bug fixes, updates)

Redhat / CentOS
```bash
yum install nginx
```

Ubuntu / Debian
```bash
apt-get install nginx
```

Arch
```bash
pacman -S nginx
```

If you would like to compile nginx from scratch with custom binaries/modules please see [openresty](https://openresty.org/en/)


* * *
### Configuring nginx<a name="config"><a/>

Ensure you have the following directive in your `/etc/nginx/nginx.conf` or add the ensuing configuration to the http block in the main config.

```
	include  /etc/nginx/conf.d/*.conf;
```

Create a new file in `/etc/nginx/conf.d/` ending with the suffix `.conf`, add the following configuration. If you are adding this to the main config instead, ensure it is inside the http block. 

```
upstream agario_websocket_server {
    server 192.168.1.1:3000;
}

server {
    listen 80;
    server_name agar.example.com;

     location / {
      proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
      proxy_set_header Host $host;

      proxy_pass http://agario_websocket_server;

      proxy_http_version 1.1;
      proxy_set_header Upgrade $http_upgrade;
      proxy_set_header Connection "upgrade";

    }
}
```

If you would like an explanation of each directive, you can read about them [here](http://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_set_header).

* * *
### SELinux/Firewall<a name="add"><a/>

In case you happen to have SELinux enabled on the client, or need to add firewall rules, this can be done with the following. In this case we are sending the traffic to port 3000, if that happens to be your http port, you may need to make sure your selinux policy will allow traffic.

```
semanage port -a -t http_port_t -p tcp 3000
```

If you are using firewalld, this is how you can open a port, if you need udp, run the same command, but replace tcp with udp.

```
firewall-cmd --permanent --zone=public --add-port=3000/tcp
firewall-cmd --reload
```

That concludes the quick reference. If you have any questions, as always, feel free to find me on my social and hit me up. Thanks for reading!
