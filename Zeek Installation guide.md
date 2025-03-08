**Zeek Cluster Installation Requirements**

A Zeek cluster is required for high-performance network monitoring, especially when dealing with high-throughput network traffic. Below are the hardware, software, and configuration requirements to set up a Zeek cluster.

###  Hardware Requirements
*Minimum Hardware Requirements (for Small Deployments)*

* CPU: 4+ cores
* RAM: 8GB+
* Storage: 100GB SSD (log storage)
* Network Interface: 1Gbps or higher (dedicated sniffing interface)

*Recommended Hardware (for Medium to Large Deployments)* 

* CPU: 12+ cores (depends on traffic volume)
* RAM: 32GB+ (depends on log retention & analysis needs)
* Storage: 500GB+ SSD (Zeek logs grow fast; consider log rotation)
* Network Interface: 10Gbps+ (for high-speed networks)
* Dedicated Network TAP or SPAN Port: Required for passive monitoring
* High-Performance NICs: Use Intel, Mellanox, or Napatech for best performance
* Recommended OS
	* Ubuntu 22.04 LTS (Preferred)

---
### Full lab requireemtns ( Network Tap, Zeek Sensor and Splunk Server)

*  Phyiscal Server ( DELL) : This will be deployed to function as zeek sensor 
	* 	Processor : 12 CUP cores
	*  Memeory : 8GB 
	*  Disk : 1TB
	*  Network Interfaces : 
		*  eno1 : Management Interface (192.168.1.121)
		*  eno2 : TAP Port ( for receiving the tapped traffic, NO IP SET) 
		*  idrac : IDRAC Interface (192.168.1.120)
* Network Tap :
	* Passive Tap or SPAN Protocol 
* Splunk Server : 
	* Virtual Machine : Ubuntu OS installed with latest splunk server
	* Network IP : 192.168.1.145
---

**Zeek Cluster Installation Requirements**

A Zeek cluster is required for high-performance network monitoring, especially when dealing with high-throughput network traffic. Below are the hardware, software, and configuration requirements to set up a Zeek cluster.

###  Installation Requirements
**Minimum Hardware Requirements (for Small Deployments)**

	* CPU: 4+ cores
	* RAM: 8GB+
	* Storage: 100GB SSD (log storage)
	* Network Interface: 1Gbps or higher (dedicated sniffing interface)


**Recommended Hardware (for Medium to Large Deployments)** 

	* CPU: 12+ cores (depends on traffic volume)
	* RAM: 32GB+ (depends on log retention & analysis needs)
	* Storage: 500GB+ SSD (Zeek logs grow fast; consider log rotation)
	* Network Interface: 10Gbps+ (for high-speed networks)
	* Dedicated Network TAP or SPAN Port: Required for passive monitoring
	* High-Performance NICs: Use Intel, Mellanox, or Napatech for best performance
	
**Required OS/Software**
	* Ubuntu 22.04 LTS (Preferred)
	* Zeek depenedcies 

--

####Installation Steps####
* Network TAP Port tuning : 
	* Minimize packet loss and ensure Zeek sees full packet data by applying network sniffing optimizations: settings max ring parameters, disabling NIC offloading, and enabling promiscuous mode.
*     Build Zeek from source with optimizations & jemalloc.
*     Create a non-root Zeek user to minimize impact in the event that Zeek is compromised.
*     Deploy and run Zeek to start analyzing traffic.
*     Create a cron job to perform Zeek maintenance tasks.

**Step 1 : Network TAP Port Tuning**

```
1. login to the sensor : ssh msalah@192.168.1.121 and make sure the tap port interface is up eno2
msalah@zeek-sensor:~$ sudo ip link set dev eno2 up
msalah@zeek-sensor:~$ ifconfig
eno1: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.1.121  netmask 255.255.255.0  broadcast 192.168.1.255
        inet6 fe80::bacb:29ff:feb6:9f20  prefixlen 64  scopeid 0x20<link>
        inet6 2001:8f8:1e3f:4a19:bacb:29ff:feb6:9f20  prefixlen 64  scopeid 0x0<global>
        ether b8:cb:29:b6:9f:20  txqueuelen 1000  (Ethernet)
        RX packets 3406  bytes 610721 (610.7 KB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 1736  bytes 194516 (194.5 KB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
        device interrupt 17

eno2: flags=387<UP,BROADCAST,NOARP,PROMISC>  mtu 1500
        ether b8:cb:29:b6:9f:21  txqueuelen 1000  (Ethernet)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
        device interrupt 18

lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        inet6 ::1  prefixlen 128  scopeid 0x10<host>
        loop  txqueuelen 1000  (Local Loopback)
        RX packets 21245  bytes 5964156 (5.9 MB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 21245  bytes 5964156 (5.9 MB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

```

