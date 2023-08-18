---
author: Stu
title: "Remote packet capture using tcpdump and SSH; a cross platform approach"
summary: "How to do remote packet capture on Linux machines and stream the packets to a Linux, MacOS, or Windows host to view them on."
date: 2023-08-13T20:40:36Z
draft: false
tags: [networking, tcpdump, wireshark]
---

I'm going to step through how I do remote packet capture on Linux machines and stream the packets to a Linux, MacOS, or Windows host to view them on.

Why would you do this? If you want to understand how something works over a network, or perhaps you're debugging a network issue, you can't beat firing up Wireshark. However, if you only have remote console access to the box or VM the alternatives like tcpdump and tshark can be difficult to work with. Plus, having the packets on your local machine makes processing, storing or sharing them easier.

# Pre-requisites

* A remote Linux machine that you'd like to capture packets on
* SSH access to the remote machine
* tcpdump installed on the remote machine
* Netcat (nc) installed on the remote machine
* Good bandwidth to the remote machine, as you're streaming the packet capture back to your local machine.
* SSH client installed on your local machine
* Wireshark installed on your local machine

# A cross-platform approach 

Connect to the remote server you'd like to capture packets on using the following command:
```bash
ssh -L 8888:127.0.0.1:8888 <user>@<host> "sudo tcpdump -U -w - 'not port 22' | nc -l -s 127.0.0.1 -p 8888"
```

This sets up local port forwarding (`-L`) to the remote machine on port 8888. Then runs tcpdump on the remote machine in unbuffered mode (`-U-`) and writes the packets to standard out (`-w -`), which is then piped into netcat (`nc`). Netcat listens for connections (`-l`) on the local interface as specified by `-l 127.0.0.1` on port 8888 (`-p`).

The tcpdump filter `'not port 22'` excludes connections on port 22 to prevent your SSH session show up in the packet capture, if you need to see SSH traffic other than your own use `'not host <your IP>'`.

Once the above command is running, you can then open another terminal and run Wireshark on your local box using one of the commands below.

## Linux 
```bash
wireshark -k -i TCP@127.0.0.1:8888
```

## MacOS
```bash
/Applications/Wireshark.app/Contents/MacOS/Wireshark -k -i TCP@127.0.0.1:8888
```

## Windows
```cmd
& 'C:\Program Files\Wireshark\Wireshark.exe' -k -i TCP@127.0.0.1:8888
```
