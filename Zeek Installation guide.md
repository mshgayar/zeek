**Zeek Cluster Installation Requirements**

A Zeek cluster is required for high-performance network monitoring, especially when dealing with high-throughput network traffic. Below are the hardware, software, and configuration requirements to set up a Zeek cluster.

### . Hardware Requirements
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