a) Use ethtool to determine the maximum ring parameters for your sniffing interfaces

```
msalah@zeek-sensor:~$ ethtool -g eno2
	Ring parameters for eno2:
	Pre-set maximums:
	RX:			2047
	RX Mini:		n/a
	RX Jumbo:		n/a
	TX:			511
	TX push buff len:	n/a
	Current hardware settings:
	RX:			2047
	RX Mini:		n/a
	RX Jumbo:		n/a
	TX:			511
	RX Buf Len:		n/a
	CQE Size:		n/a
	TX Push:		off
	RX Push:		off
	TX push buff len:	n/a
	TCP data split:		n/a
```
Here we have adjusted the TAP port with it's max ring rx=2047

b) Disable NIC Offloading
```
msalah@zeek-sensor:~$ sudo ethtool -K eno2 rx off tx off sg off tso off ufo off gso off gro off lro off
```

c) Set sniffing network interfaces to promiscuous mode

```
msalah@zeek-sensor:~$ sudo ip link set eno2 arp off multicast off allmulticast off promisc on
msalah@zeek-sensor:~$ ip ad sh | grep -i promisc
3: eno2: <BROADCAST,NOARP,PROMISC,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000

```

**Optional to make all above network tuning to be persistenatn across system reboot**

```
1. set-max-ring 
msalah@zeek-sensor:~$ sudo vim /etc/networkd-dispatcher/routable.d/10-set-max-ring
#!/bin/sh
# Set ring rx parameters for all sniffing interfaces
ethtool -G eno2 rx 2047
msalah@zeek-sensor:~$ sudo chmod 750 /etc/networkd-dispatcher/routable.d/10-set-max-ring

2. disbale nic offloading
msalah@zeek-sensor:~$ sudo vim /etc/networkd-dispatcher/routable.d/20-disable-checksum-offload
#!/bin/sh
# Disable checksum offloading for all sniffing interfaces
ethtool -K eno2 rx off tx off sg off tso off ufo off gso off gro off lro off
msalah@zeek-sensor:~$ sudo chmod 750 /etc/networkd-dispatcher/routable.d/20-disable-checksum-offload

3. set promiscous mode on the tap port

msalah@zeek-sensor:~$ sudo vim /etc/networkd-dispatcher/routable.d/30-enable-promisc-mode
#!/bin/sh
# Enable promiscuous mode for all sniffing interfaces
ip link set eno2 arp off multicast off allmulticast off promisc o

msalah@zeek-sensor:~$ sudo chmod 750 /etc/networkd-dispatcher/routable.d/30-enable-promisc-mode
```

**Check that all network tap port tunings have been applied**

```
msalah@zeek-sensor:~$ sudo ethtool -g eno2
Ring parameters for eno2:
Pre-set maximums:
RX:			2047
RX Mini:		n/a
RX Jumbo:		n/a
TX:			511
TX push buff len:	n/a
Current hardware settings:
RX:			2047
RX Mini:		n/a
RX Jumbo:		n/a
TX:			511
RX Buf Len:		n/a
CQE Size:		n/a
TX Push:		off
RX Push:		off
TX push buff len:	n/a
TCP data split:		n/a
msalah@zeek-sensor:~$ sudo ethtool -k eno2 |grep -i checksumming
rx-checksumming: off
tx-checksumming: off
msalah@zeek-sensor:~$ sudo ip ad sh eno2 | grep -i promisc
3: eno2: <BROADCAST,NOARP,PROMISC,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000

```

**Step 2: Install and Build Zeek from Source code with optimization**

**Install Zeek dependencies :**
``sudo apt-get install cmake make gcc g++ flex bison libpcap-dev libssl-dev python3 python3-dev python3-git python3-semantic-version swig zlib1g-dev libjemalloc-dev``

**Ensure all packages aer up to date**

```
sudo apt-get update
sudo apt-get dist-upgrade
sudo reboot
```

**Create a dedicated user "zeek" to have ownership on the zeek source code & and tun it.**

