# IVCM8260-2018-PacketCapture
Packet Capture using combination of Cisco Catalyst 3750-E, Moloch, and dsniff. 

For detailed information, please refer to [k-space](https://wiki.k-space.ee/index.php?title=Packet_capture)

**Current project status:** Moloch operates successfully with Elasticsearch on seperate server

**Problems:**

**Plans for next week:** Moloch plugins

## Members ##
**Supervisors:** Toomas Lepik, Lauri Võsandi

**Students:** Ilkan Mustafa Sahan IVCM, ChengYu Lu 184679IVCM

## Contents ##
- [Cisco Catalyst 3750-E](#cisco-catalyst-3750-E)
- [Moloch](#moloch)
- [dsniff](#dsniff)

## Cisco Catalyst 3750-E ##
### Connect Linux to Cisco serial console
1. Install cu: `$ sudo apt-get install cu`

2. Use USB to serial converter and Cisco console cable to connect our laptop to switch and install driver

3. Identify the port of the serial cable: `$ sudo dmesg | grep -i tty`

4. Connect to a switch with cu command: `$ sudo cu -l /dev/device -s baud-rate-speed`

5. Cisco prompt for configuration

### Catalyst Switched Port Analyzer (SPAN) Configuration ###
1. Speed up switch port initialization process:

`Switch> enable`

`Switch# config terminal`

`Switch(config)# int range fastEthernet 0/1 - 24`

`Switch(config-if-range)# switchport mode access`

`Switch(config-if-range)# spanning-tree portfast`

2. Creating a SPAN Session

`Switch> enable`

`Switch# config terminal`

`Switch(config)# monitor session 1 source interface fastEthernet 0/25`

`Switch(config)# monitor session 1 destination interface fastEthernet 0/26`

`Switch(config)# exit`

`Switch# copy run start`

## Moloch ##
### Ubuntu server configuration ###
Setting listening port to promiscuous mode:

`$ sudo ip link set [port] up`

`$ sudo ip link set [port] promisc on`

### Moloch Installation ##
`$ sudo apt-get update`

`$ sudo apt-get install npm python-software-properties oracle-java8-installer curl apt-transport-https`

`$ curl -sL -o https://deb.nodesource.com/setup_8.x | sudo -E bash -`

`$ sudo apt-get install nodejs`

`$ wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add -`

`$ echo "deb https://artifacts.elastic.co/packages/6.x/apt stable main" | sudo tee -a /etc/apt/sources.list.d/elastic-6.x.list`

`$ wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-6.4.2.deb`

`$ sudo dpkg -i elasticsearch-6.4.2.deb`

`$ sudo /bin/systemctl daemon-reload`

`$ sudo /bin/systemctl enable elasticsearch.service`

`$ sudo systemctl start elasticsearch.service`

`$ wget https://files.molo.ch/builds/ubuntu-16.04/moloch_1.5.3-1_amd64.deb`

`$ dpkg -i moloch_1.5.3-1_amd64.deb`

### Moloch Configuration ###
`$ sudo /data/moloch/bin/Configure`

`$ sudo /data/moloch/db/db.pl http://localhost:9200 init`

`$ /data/moloch/bin/moloch_add_user.sh admin "Admin user" [password] --admin`

`$ systemctl start molochcapture.service`

`$ systemctl start molochviewer.service`

### Enabling TLS for Viewer ###
(1) Creating keypair and Certificate
`$ sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /data/moloch/etc/moloch.key -out /data/moloch/etc/moloch.cert`

(2) Comment out the below line in the configuration file(/data/moloch/etc/config.ini) to enable TLS

certFile=/data/moloch/etc/moloch.cert

keyFile=/data/moloch/etc/moloch.key

### Moloch packet filtering ###
(1) Edit bpf under /data/moloch/etc/config.ini by using bpf filtering syntax

### Moloch data deletion ###
(1) Moloch pcap files deletion:

/data/moloch/etc/config.ini: freeSpaceG = 5%

(2) Elasticsearch indices deletion

Build-in shell script: /data/moloch/db/daily.sh

Scheduling using /etc/crontab

### Moloch analysis ###

## dsniff ##
`$ sudo apt-get update`

`$ sudo apt-get install dsniff`

`$ nano ./dsniffloop.sh`

For loop through all captured .pcap files by using dsniff -p:

for file in `find /data/moloch/raw/`;

do

  dsniff -p file

done

## Problems faced ##
1. Old version elasticsearch: If you choose to install elasticseach during Moloch installation, the elasticsearch version accompanied is too old, and it binds elasticsearch service under /data/moloch/elasticsearch

Solution: (1) $ sudo systemcl status elasticserach service (2) rm -rf /data/moloch/elasticsearch

2. ReadOnly protection by moloch: Moloch captures tons of packets, and there is a freespaceG setting in /data/moloch/etc/config.ini to delete pcap files when free space is lower then this. Also, elasticsearch will switch the state into read only when in low storage.

Solution: Use the script below to switch read_only_allow_delete to false

curl -X PUT "localhost:9200/_settings" -H 'Content-Type:application/json' -d '

{

    "index": {
    
    "blocks": {
    
    "read_only_allow_delete": "false"
    
    }
    
    }

}'

## Reference websites ##
1. [5 Linux / Unix Commands For Connecting To The Serial Console](https://www.cyberciti.biz/hardware/5-linux-unix-commands-for-connecting-to-the-serial-console/)

2. [Connect your Ubuntu Linux machine to cisco serial console](https://linuxconfig.org/connect-your-ubuntu-linux-machine-to-cisco-serial-console)

3. [Catalyst 3750-E and 3560-E Switch Software Configuration Guide, 12.2(37)SE](https://www.cisco.com/c/en/us/td/docs/switches/lan/catalyst3750e_3560e/software/release/12-2_37_se/configuration/guide/3750escg/swspan.html)

4. [Install Elasticsearch with Debian Package](https://www.elastic.co/guide/en/elasticsearch/reference/current/deb.html#deb)

5. [Moloch GitHub page](https://github.com/aol/moloch)

6. [Clean up indices in elasticsearch](https://discuss.elastic.co/t/how-to-clean-up-storage-space-in-elasticsearch-cluster/43044/4)

7. [ReadOnly error solution](https://discuss.elastic.co/t/forbidden-12-index-read-only-allow-delete-api/110282/5)

8. [Moloch configuration file](https://github.com/aol/moloch/wiki/Settings)

9. [Data never gets deleted](https://github.com/aol/moloch/wiki/FAQ#data-never-gets-deleted)
