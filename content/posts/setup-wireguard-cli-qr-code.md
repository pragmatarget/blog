---
author: Stu
title: "Setup a WireGuard server with client config as QR codes"
summary: "How to set up a WireGuard server and generate client configurations which are shared using QR codes."
date: 2023-07-16T17:40:23Z
draft: false
tags: [wireguard, vpn, networking, security]
---

There are plenty of guides on how to set up a WireGuard, this is mine. It aims to be quick and easy to set up, and is specifically for the "road warrior" scenario for my phone when I'm out on the move.

I've also noticed that some guides make it overly complicated to configure the WireGuard app, when it's easy using a QR code generated right from the command line using `qrencode`.

## Pre-requisites

Install the following pre-requisites

```bash
sudo apt update
sudo apt install -y wireguard qrencode
```

## Generate your server (gateway) keys

```bash
wg genkey | sudo tee /etc/wireguard/private.key
sudo chmod go= /etc/wireguard/private.key
sudo cat /etc/wireguard/private.key | wg pubkey | sudo tee /etc/wireguard/public.key
```

## Generate your server configuration

```bash
sudo su -c 'cat <<EOF > /etc/wireguard/wg0.conf
[Interface]
PrivateKey = `cat /etc/wireguard/private.key`
Address = 10.32.0.1/24
ListenPort = 51820
SaveConfig = true
PostUp = ufw route allow in on wg0 out on eth0
PostUp = iptables -t nat -I POSTROUTING -o eth0 -j MASQUERADE
PreDown = ufw route delete allow in on wg0 out on eth0
PreDown = iptables -t nat -D POSTROUTING -o eth0 -j MASQUERADE
EOF'
```

## Start the WireGuard server

```bash
sudo systemctl enable wg-quick@wg0.service
sudo systemctl start wg-quick@wg0.service
```

Once started, check the status.

```bash
sudo systemctl status wg-quick@wg0.service
```

## Generate client (peer) keys and configuration

The below bash script generated client keys and a configuration to route all traffic over the WireGuard tunnel. The configuration is piped out to `qrencode` to generate a QR code that you can use to configure your phone.

```bash
# Careful: Don't do this on a shared system
export PEER_PRIVATE_KEY=`wg genkey`
export PEER_PUBLIC_KEY=`echo $PEER_PRIVATE_KEY | wg pubkey`
# Attempt to get the interface facing address of the gateway
export ENDPOINT_IP=`curl -s ipinfo.io/ip`

sudo wg set wg0 peer $PEER_PUBLIC_KEY allowed-ips 10.32.0.0/24

cat <<EOF | qrencode -t utf8
[Interface]
PrivateKey = $PEER_PRIVATE_KEY
DNS = 1.1.1.1
Address = 10.32.0.2/24

[Peer]
PublicKey = `sudo cat /etc/wireguard/public.key`
AllowedIPs = 0.0.0.0/0
Endpoint = $ENDPOINT_IP:51820
EOF

unset PEER_PRIVATE_KEY
```

After the above has finished executing a QR code will be output in your console, ready to scan.

![](../../images/2023-07-16-22-15-31.png)

## WireGuard quick reference commands

Below are some quick reference commands to help manage WireGuard

### Show status
```bash
sudo wg
```
 
### Remove a specific peer
```bash
sudo wg set wg0 peer $PEER_PUBLIC_KEY remove
```

### Remove all peers

```bash
sudo wg show wg0 peers | while read -r line; do sudo wg set wg0 peer "$line" remove; done 
```

### Restart WireGuard service

```bash
sudo systemctl restart wg-quick@wg0.service
```