```
sudo adduser zeek
sudo mkdir /opt/zeek
sudo chown -R zeek:zeek /opt/zeek
sudo chmod 750 /opt/zeek
sudo usermod -G sudo zeek

```
**Download, Compile, and Install Zeek**

```
su - zeek
wget https://download.zeek.org/zeek-7.1.0.tar.gz
tar -xzvf zeek-7.1.0.tar.gz
cd zeek-7.1.0
./configure --prefix=/opt/zeek --enable-jemalloc --build-type=release
make
make install

```
Explanation of Flags:
``    --prefix=/opt/zeek → Installs Zeek in /opt/zeek instead of the default location (/usr/local).``

``    --enable-jemalloc → Enables jemalloc, a memory allocator that improves Zeek’s memory management, reducing fragmentation and enhancing performance.``

 ``   --build-type=release → Compiles Zeek with optimizations for production use (better performance compared to debug builds).``
 
The following commands assign capabilities to Zeek and Capstats, allowing them to capture network traffic without requiring root privileges:

```
sudo setcap cap_net_raw=eip /opt/zeek/bin/zeek
sudo setcap cap_net_raw=eip /opt/zeek/bin/capstats

```
Explanation:

* ``setcap cap_net_raw=eip`` → Grants the cap_net_raw capability, which allows an application to open raw sockets (necessary for packet capture).
* ``    /opt/zeek/bin/zeek`` → Applies this capability to Zeek.
*  ``   /opt/zeek/bin/capstats`` → Applies this capability to Capstats (used for monitoring packet capture performance).
* ``    eip (Effective + Inheritable + Permitted)`` → Ensures the program retains this capability even when executed by a non-root user.

**Add Zeek to PATH**

```
zeek@zeek-sensor:~$ export PATH="/opt/zeek/bin:$PATH" >>~/.bashrc
zeek@zeek-sensor:~$ source ~/.bashrc

```
**Zeek Cluster Configurations :**
A Zeek Cluster is essential for scaling Zeek in high-traffic environments. Your setup should be optimized to handle 1Gbps traffic ingestion efficiently. Below are the best practices for configuring and tuning a Zeek cluster.

Cluster Architecture Overview

A Zeek cluster consists of:

    Manager Node → Controls the cluster, aggregates logs, and manages workers.
    Logger Node → Handles log storage (can be combined with the Manager).
    Proxy Node(s) → Distributes workload among workers (usually 1-2 proxies).
    Worker Nodes → Capture and analyze traffic, distributed across CPU cores.

    ⚡ Your Setup:

    Manager & Logger: Combined on the same node
    Workers: 10 workers (optimized for a 12-core CPU)
    Traffic Rate: 1Gbps
    
```
zeek@zeek-sensor:~$ cat /proc/cpuinfo | grep processor
processor	: 0
processor	: 1
processor	: 2
processor	: 3
processor	: 4
processor	: 5
processor	: 6
processor	: 7
processor	: 8
processor	: 9
processor	: 10
processor	: 11
```

Configure cluster : /opt/zeek/etc/node.cfg

```
sudo vim /opt/zeek/etc/node.cfg
Example ZeekControl node configuration
#
# This example has a standalone node ready to go except for possibly changing
# the sniffing interface.

# This is a complete standalone configuration.  Most likely you will
# only need to change the interface.
#[zeek]
#type=standalone
#host=localhost
#interface=eth0

## Below is an example clustered configuration. If you use this,
## remove the [zeek] node above.

[logger-1]
type=logger
host=localhost
#
[manager]
type=manager
host=localhost
#
[proxy-1]
type=proxy
host=localhost
#

[worker-1]
type=worker
host=localhost
interface=eno2
lb_method=pf_ring
lb_procs=10
# Number of workers (10 workers for a 12-core CPU)

[worker-2]
type=worker
host=localhost
interface=eno2
#interface=af_packet::eno2
lb_method=pf_ring
lb_procs=10

[worker-3]
type=worker
host=localhost
interface=eno2
lb_method=pf_ring
lb_procs=10

[worker-4]
type=worker
host=localhost
interface=eno2
lb_method=pf_ring
lb_procs=10

[worker-5]
type=worker
host=localhost
interface=eno2
lb_method=pf_ring
lb_procs=10

[worker-6]
type=worker
host=localhost
interface=eno2
lb_method=pf_ring
lb_procs=10

[worker-7]
type=worker
host=localhost
interface=eno2
lb_method=pf_ring
lb_procs=10

[worker-8]
type=worker
host=localhost
interface=eno2
lb_method=pf_ring
lb_procs=10

[worker-9]
type=worker
host=localhost
interface=eno2
lb_method=pf_ring
lb_procs=10

```

