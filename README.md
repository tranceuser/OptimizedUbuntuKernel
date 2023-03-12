# Optimizing Ubuntu 22.04 for Low Latency and High Performance

This guide will walk you through the steps for optimizing your Ubuntu 22.04 system for low latency and high performance.

## Updating and Upgrading the System
Before we start optimizing the system, it's important to make sure it's up to date. To update and upgrade your Ubuntu 22.04 system, run the following commands in the terminal:
```
sudo apt-get update && sudo apt-get upgrade -y
```

## Installing Nano
Nano is a popular text editor that is easy to use. To install Nano, run the following command in the terminal:
```
sudo apt install nano -y
```

## Installing a Real-Time Kernel
A real-time kernel is designed to provide predictable and consistent response times. To install a real-time kernel on Ubuntu 22.04, follow these steps:
```
sudo apt-get install linux-image-lowlatency -y
```
Finally, reboot your system to start using the real-time kernel:
```
reboot
```

## Use a Low-Latency Scheduler
A low-latency scheduler is designed to minimize the amount of time a task spends waiting for CPU time.
To use a low-latency scheduler, you need to set the scheduler type to bfq by adding the following line to the grub file:
```
sudo nano /etc/default/grub
```

Find the line that starts with GRUB_CMDLINE_LINUX_DEFAULT, and add elevator=bfq to the end of the line:
```
GRUB_CMDLINE_LINUX_DEFAULT="quiet splash elevator=bfq"
```

Save the file and update the grub configuration:
```
sudo update-grub
```

## Reduce Swap Usage
Swap usage can cause increased latency, so it is recommended to reduce swap usage. You can add the following line to the sysctl.conf file:
```
sudo nano /etc/sysctl.conf
```
Add the following line to the end of the file:
```
vm.swappiness=1
```
Save the file and reload the sysctl configuration:
```
sudo sysctl -p
```

## [Ubuntu Firewall Prerequisites](https://github.com/tranceuser/ubuntu_firewall_prerequisites)

  Before moving forward with the below system optimizations, please make sure to check out my Ubuntu Firewall Prerequisites repository [here](https://github.com/tranceuser/ubuntu_firewall_prerequisites). This repository will guide you through the process of installing IPTables, Ipset, Conntrack, and netfilter-persistent, which are required for the system optimizations described below.

## System Optimization

The below optimizations aim to improve the performance of a Linux-based system by modifying various network settings. Some of the changes include increasing the maximum number of open file descriptors, increasing the maximum number of processes, enabling TCP time-wait reuse, adjusting TCP keepalive time, setting the local port range for TCP connections, and increasing the maximum number of allowed concurrent network connections. Additionally, the values for the system's network buffer sizes are also increased to improve network performance. The optimizations aim to improve the system's overall stability and responsiveness in handling high network traffic.
1. Open the /etc/sysctl.conf file for editing:
```
sudo nano /etc/sysctl.conf
```
2. General Optimization
```
net.core.somaxconn = 65535
net.ipv4.tcp_tw_reuse = 1
net.ipv4.tcp_fin_timeout = 30
net.ipv4.tcp_keepalive_time = 1200
net.ipv4.ip_local_port_range = 1024 65000
net.ipv4.tcp_syncookies = 1
net.ipv4.tcp_synack_retries = 2
net.ipv4.tcp_timestamps = 1
net.netfilter.nf_conntrack_tcp_loose = 0
net.ipv4.tcp_max_syn_backlog = 65535
net.core.netdev_max_backlog = 65535
net.netfilter.nf_conntrack_max = 1048576
net.nf_conntrack_max = 1048576
net.netfilter.nf_conntrack_buckets = 4194304
```
3. Optimize the network settings and performance for a system with:
16 GB of RAM:
```
net.ipv4.tcp_rmem = 4096 87380 16777216
net.ipv4.tcp_wmem = 4096 87380 16777216
net.core.rmem_max = 16777216
net.core.wmem_max = 16777216
```
32 GB of RAM:
```
net.ipv4.tcp_rmem = 4096 87380 33554432
net.ipv4.tcp_wmem = 4096 87380 33554432
net.core.rmem_max = 33554432
net.core.wmem_max = 33554432
```
64 GB of RAM:
```
net.ipv4.tcp_rmem = 4096 87380 67108864
net.ipv4.tcp_wmem = 4096 87380 67108864
net.core.rmem_max = 67108864
net.core.wmem_max = 67108864
```
4. Save and exit the file.
5. Apply the changes by running:
```
sudo sysctl -p
```
## Maximum number of open files
The nofile parameter sets the maximum number of open files that can be used by a process. In some cases, a high number of open files may be required by certain applications, especially those that handle large amounts of data or perform heavy I/O operations.
1. Open the /etc/security/limits.conf file with a text editor using the following command:
```
sudo nano /etc/security/limits.conf
```
2. Add the following lines to the end of the file to set the nofile parameter:
```
* hard nofile 4194304
* soft nofile 4194304
```
This sets the maximum number of open files to 4194304 for all users.
To apply the changes, you need to log out and log back in, or you can restart your system using the following command:
```
sudo reboot
```
## Conntrack
You can also check the hashsize of Conntrack by using cat:
```
cat /sys/module/nf_conntrack/parameters/hashsize
```
Hashsize value is determined dynamically based on the amount of memory available on the system. However, you can force the hashsize value to a specific value.

If the Conntrack hashsize value is still not set to 1048576 after following the steps I provided earlier, it's possible that there's another setting overriding the value you're trying to set.

One possible solution is to create a systemd service that sets the Conntrack hashsize value at boot time. Here's how you can do this:
1. Create a new file in the /etc/systemd/system directory called conntrack-hashsize.service:
```
sudo nano /etc/systemd/system/conntrack-hashsize.service
```
2. Add the following lines to the file:
```
[Unit]
Description=Set Conntrack Hashsize Value

[Service]
Type=oneshot
ExecStart=/bin/bash -c "modprobe nf_conntrack && echo 1048576 > /sys/module/nf_conntrack/parameters/hashsize"

[Install]
WantedBy=multi-user.target
```
This defines a new systemd service that sets the Conntrack hashsize value to 1048576 at boot time.

3. Save the file and close the editor.
4. Enable the new service by running the following command:
```
sudo systemctl enable conntrack-hashsize.service
```
This will ensure that the new service is started automatically at boot time.
Reboot the system.
