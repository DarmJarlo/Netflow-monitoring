# SoSe22-ACS6-Lab
KOM Lab- ACS6

## Description
This project implements the flow observation approach to monitor the network traffic. It supports NetFlow, IPFIX and sFlow. The project consists of four components, namely network generator, flow exporter, flow collector and analyzation. Each component has several offerings which you can make use of a combination of them. To run our network monitroing system, you need a Ubuntu system.

## Tools
Network generator
- [Mininet](https://github.com/mininet/mininet/)

Flow exporter
- [ovs-vsctl command line](http://www.openvswitch.org/support/dist-docs/ovs-vsctl.8.txt) (NetFlow, IPFIX, sFlow)
- pmacctd daemon ([pmacctd project](https://github.com/pmacct/pmacct/)) (NetFlow, IPFIX, sFlow)

Flow collector
- [sFlowTrend](https://inmon.com/products/sFlowTrend.php) (sFlow)
- nfacctd daemon ([pmacctd project](https://github.com/pmacct/pmacct/)) (NetFlow, IPFIX)
- sfacctd daemon ([pmacctd project](https://github.com/pmacct/pmacct/)) (sFlow)

Analyzation
- [Grafana](https://grafana.com/)
- [sFlowTrend](https://inmon.com/products/sFlowTrend.php)

Data storage
- [MySQL](https://www.mysql.com/)
- [Apache Kafka](https://kafka.apache.org/)

Replay traffic dataset
- [tcpreplay](https://tcpreplay.appneta.com/)

DoS Attack Simulator
- [Hping3](http://www.hping.org)


## Installation
### Mininet
```
git clone https://github.com/mininet/mininet
cd mininet
git tag  # list available versions
git checkout -b mininet-2.3.0 2.3.0  # or whatever version you wish to install
cd ..
mininet/util/install.sh -a  # enable -a option, install everything that is included in the Mininet like Open vSwitch   
```
For more options of installation, you can refer to the [Mininet Download](http://mininet.org/download/).

### ovs-vsctl command line
If you follow the steps of Mininet installation, you already have this package in your machine.

### Pmacct
Pmacct project can be extented with mutiple plugins. Before you install pmacct, you have to install the required packages according to your needs. To enable pmacct to send flows to Kafka broker, you have to install these two packages. Libjansson is used to define the flow data in JSON format to Kafka broker.
- [librdkafka](https://github.com/edenhill/librdkafka/)
```
sudo apt install librdkafka-dev
```
- [libjansson](https://github.com/akheron/jansson)
```
git clone https://github.com/akheron/jansson
cd jansson
autoreconf -fi
./configure
make
sudo make install
```

After you installed all required dependencies, you can run the following commands to install pmacct.
```
git clone https://github.com/pmacct/pmacct.git
cd pmacct
autoreconf -fi
./configure --enable-kafka --enable-jansson --enable-mysql  # the enable options depends on your needs
make
sudo make install
```

### sFlowTrend
sFlowTrend is written in Java. It requires Java version 8 or later. If you don't have Java in your machine, please refer to [Oracle Java](http://www.java.com/). sFlowTrend supports Windows and Linux systems, you can download the current version from [sFlowTrend website](https://inmon.com/products/sFlowTrend.php). We only test sFlowTrend in Windows system, you can download [sFlowTrend Windows 64-bit wiht 64-bit JRE](https://inmon.com/products/sFlowTrend/downloads/sFlowTrend-windows-x64-7_3_1.exe) direct here. To install sFlowTrend, just run the file by double-clicking it. If you want to install sFlowTrend in Linux, you can follow the [installation instructions](https://inmon.com/products/sFlowTrend/help/html/installation.html).

### Apache Kafka
Apache Kafka requires Java as well. If you don't have Java in your Ubuntu system, you can run the below commands to install OpenJDK.
```
sudo apt update
sudo apt intall default-jdk
```
Once you installed OpenJDK successfully, you can check the default version.
```
java --version
```
Download the current Apache Kafka binary files from [Apache Kafka Download](https://kafka.apache.org/downloads). Alternatively, you can download it with wget.
```
wget https://dlcdn.apache.org/kafka/3.2.0/kafka_2.13-3.2.0.tgz 
```
Extract the file and move it to /usr/local/kafka folder.
```
tar xzf kafka_2.13-3.2.0.tgz 
sudo mv kafka_2.13-3.2.0 /usr/local/kafka 
```
To start and stop the Kafka services, you need to create systemd unit files for the Zookeeper and Kafka services.
First, you have to create a systemd unit file for Zookeeper.
```
vim /etc/systemd/system/zookeeper.service
```
Add the following content to this file:
```
[Unit]
Description=Apache Zookeeper server
Documentation=http://zookeeper.apache.org
Requires=network.target remote-fs.target
After=network.target remote-fs.target

[Service]
Type=simple
ExecStart=/usr/local/kafka/bin/zookeeper-server-start.sh /usr/local/kafka/config/zookeeper.properties
ExecStop=/usr/local/kafka/bin/zookeeper-server-stop.sh
Restart=on-abnormal

[Install]
WantedBy=multi-user.target
```
Then create a systemd unit file for the Kafka service
```
vim /etc/systemd/system/kafka.service
```
Add the following content. If you have the different JAVA_HOME path, please replace the path of JAVA_HOME.
```
[Unit]
Description=Apache Kafka Server
Documentation=http://kafka.apache.org/documentation.html
Requires=zookeeper.service

[Service]
Type=simple
Environment="JAVA_HOME=/usr/lib/jvm/java-1.11.0-openjdk-amd64"
ExecStart=/usr/local/kafka/bin/kafka-server-start.sh /usr/local/kafka/config/server.properties
ExecStop=/usr/local/kafka/bin/kafka-server-stop.sh

[Install]
WantedBy=multi-user.target
```
Reload the systemd daemon to apply new changes.
```
systemctl daemon-reload
```

### tcpreplay
```
sudo apt update
sudo apt install tcpreplay
```

### Hping3
```
sudo apt-get install hping3
```


### MySQL
```
sudo apt install mysql-server
sudo systemctl start mysql.service
```
set up password for root
```
sudo mysql
MySQL>ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'password';
```
### Grafana
```
sudo apt-get install -y adduser libfontconfig1
wget https://dl.grafana.com/enterprise/release/grafana-enterprise_8.5.2_amd64.deb
sudo dpkg -i grafana-enterprise_8.5.2_amd64.deb
```

## Quick Start
To start monitoring traffic, you need to create a network with Mininet. Run the following command to create a simpel default network with two hosts, one switch and one controller.
```
sudo mn
```

A simple way to generate traffic in the network is using **ping** command to create ICMP echo request and echo reply traffic. Run the following command in the Mininet terminal.
```
h1 ping h2
```
Then you will see the ICMP traffic in the terminal.


On the Mininet terminal, you can flood packets to h2 to simulate DoS attack.Run the following command in the Mininet terminal.
```
h1 hping3 -c 10000 -S --flood --rand-source -V h2
```
On the Mininet terminal, stop hping3 by ctrl + C.
Then you could see the traffic in the terminal.
To see the process you can run tcupdump in each host and monitor the sending and receiving packets.

In addition, you can replay the traffic data in PCAP fromat with tcpreplay. We uploaded a sample PCAP dataset file [SAT-03-11-2018_06](SAT-03-11-2018_06) from [CIC-DDoS2019](https://www.unb.ca/cic/datasets/ddos-2019.html) in the repository. Feel free to use it to do your test. First, you have to modify the IP source and destination addresses of the traffic. In the default network you created before, the IP addresses of host 1 and host 2 are 10.0.0.1 and 10.0.0.2. You have to input the original and output PCAP file in the following command. (Reminder: you may have to rename the given PCAP file with .pcap suffix) 
```
tcprewrite --infile=original.pcap --outfile=modified.pcap --srcipmap=0.0.0.0/0:10.0.0.1 --dstipmap=0.0.0.0/0:10.0.0.2

```
You will see a modified PCAP file in the same folder. Then, you have to recalculate the checksums with the below command.
```
tcprewrite --infile=modified.pcap --outfile=result.pcap --fixcsum
```
The final PCAP file is now generated as result.pcap. You have to replay that file in a host terminal. Open the host 1 terminal with **xterm**. Run the below command in the Mininet terminal.
```
xterm h1
```
Then, you have to replay the result.pcap file with tcpreplay. Run the following command in host 1 terminal. Host 1 will send the traffic to host 2.
```
# -i select a network interface. 
# -x the replay speed. Here, 1.0 is the orignal speed. You can give the other number to speed up or slow down the replay.
# -K preload file into memory to increase speed.
tcpreplay -i h1-eth0 -x 1.0 -K result.pcap
```
So far, a network with traffic is built successfully. Next you can start to monitor the traffic between the two hosts. You can combine the different tools of each component. Here, we give several options of the combination.

### Option 1: ovs-vsctl + sFlowTrend + sFlow
First, you have to start sFlowTrend in a web browser with the URL http://[hostname]:8087/sflowtrend. The hostname is the IP address of your machine, or you can click http://localhost:8087/sflowtrend. Then, you will see the following screen. 
![sFlowTrend dashboard screen](/images/sflowtrend.png)

You can click the gear icon ![icon](/images/sflowtrend_icon.png) on the top right to open the configuration menu. The sFlow collector address and port number is in the "System configuration" item. 

Run the following command to enable the exporter to send sampled packets passing through switch s1 to the sFlowTrend.
```
# agent: the ip address of the agent, set to the interface eth1
# target: the sFlow collector address
# header: byte of the header
# sampling: sampling rate. here, we set 64 means sampling packets at 1-in-64
# polling: poll counters every 10 seconds
sudo ovs-vsctl -- --id=@s create sFlow agent=eth1 target=\"[IP address of sFlow collector]:[port number]\" header=128 sampling=64 polling=10 -- set Bridge s1 sflow=@s
```
sFlowTrend porvides various predefined charts to visualize traffic. You can click the "Network" tab bar on the sFlowTrend to see the status of the traffic in the network. 

### Option 2: Pmacct + sFlowTrend + sFlow
You can use pmacctd daemon to export data to sFlowTrend as well. In this case, a configuration file is required. We uploaded a template of pmacctd configuration file [pmacctd_sflow.conf](pmacct_configuration_files/pmacctd_sflow.conf) in the repository. You can simply download it and modify the key of **sfprobe_receiver** to your sFlowTrend server address. Excute the below command.
```
sudo pmacctd -f [path of pmacctd_sflow.conf]
```
Turn to sFlowTrend to see the results.

### Option 3: Pmacct + Kafka + NetFlow/IPFIX
In this option, pmacctd daemon exports flows to nfacctd daemon using NetFlow or IPFIX. The nfacctd daemon sends the collected flows to the Kafka broker. You can find the configuration files [pmacctd.conf](pmacct_configuration_files/pmacctd.conf) and [nfacctd_kafka.conf](pmacct_configuration_files/nfacctd_kafka.conf) for pmacctd daemon and nfacctd daemon. In the following initialization, pmacct and Kafka server are running locally. 

Start Zookeeper and Kafka server.
```
sudo systemctl start zookeeper
sudo systemctl start kafka
```
Check the running status of Kafka
```
sudo systemctl status kafka
```
Create a topic in Kafka. If you have a different path of Kafka, please replace the path in the following command.
```
# create a topic "mininet". The topic has to be the same as the value of kafka_topic in the configuration of nfacctd daemon.
# replicate events only once. use one partition
/usr/local/kafka/bin/kafka-topics.sh --create --bootstrap-server localhost:9092 --replication-factor 1 --partitions 1 --topic mininet
```

Start the Kafka consumer to extract events from the Kafka broker.
```
/usr/local/kafka/bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic mininet --from-beginning
```

Enable pmacctd daemon and nfacctd daemon.
```
sudo pmacctd -f [path of pmacctd.conf]
sudo nfacctd -f [path of nfacctd_kafka.conf]
```
You will see the collected flows of nfacctd daemon in the Kafka consumer terminal.


### Option 4: Pmacct + MySQL + Grafana
First, make sure plugin of MySQL is enabled, when you install Pmacct.

Start the grafana server and set password as newpasswd for admin
```
sudo service grafana-server start
sudo grafana-cli admin reset-admin-password newpasswd
```
To start the MySQL
```
sudo service mysql start
mysql -u root -p
```
After login the MySQL, create database, user, and also a kind of table into MySQL 
```
drop database if exists pmacct;
create database pmacct;

use pmacct;

drop table if exists acct_v8; 
create table acct_v8 (
    agent_id INT(4) UNSIGNED NOT NULL,
    class_id CHAR(16) NOT NULL,
    mac_src CHAR(17) NOT NULL,
    mac_dst CHAR(17) NOT NULL,
	
    timestamp_start_residual VARCHAR(100) NOT NULL,
    timestamp_end_residual VARCHAR(100) NOT NULL,

    vlan INT(2) UNSIGNED NOT NULL,
    as_src INT(4) UNSIGNED NOT NULL,
    as_dst INT(4) UNSIGNED NOT NULL,
    ip_src CHAR(45) NOT NULL,
    ip_dst CHAR(45) NOT NULL,
    port_src INT(2) UNSIGNED NOT NULL,
    port_dst INT(2) UNSIGNED NOT NULL,
    tcp_flags INT(4) UNSIGNED NOT NULL,
    ip_proto CHAR(6) NOT NULL, 
    tos INT(4) UNSIGNED NOT NULL, 
    packets INT UNSIGNED NOT NULL,
    bytes BIGINT UNSIGNED NOT NULL,
    flows INT UNSIGNED NOT NULL,
    stamp_inserted DATETIME NOT NULL,
    stamp_updated DATETIME,
    timestamp_start DATETIME NOT NULL,
    timestamp_end DATETIME NOT NULL,"
    PRIMARY KEY (agent_id, class_id, mac_src, mac_dst, vlan, as_src, as_dst, ip_src, ip_dst, port_src, port_dst, ip_proto, tos, stamp_inserted,timestamp_start, timestamp_end)
);
CREATE USER 'pmacct'@'localhost' IDENTIFIED BY 'Yourpassword1!';
GRANT ALL PRIVILEGES ON pmacct.* TO pmacct@localhost;

```


Then start the traffic and run the exporter and collector as follows.

```
sudo pmacctd -f [path of pmacctd.conf]
sudo nfacctd -f [path of nfacctd_mysql.conf]
```

And then go to mysql to have a look at the data

```
select * from acct_v8
```

As for the grafana, use your browser and open http://localhost:3000 ,there you could have the Grafana Webpage，then you could login with user admin and password newpasswd
First klick "add the data source", select MySQL and fill out the configuration with your HOST, database and User. 

![Grafana dashboard screen](/images/grafana1.jpg)

![Add data source](/images/grafana2.png)

Then click "create your first dashboard" and then "add a new panel", where you could select data through writing the SQL script

![Graph setup](/images/grafanaexample.png)

For example, you could write your SQL script like:
```
SELECT
  $__timeGroupAlias(stamp_inserted,$__interval),
  sum(packets) AS "packets"
FROM acct_v8
WHERE
  $__timeFilter(stamp_inserted) AND
  ip_dst = '10.0.0.2'
GROUP BY 1
ORDER BY $__timeGroup(stamp_inserted,$__interval)
```
to show the packets number in a period of time

After the Setup is finished, click " save" 

Finally we get a dashboard like:

![Graph Result](/images/grafbytespng.png)











# Netflow-monitoring