```
zeek@zeek-sensor:~$ zeekctl deploy
waiting for lock (owned by PID 8538) ...
checking configurations ...
installing ...
removing old policies in /opt/zeek/spool/installed-scripts-do-not-touch/site ...
removing old policies in /opt/zeek/spool/installed-scripts-do-not-touch/auto ...
creating policy directories ...
installing site policies ...
generating cluster-layout.zeek ...
generating local-networks.zeek ...
generating zeekctl-config.zeek ...
generating zeekctl-config.sh ...
stopping ...
stopping workers ...
stopping proxy ...
stopping manager ...
stopping logger ...
starting ...
starting logger ...
starting manager ...
starting proxy ...
starting workers ...
```

```
zeek@zeek-sensor:~$ zeekctl status
Name         Type    Host             Status    Pid    Started
logger-1     logger  localhost        running   40030  08 Mar 16:06:22
manager      manager localhost        running   40083  08 Mar 16:06:23
proxy-1      proxy   localhost        running   40135  08 Mar 16:06:24
worker-1-1   worker  localhost        running   40488  08 Mar 16:06:25
worker-1-2   worker  localhost        running   40508  08 Mar 16:06:25
worker-1-3   worker  localhost        running   40553  08 Mar 16:06:25
worker-1-4   worker  localhost        running   40616  08 Mar 16:06:25
worker-1-5   worker  localhost        running   40579  08 Mar 16:06:25
worker-1-6   worker  localhost        running   40528  08 Mar 16:06:25
worker-1-7   worker  localhost        running   40585  08 Mar 16:06:25
worker-1-8   worker  localhost        running   40540  08 Mar 16:06:25
worker-1-9   worker  localhost        running   40569  08 Mar 16:06:25
worker-1-10  worker  localhost        running   40573  08 Mar 16:06:25
worker-2-1   worker  localhost        running   40586  08 Mar 16:06:25
worker-2-2   worker  localhost        running   40649  08 Mar 16:06:25
worker-2-3   worker  localhost        running   40612  08 Mar 16:06:25
worker-2-4   worker  localhost        running   40613  08 Mar 16:06:25
worker-2-5   worker  localhost        running   40682  08 Mar 16:06:25
worker-2-6   worker  localhost        running   40653  08 Mar 16:06:26
worker-2-7   worker  localhost        running   40695  08 Mar 16:06:25
worker-2-8   worker  localhost        running   40699  08 Mar 16:06:25
worker-2-9   worker  localhost        running   40689  08 Mar 16:06:26
worker-2-10  worker  localhost        running   40688  08 Mar 16:06:26
worker-3-1   worker  localhost        running   40763  08 Mar 16:06:26
worker-3-2   worker  localhost        running   40736  08 Mar 16:06:26
worker-3-3   worker  localhost        running   40713  08 Mar 16:06:26
worker-3-4   worker  localhost        running   40774  08 Mar 16:06:26
worker-3-5   worker  localhost        running   40676  08 Mar 16:06:26
worker-3-6   worker  localhost        running   40810  08 Mar 16:06:26
worker-3-7   worker  localhost        running   40698  08 Mar 16:06:26
worker-3-8   worker  localhost        running   40745  08 Mar 16:06:26
worker-3-9   worker  localhost        running   40693  08 Mar 16:06:26
worker-3-10  worker  localhost        running   40773  08 Mar 16:06:26
worker-4-1   worker  localhost        running   40881  08 Mar 16:06:26
worker-4-2   worker  localhost        running   40828  08 Mar 16:06:26
worker-4-3   worker  localhost        running   40890  08 Mar 16:06:26
worker-4-4   worker  localhost        running   40840  08 Mar 16:06:26
worker-4-5   worker  localhost        running   40891  08 Mar 16:06:26
worker-4-6   worker  localhost        running   40958  08 Mar 16:06:26
worker-4-7   worker  localhost        running   40875  08 Mar 16:06:26
worker-4-8   worker  localhost        running   40977  08 Mar 16:06:26
worker-4-9   worker  localhost        running   40991  08 Mar 16:06:26
worker-4-10  worker  localhost        running   40973  08 Mar 16:06:26
worker-5-1   worker  localhost        running   40943  08 Mar 16:06:26
worker-5-2   worker  localhost        running   40910  08 Mar 16:06:26
worker-5-3   worker  localhost        running   40918  08 Mar 16:06:26
worker-5-4   worker  localhost        running   40944  08 Mar 16:06:26
worker-5-5   worker  localhost        running   40964  08 Mar 16:06:26
worker-5-6   worker  localhost        running   40996  08 Mar 16:06:26
worker-5-7   worker  localhost        running   40935  08 Mar 16:06:26
worker-5-8   worker  localhost        running   41019  08 Mar 16:06:26
worker-5-9   worker  localhost        running   41011  08 Mar 16:06:26
worker-5-10  worker  localhost        running   41063  08 Mar 16:06:26
worker-6-1   worker  localhost        running   41113  08 Mar 16:06:26
worker-6-2   worker  localhost        running   41058  08 Mar 16:06:26
worker-6-3   worker  localhost        running   41216  08 Mar 16:06:26
worker-6-4   worker  localhost        running   41187  08 Mar 16:06:26
worker-6-5   worker  localhost        running   41151  08 Mar 16:06:26
worker-6-6   worker  localhost        running   41126  08 Mar 16:06:26
worker-6-7   worker  localhost        running   41156  08 Mar 16:06:26
worker-6-8   worker  localhost        running   41165  08 Mar 16:06:26
worker-6-9   worker  localhost        running   41181  08 Mar 16:06:26
worker-6-10  worker  localhost        running   41234  08 Mar 16:06:26
worker-7-1   worker  localhost        running   41206  08 Mar 16:06:26
worker-7-2   worker  localhost        running   41250  08 Mar 16:06:26
worker-7-3   worker  localhost        running   41301  08 Mar 16:06:26
worker-7-4   worker  localhost        running   41342  08 Mar 16:06:26
worker-7-5   worker  localhost        running   41278  08 Mar 16:06:26
worker-7-6   worker  localhost        running   41338  08 Mar 16:06:26
worker-7-7   worker  localhost        running   41280  08 Mar 16:06:26
worker-7-8   worker  localhost        running   41331  08 Mar 16:06:26
worker-7-9   worker  localhost        running   41392  08 Mar 16:06:26
worker-7-10  worker  localhost        running   41401  08 Mar 16:06:26
worker-8-1   worker  localhost        running   41368  08 Mar 16:06:26
worker-8-2   worker  localhost        running   41402  08 Mar 16:06:26
worker-8-3   worker  localhost        running   41457  08 Mar 16:06:26
worker-8-4   worker  localhost        running   41432  08 Mar 16:06:26
worker-8-5   worker  localhost        running   41486  08 Mar 16:06:26
worker-8-6   worker  localhost        running   41493  08 Mar 16:06:26
worker-8-7   worker  localhost        running   41506  08 Mar 16:06:26
worker-8-8   worker  localhost        running   41524  08 Mar 16:06:26
worker-8-9   worker  localhost        running   41523  08 Mar 16:06:26
worker-8-10  worker  localhost        running   41554  08 Mar 16:06:26
worker-9-1   worker  localhost        running   41560  08 Mar 16:06:26
worker-9-2   worker  localhost        running   41552  08 Mar 16:06:26
worker-9-3   worker  localhost        running   41577  08 Mar 16:06:26
worker-9-4   worker  localhost        running   41566  08 Mar 16:06:26
worker-9-5   worker  localhost        running   41575  08 Mar 16:06:26
worker-9-6   worker  localhost        running   41576  08 Mar 16:06:26
worker-9-7   worker  localhost        running   41583  08 Mar 16:06:26
worker-9-8   worker  localhost        running   41591  08 Mar 16:06:26
worker-9-9   worker  localhost        running   41585  08 Mar 16:06:26
worker-9-10  worker  localhost        running   41587  08 Mar 16:06:26
```

