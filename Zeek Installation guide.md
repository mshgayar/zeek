**Zeek Cluster Installation Requirements**

A Zeek cluster is required for high-performance network monitoring, especially when dealing with high-throughput network traffic. Below are the hardware, software, and configuration requirements to set up a Zeek cluster.

### 1. Hardware Requirements
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

--
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

--
