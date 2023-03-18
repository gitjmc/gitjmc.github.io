---
title: WireGuard 
author:
  name: 
  link: 
date: 2023-03-12 13:30:00 -0400
categories: [Privacy]
tags: [encryption, vpn]
render_with_liquid: false
---

This post will guide and help you how to install[^ref-1] and use[^ref-2] WireGuard.

# Install WireGuard VPN Server

## Prerequisites

In this example[^ref-3]:
- My IP: 207.148.66.172
- Network Name: emp1 s0
- WorGuard is listining on port: 33333
- Range private IP: 192.168.10.0/24

### update the system
```console
$ apt update -y 
```

### install utilyties (if not already installed)
- iptables: Iptables is an extremely flexible firewall utility built for Linux operating systems[^ref-4], [^ref-5]
- net-tools: the collection of base networking ultilities fot linux
- vim: editor

```console
$ apt install iptables net-tools vim -y
```

### Enable IP Forwarding to route packets between vpn clients and internet
Make these changes in the file below

```conf
# uncomment line below
# net.ipv4.ip_forward=1
net.ipv4.ip_forward=1
```
{: file='_conf/etc/sysctl.conf'}

#### to validate changes
```console
$ sysctl -p
net.ipv4.ip_forward=1
```

### Check Network interface Name and IP of the system
```console
$ ifconfig
```


## install Wireguard vpn server on the system
```console
$ apt install wireguard -y
```

## Configure WireGuard Server
- go to `/etc/wireguard` directory
- create private and public pair keys for wg server

```console
$ wg genkey | tee privatekey | wg publickey > publickey
```
two files `privatekey` and `publickey` must be created

3.3 show private, public keys and copy it

3.4 create a new config file called wg0.conf

3.5 open wg0.conf and add new content
Replace `Rang IP`, `Listen Port`, `Private key`, `Netwrk Interface Name`, with you info

```shell
[Interface]
## Address - A private IP address for wg0 interface. 
Address = 10.13.13.1
## ListenPort - Specify the listening port of wireguard by default 51820, but you can change it
ListenPort = 51820
## PrivateKey - A private key of the server (/etc/wireguard/privatekey)
PrivateKey = your-private-key
## PostUp - The postup will run when the WireGuard Server starts the virtual VPN tunnel.
PostUp = iptables -A FORWARD -i %i -j ACCEPT; iptables -A FORWARD -o %i -j ACCEPT; iptables -t nat -A POSTROUTING -o eth+ -j MASQUERADE
## PostDown - The postdown rules run when the WireGuard Server stops the virtual VPN tunnel.
PostDown = iptables -D FORWARD -i %i -j ACCEPT; iptables -D FORWARD -o %i -j ACCEPT; iptables -t nat -D POSTROUTING -o eth+ -j MASQUERADE
## Specify the command that allows traffinc to leave the server and give the VPN clients access to the internet.

```
{: file="/etc/wireguard/wg0.conf" }

## Change permission on privatekey and wg0.conf files

```console
$ chmod 600 privatekey
$ chmod 600 wg0.conf

# to check changes
$ ll
```

## start and enable wg server

```console
$ systemctl start wg-quick@wg0
$ systemctl enable wg-quick@wg0
```

## Make sure wg vpn server is running

```console
$ systemctl status wg-quick@wg0
```

## Verify interface named wg0 is up

```console
$ wg
```

## Configure ufw firewall to open wg vpn server
### Check status of the ufw firewall
```console
$ ufw status
```

### Allow port 33333/udp

```console
$ ufw allow 33333/udp
```

### Reload ufw

```console
$ ufw reload
```

## dockorize wireguard

### Prerequistes
- docker and docker compose installed

### Run docker compose
```yaml
version: "2.1"
services:
  wireguard:
    image: linuxserver/wireguard
    container_name: wireguard
    cap_add:
      - NET_ADMIN
      - SYS_MODULE
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Etc/UTC
      - SERVERURL=auto #optional
      - SERVERPORT=51820 #optional
      - PEERS=5 #optional
      - PEERDNS=auto #optional
      - INTERNAL_SUBNET=10.13.13.0 #optional
      - ALLOWEDIPS=0.0.0.0/0 #optional
      - PERSISTENTKEEPALIVE_PEERS= #optional
      - LOG_CONFS=true #optional
    volumes:
      - ./config:/config
      - /lib/modules:/lib/modules #optional
    ports:
      - 51820:51820/udp
    sysctls:
      - net.ipv4.conf.all.src_valid_mark=1
    restart: unless-stopped

```
{: file="docker-compose.yaml" }



# tempo content to delete

```shell
[Interface]
Address = 10.13.13.1
ListenPort = 51820
PrivateKey = your-private-key
PostUp = iptables -A FORWARD -i %i -j ACCEPT; iptables -A FORWARD -o %i -j ACCEPT; iptables -t nat -A POSTROUTING -o eth+ -j MASQUERADE
PostDown = iptables -D FORWARD -i %i -j ACCEPT; iptables -D FORWARD -o %i -j ACCEPT; iptables -t nat -D POSTROUTING -o eth+ -j MASQUERADE

[Peer]
# peer1
PublicKey = K8cS34Z+1Hy/fXxRdkFS/Cz+PxjrAvnPp3dEfspw6UA=
PresharedKey = I62J+xVCyYksK9wL0uobAMva276yyJJf/kr2low3uwQ=
AllowedIPs = 10.13.13.2/32

```
{: file="/etc/wireguard/wg0.conf" }


# Resources

[^ref-1]: [**Wireguard installation**](https://www.wireguard.com/install/)
[^ref-2]: [**Wireguard home page**](https://www.wireguard.com/)
[^ref-3]: [**How to Install and Configure WireGuard VPN Server on Ubuntu (youtube)**](https://www.youtube.com/watch?v=ZMqUyu53PbA) 
[^ref-4]: [**The Beginner’s Guide to iptables, the Linux Firewall**](https://www.howtogeek.com/177621/the-beginners-guide-to-iptables-the-linux-firewall) 
[^ref-5]: [**Iptables Tutorial – Securing Ubuntu VPS with Linux Firewall**](https://www.hostinger.com/tutorials/iptables-tutorial#:~:text=Simply%20put%2C%20iptables%20is%20a,incoming%20and%20outgoing%20data%20packets.)