**to run zeek as service**

```
zeek@zeek-sensor:~$ sudo vim /etc/systemd/system/zeek.service
[Unit]
Description=Zeek
[Service]
User=zeek
Type=forking
Restart=on-failure
RestartSec=5
StartLimitAction=reboot
ExecStart=/opt/zeek/bin/zeekctl deploy
ExecStop=/opt/zeek/bin/zeekctl stop
[Install]
WantedBy=multi-user.target

```

```
zeek@zeek-sensor:~$ sudo systemctl enable zeek.service
zeek@zeek-sensor:~$ sudo systemctl status zeek
● zeek.service - Zeek
     Loaded: loaded (/etc/systemd/system/zeek.service; enabled; preset: enabled)
     Active: active (running) since Sat 2025-03-08 16:06:33 UTC; 3min 1s ago
    Process: 39947 ExecStart=/opt/zeek/bin/zeekctl deploy (code=exited, status=0/SUCCESS)
      Tasks: 1312 (limit: 38235)
     Memory: 7.8G (peak: 8.0G)
        CPU: 2min 3.985s
     CGroup: /system.slice/zeek.service
             ├─40024 /usr/bin/bash /opt/zeek/share/zeekctl/scripts/run-zeek -1 -U .status -p zeekctl -p zeekctl-live -p local -p logger-1 local.zeek zeekctl base/frameworks/cluster zeekctl/auto
             ├─40030 /opt/zeek/bin/zeek -U .status -p zeekctl -p zeekctl-live -p local -p logger-1 local.zeek zeekctl base/frameworks/cluster zeekctl/auto
             ├─40077 /usr/bin/bash /opt/zeek/share/zeekctl/scripts/run-zeek -1 -U .status -p zeekctl -p zeekctl-live -p local -p manager local.zeek zeekctl base/frameworks/cluster zeekctl/auto
             ├─40083 /opt/zeek/bin/zeek -U .status -p zeekctl -p zeekctl-live -p local -p manager local.zeek zeekctl base/frameworks/cluster zeekctl/auto
             ├─40129 /usr/bin/bash /opt/zeek/share/zeekctl/scripts/run-zeek -1 -U .status -p zeekctl -p zeekctl-live -p local -p proxy-1 local.zeek zeekctl base/frameworks/cluster zeekctl/auto
             ├─40135 /opt/zeek/bin/zeek -U .status -p zeekctl -p zeekctl-live -p local -p proxy-1 local.zeek zeekctl base/frameworks/cluster zeekctl/auto
             ├─40350 /usr/bin/bash /opt/zeek/share/zeekctl/scripts/run-zeek -1 -i eno2 -U .status -p zeekctl -p zeekctl-live -p local -p worker-1-2 local.zeek zeekctl base/frameworks/cluster zeekctl/auto
             ├─40359 /usr/bin/bash /opt/zeek/share/zeekctl/scripts/run-zeek -1 -i eno2 -U .status -p zeekctl -p zeekctl-live -p local -p worker-1-1 local.zeek zeekctl base/frameworks/cluster zeekctl/auto
             ├─40362 /usr/bin/bash /opt/zeek/share/zeekctl/scripts/run-zeek -1 -i eno2 -U .status -p zeekctl -p zeekctl-live -p local -p worker-1-6 local.zeek zeekctl base/frameworks/cluster zeekctl/auto
             ├─40373 /usr/bin/bash /opt/zeek/share/zeekctl/scripts/run-zeek -1 -i eno2 -U .status -p zeekctl -p zeekctl-live -p local -p worker-1-5 local.zeek zeekctl base/frameworks/cluster zeekctl/auto
             ├─40381 /usr/bin/bash /opt/zeek/share/zeekctl/scripts/run-zeek -1 -i eno2 -U .status -p zeekctl -p zeekctl-live -p local -p worker-1-4 local.zeek zeekctl base/frameworks/cluster zeekctl/auto
             ├─40383 /usr/bin/bash /opt/zeek/share/zeekctl/scripts/run-zeek -1 -i eno2 -U .status -p zeekctl -p zeekctl-live -p local -p worker-1-8 local.zeek zeekctl base/frameworks/cluster zeekctl/auto
             ├─40388 /usr/bin/bash /opt/zeek/share/zeekctl/scripts/run-zeek -1 -i eno2 -U .status -p zeekctl -p zeekctl-live -p local -p worker-1-3 local.zeek zeekctl base/frameworks/cluster zeekctl/auto
             ├─40405 /usr/bin/bash /opt/zeek/share/zeekctl/scripts/run-zeek -1 -i eno2 -U .status -p zeekctl -p zeekctl-live -p local -p worker-1-9 local.zeek zeekctl base/frameworks/cluster zeekctl/auto
             ├─40414 /usr/bin/bash /opt/zeek/share/zeekctl/scripts/run-zeek -1 -i eno2 -U .status -p zeekctl -p zeekctl-live -p local -p worker-1-7 local.zeek zeekctl base/frameworks/cluster zeekctl/auto
             ├─40421 /usr/bin/bash /opt/zeek/share/zeekctl/scripts/run-zeek -1 -i eno2 -U .status -p zeekctl -p zeekctl-live -p local -p worker-2-1 local.zeek zeekctl base/frameworks/cluster zeekctl/auto
             ├─40422 /usr/bin/bash /opt/zeek/share/zeekctl/scripts/run-zeek -1 -i eno2 -U .status -p zeekctl -p zeekctl-live -p local -p worker-2-2 local.zeek zeekctl base/frameworks/cluster zeekctl/auto
             ├─40426 /usr/bin/bash /opt/zeek/share/zeekctl/scripts/run-zeek -1 -i eno2 -U .status -p zeekctl -p zeekctl-live -p local -p worker-1-10 local.zeek zeekctl base/frameworks/cluster zeekctl/auto
             ├─40429 /usr/bin/bash /opt/zeek/share/zeekctl/scripts/run-zeek -1 -i eno2 -U .status -p zeekctl -p zeekctl-live -p local -p worker-2-3 local.zeek zeekctl base/frameworks/cluster zeekctl/auto
             ├─40433 /usr/bin/bash /opt/zeek/share/zeekctl/scripts/run-zeek -1 -i eno2 -U .status -p zeekctl -p zeekctl-live -p local -p worker-2-5 local.zeek zeekctl base/frameworks/cluster zeekctl/auto
             ├─40466 /usr/bin/bash /opt/zeek/share/zeekctl/scripts/run-zeek -1 -i eno2 -U .status -p zeekctl -p zeekctl-live -p local -p worker-2-4 local.zeek zeekctl base/frameworks/cluster zeekctl/auto
             ├─40475 /usr/bin/bash /opt/zeek/share/zeekctl/scripts/run-zeek -1 -i eno2 -U .status -p zeekctl -p zeekctl-live -p local -p worker-2-7 local.zeek zeekctl base/frameworks/cluster zeekctl/auto
             ├─40488 /opt/zeek/bin/zeek -i eno2 -U .status -p zeekctl -p zeekctl-live -p local -p worker-1-1 local.zeek zeekctl base/frameworks/cluster zeekctl/auto
             ├─40508 /opt/zeek/bin/zeek -i eno2 -U .status -p zeekctl -p zeekctl-live -p local -p worker-1-2 local.zeek zeekctl base/frameworks/cluster zeekctl/auto
             ├─40518 /usr/bin/bash /opt/zeek/share/zeekctl/scripts/run-zeek -1 -i eno2 -U .status -p zeekctl -p zeekctl-live -p local -p worker-2-8 local.zeek zeekctl base/frameworks/cluster zeekctl/auto
             ├─40525 /usr/bin/bash /opt/zeek/share/zeekctl/scripts/run-zeek -1 -i eno2 -U .status -p zeekctl -p zeekctl-live -p local -p worker-3-3 local.zeek zeekctl base/frameworks/cluster zeekctl/auto
             ├─40528 /opt/zeek/bin/zeek -i eno2 -U .status -p zeekctl -p zeekctl-live -p local -p worker-1-6 local.zeek zeekctl base/frameworks/cluster zeekctl/auto
             ├─40534 /usr/bin/bash /opt/zeek/share/zeekctl/scripts/run-zeek -1 -i eno2 -U .status -p zeekctl -p zeekctl-live -p local -p worker-2-6 local.zeek zeekctl base/frameworks/cluster zeekctl/auto
             ├─40538 /usr/bin/bash /opt/zeek/share/zeekctl/scripts/run-zeek -1 -i eno2 -U .status -p zeekctl -p zeekctl-live -p local -p worker-2-9 local.zeek zeekctl base/frameworks/cluster zeekctl/auto
             ├─40540 /opt/zeek/bin/zeek -i eno2 -U .status -p zeekctl -p zeekctl-live -p local -p worker-1-8 local.zeek zeekctl base/frameworks/cluster zeekctl/auto
             ├─40544 /usr/bin/bash /opt/zeek/share/zeekctl/scripts/run-zeek -1 -i eno2 -U .status -p zeekctl -p zeekctl-live -p local -p worker-2-10 local.zeek zeekctl base/frameworks/cluster zeekctl/auto
             ├─40553 /opt/zeek/bin/zeek -i eno2 -U .status -p zeekctl -p zeekctl-live -p local -p worker-1-3 local.zeek zeekctl base/frameworks/cluster zeekctl/auto
             ├─40554 /usr/bin/bash /opt/zeek/share/zeekctl/scripts/run-zeek -1 -i eno2 -U .status -p zeekctl -p zeekctl-live -p local -p worker-3-1 local.zeek zeekctl base/frameworks/cluster zeekctl/auto
             ├─40569 /opt/zeek/bin/zeek -i eno2 -U .status -p zeekctl -p zeekctl-live -p local -p worker-1-9 local.zeek zeekctl base/frameworks/cluster zeekctl/auto
             ├─40573 /opt/zeek/bin/zeek -i eno2 -U .status -p zeekctl -p zeekctl-live -p local -p worker-1-10 local.zeek zeekctl base/frameworks/cluster zeekctl/auto
             ├─40579 /opt/zeek/bin/zeek -i eno2 -U .status -p zeekctl -p zeekctl-live -p local -p worker-1-5 local.zeek zeekctl base/frameworks/cluster zeekctl/auto
             ├─40585 /opt/zeek/bin/zeek -i eno2 -U .status -p zeekctl -p zeekctl-live -p local -p worker-1-7 local.zeek zeekctl base/frameworks/cluster zeekctl/auto
             ├─40586 /opt/zeek/bin/zeek -i eno2 -U .status -p zeekctl -p zeekctl-live -p local -p worker-2-1 local.zeek zeekctl base/frameworks/cluster zeekctl/auto
             ├─40587 /usr/bin/bash /opt/zeek/share/zeekctl/scripts/run-zeek -1 -i eno2 -U .status -p zeekctl -p zeekctl-live -p local -p worker-3-5 local.zeek zeekctl base/frameworks/cluster zeekctl/auto
```

