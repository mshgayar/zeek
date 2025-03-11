## Zeek integration with Splunk ##
Here ww will configure the zeek sensor to send zeek logs to a splunk SIEM for more threat hunting purposes 

## Steps to Configure Zeek sensor to forward logs to SPLUNK SIEM
* Configure Zeek to output logs in JSON format for consumption by Splunk.
* Create an index in Splunk for Zeek data.
* Installing and configuring the Corelight For Splunk app to index and parse Zeek logs in Splunk.
* Create a splunk user to run the Splunk Universal Forwarder.
* Installing and configuring a Splunk Universal Forwarder to send Zeek logs to a Splunk instance.


Current Hardware : 

	1. Zeek Sensor :   192.168.1.121
	2. SPLUNK Server : 192.168.1.148

##Part 1 : Splunk Forwrder Installation##

**Configure Zeek to output logs in JSON format for consumption by Splunk.**

```
zeek@zeek-sensor:~$ vim /opt/zeek/share/zeek/site/local.zeek
#OUTPUT to JSON
@load policy/tuning/json-logs.zeek
```
restart zeek ``$ zeekctl deploy``

**Create Index in SPLUNK SIEM named "zeek"**

* ``Login to the splunk server : http://192.168.1.148:8000/en-US/manager/launcher/data/indexes# ``
*  ``create index with name : zeek``

**Configure Splunk SIEM to receive data from the zeek sensor**
``Login to Splunk> Settings>Forwarding and Receiving > configure receiving >Add port 9997``

**Install corelight app & configure resource type=corelight**
* ``Install app corelight for splunk from : http://192.168.1.148:8000/en-US/manager/launcher/appsremote?offset=0&count=20&order=latest``

**Create a Splunk user to run the Splunk Universal Forwarder.**

```
$ sudo groupadd splunk 
$ sudo useradd splunk -g splunk -G zeek
$ su – splunk
$ wget -O splunkforwarder-9.4.1-e3bdab203ac8-linux-amd64.deb https://download.splunk.com/products/universalforwarder/releases/9.4.1/linux/splunkforwarder-9.4.1-e3bdab203ac8-linux-amd64.deb

```

**Installing a Splunk Universal Forwarder to send Zeek logs to a Splunk instance.**

```
$ sudo dpkg -i splunkforwarder-9.4.1-e3bdab203ac8-linux-amd64.deb
$ sudo chown -R splunk:splunk /opt/splunkforwarder
$ su –splunk
$ /opt/splunkforwarder/bin/splunk start --accept-license
$ ./splunk stop
```

##Part 2 : Splunk Forwarder Configurations :##

**Remove default processing limit**

```
vim/opt/splunkforwarder/etc/system/local/limits.conf
[thruput]
maxKBps = 0 # means unlimited

```

**Configure the inputs to include all zeek logs**

```
1. Here to configure the SPLUNK forwarder to send all logs under /opt/zeek/logs/current/ 
1. THose logs will be parsed in SPLUNK under index=zeek and sourcetype=corelight because we have installed corelight for splunk app, 


Vim /opt/splunkforwarder/etc/system/local/inputs.conf

[default]
host = zeek-sensor
[monitor:///opt/zeek/logs/current/conn.log]
_TCP_ROUTING = *
index = zeek
sourcetype = corelight_conn:json
[monitor:///opt/zeek/logs/current/dns.log]
_TCP_ROUTING = *
index = zeek
sourcetype = corelight_dns
[monitor:///opt/zeek/logs/current/software.log]
_TCP_ROUTING = *
index = zeek
sourcetype = corelight_software
[monitor:///opt/zeek/logs/current/smtp.log]
_TCP_ROUTING = *
index = zeek
sourcetype = corelight_smtp
[monitor:///opt/zeek/logs/current/ssl.log]
_TCP_ROUTING = *
index = zeek
sourcetype = corelight_ssl
[monitor:///opt/zeek/logs/current/ssh.log]
_TCP_ROUTING = *
index = zeek
sourcetype = corelight_ssh
[monitor:///opt/zeek/logs/current/x509.log]
_TCP_ROUTING = *
index = zeek
sourcetype = corelight_x509
[monitor:///opt/zeek/logs/current/ftp.log]
_TCP_ROUTING = *
index = zeek
sourcetype = corelight_ftp
[monitor:///opt/zeek/logs/current/http.log]
_TCP_ROUTING = *
index = zeek
sourcetype = corelight_http
[monitor:///opt/zeek/logs/current/rdp.log]
_TCP_ROUTING = *
index = zeek
sourcetype = corelight_rdp
[monitor:///opt/zeek/logs/current/smb_files.log]
_TCP_ROUTING = *
index = zeek
sourcetype = corelight_files
[monitor:///opt/zeek/logs/current/smb_mapping.log]
_TCP_ROUTING = *
index = zeek
sourcetype = corelight_smb_mapping
[monitor:///opt/zeek/logs/current/snmp.log]
_TCP_ROUTING = *
index = zeek
sourcetype = corelight_snmp
[monitor:///opt/zeek/logs/current/sip.log]
_TCP_ROUTING = *
index = zeek
sourcetype = corelight_sip
[monitor:///opt/zeek/logs/current/files.log]
_TCP_ROUTING = *
index = zeek
sourcetype = corelight_files

```

**Add SPLUNK SIEM IP to the splunkforwader**

```
splunk@zeek-sensor:~$ vim /opt/splunkforwarder/etc/system/local/outputs.conf

[tcpout]
defaultGroup = default-autolb-group

[tcpout-server://192.168.1.148:9997]

[tcpout:default-autolb-group]
disabled = false
server = 192.168.1.148:9997,192.168.1.148:9997

[tcpout-server://192.168.1.148:9997]
```

**Start SplunkForwarder**

```
$ /opt/splunkforwarder/bin/splunk start
```

**Login to SPLUNK http://192.168.1.148:8000**

try to search in splunk events to make sure that events are reterived and displayed in json format

```
index=zeek sourcetype="corelight_conn" | where (src LIKE "192.168.1.138")
```
![alt text](https://github.com/mshgayar/zeek/blob/main/splunk1.png)


### Congratulations the zeek sensor now is configured to send logs to SPLUNK SERVER http://192.168.1.148:8000 ###
