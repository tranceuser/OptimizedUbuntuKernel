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

## System Optimization

The below optimizations aim to improve the performance of a Linux-based system by modifying various network settings. Some of the changes include increasing the maximum number of open file descriptors, increasing the maximum number of processes, enabling TCP time-wait reuse, adjusting TCP keepalive time, setting the local port range for TCP connections, and increasing the maximum number of allowed concurrent network connections. Additionally, the values for the system's network buffer sizes are also increased to improve network performance. The optimizations aim to improve the system's overall stability and responsiveness in handling high network traffic.

General Optimization
```
net.core.somaxconn = 65535
net.ipv4.tcp_tw_reuse = 1
net.ipv4.tcp_fin_timeout = 30
net.ipv4.tcp_keepalive_time = 1200
net.ipv4.ip_local_port_range = 1024 65000
net.ipv4.tcp_syncookies = 1
net.ipv4.tcp_synack_retries = 2
net.netfilter.nf_conntrack_tcp_loose = 0 
net.ipv4.tcp_timestamps = 1
# net.netfilter.nf_conntrack_buckets value * 4
net.netfilter.nf_conntrack_max = 2000000
```
Optimize the network settings and performance for a system with:
32 GB of RAM:
```
net.ipv4.tcp_max_syn_backlog=16384
net.ipv4.tcp_rmem = "4096 87380 33554432"
net.ipv4.tcp_wmem = "4096 87380 33554432"
net.core.rmem_max = 33554432
net.core.wmem_max = 33554432
net.core.netdev_max_backlog = 15000
```
64 GB of RAM:
```
net.ipv4.tcp_max_syn_backlog = 32768
net.ipv4.tcp_rmem = "4096 87380 67108864"
net.ipv4.tcp_wmem = "4096 87380 67108864"
net.core.rmem_max = 67108864
net.core.wmem_max = 67108864
net.core.netdev_max_backlog = 30000
```
Save and exit the file.
Apply the changes by running:
```
sudo sysctl -p
```
And finally, run the last commands:
```
echo 2000000 > /sys/module/nf_conntrack/parameters/hashsize
echo 0 > /proc/sys/net/netfilter/nf_conntrack_helper
```
The first command changes the size of the hash table used by the connection tracking system, which is responsible for tracking network connections in the Linux kernel. By setting the hashsize to 2000000, you are increasing the number of entries that the table can store, potentially improving performance in high-traffic network environments.

The second command disables the use of connection tracking helpers, which are modules that assist in the handling of specific protocols (e.g., FTP, SIP, etc.). Disabling these helpers can reduce the memory overhead of the connection tracking system and improve performance. However, this may negatively impact the functionality of some protocols, so it's important to carefully evaluate the effects of this change in your specific use case.