zeek logs should be generated in /opt/zeek/logs/current

```
zeek@zeek-sensor:~$ ll /opt/zeek/logs/current/
total 16324
drwxr-xr-x   2 zeek zeek     4096 Mar  8 16:07 ./
drwxr-xr-x 109 zeek zeek     4096 Mar  8 16:10 ../
-rw-r--r--   1 zeek zeek    71295 Mar  8 16:06 broker.log
-rw-r--r--   1 zeek zeek    10011 Mar  8 16:07 capture_loss.log
-rw-r--r--   1 zeek zeek    64613 Mar  8 16:06 cluster.log
-rw-r--r--   1 zeek zeek      115 Mar  8 16:06 .cmdline
-rw-r--r--   1 zeek zeek   879565 Mar  8 16:11 conn.log
-rw-r--r--   1 zeek zeek  3434416 Mar  8 16:11 dns.log
-rw-r--r--   1 zeek zeek      359 Mar  8 16:06 .env_vars
-rw-r--r--   1 zeek zeek    39576 Mar  8 16:06 loaded_scripts.log
-rw-r--r--   1 zeek zeek    20341 Mar  8 16:07 notice.log
-rw-r--r--   1 zeek zeek        0 Mar  8 16:06 packet_filter.log
-rw-r--r--   1 zeek zeek        6 Mar  8 16:06 .pid
-rw-r--r--   1 zeek zeek       61 Mar  8 16:06 .startup
-rw-r--r--   1 zeek zeek    41118 Mar  8 16:06 stats.log
-rwxr-xr-x   1 zeek zeek       19 Mar  8 16:06 .status*
-rw-r--r--   1 zeek zeek        0 Mar  8 16:06 stderr.log
-rw-r--r--   1 zeek zeek      204 Mar  8 16:06 stdout.log
-rw-r--r--   1 zeek zeek 12104260 Mar  8 16:10 telemetry.log
```

**Zeek control crom : ZeekControl features a cron command to check for and restart crashed nodes and to perform other maintenance tasks.  To take advantage of this, let’s set up a cron job.**

```
zeek@zeek-sensor:~$ crontab -e
*/5 * * * * /opt/zeek/bin/zeekctl cron
```

###Installation has been succeffully completed, we will complete our guide in next lesson with How to send Zeek logs to Splunk Server ......
