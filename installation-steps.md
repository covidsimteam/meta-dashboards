# COVIDSIM.TEAM DB Installation

The following are the installation steps for Neo4j and CouchDB databases. We use Neo4j for our complex graph data and CouchDB for document data as we need features for audits of data mutation/deletion and highly customizable database level events for the rest of the data analysis pipeline. Once setup on a server and made accessible to us, the databases will be setup and populated with sample data remotely. Our frontends will also provide ways to do batch imports of CSV/GeoJSON files directly to these databases.

We would like to note that 
- We advise installing only Neo4j and CouchDB and providing us remote access to these databases at the moment.
- The [server side application](https://github.com/covidsimteam/socnetgen) maintains its own dockerized PostgreSQL database. This backend and its DB can be installed using `docker-compose -f docker-compose.yml up` at a later point after Neo4j and CouchDB are setup.
- CouchDB and Neo4j only contain anonymized data whereas person identifying data are stored in PostgreSQL and require highest level of authorization for access. 
- The dashboard-related frontends can be hosted anywhere because they do not display any person identifying data thanks to our inherent anonymization by design.

## Installation Steps

1. Install SDKMAN and JDK 11.

```Bash
sudo apt install zip unzip -y
curl -s "https://get.sdkman.io" | bash
source ~/.sdkman/bin/sdkman-init.sh
sdk install java 11.0.7.hs-adpt
```


2. Install Neo4j 4.0:

```Bash
wget -O - https://debian.neo4j.com/neotechnology.gpg.key | sudo apt-key add -
echo 'deb https://debian.neo4j.com stable 4.0' | sudo tee -a /etc/apt/sources.list.d/neo4j.list
sudo apt-get update

sudo apt-get install neo4j=1:4.0.4 -y

sudo neo4j-admin set-initial-password passwordOfYourChoice
```


3. Install CouchDB:

```Bash
echo "deb https://apache.bintray.com/couchdb-deb focal main" | sudo tee -a /etc/apt/sources.list
curl -L https://couchdb.apache.org/repo/bintray-pubkey.asc   | sudo apt-key add -
sudo apt-get update
sudo apt-get install couchdb
```

When prompted please select 'standalone' setup and use 0.0.0.0 address instead of 127.0.0.1. 


4. Enable DB access from non-local IP addresses:

We will be needing the following ports to be accessible remotely:

- CoucbDB: 5984
- Neo4j: 7473, 7687
- Neo4j Graphite monitoring: 2003
- Java/Scala application server: 8081 (after it will be added later)


For Neo4j:

Edit neo4j.conf as superuser e.g.:

```Bash
sudo nano /etc/neo4j/neo4j.conf
```

Uncomment the line with the following key-value around line 54:

```
dbms.default_listen_address=0.0.0.0
```

Restart neo4j service and check that the ports are binding to 0.0.0.0 instead of 127.0.0.1:

```Bash
sudo systemctl restart neo4j.service
netstat -tlpn | grep 7687
netstat -tlpn | grep 7473
```

For couchdb:
```Bash
netstat -tlpn | grep 5984
```

Lastly, please also check the firewall and iptables for any conflicting rules disallowing these ports.

_____________________________________________________________________________________________________


### Only For High Performance Servers

I have relegated the use of couchbase in favor couchdb due to poor performance on low spec servers. However to use couchbase in the future, when we might need its features not available in couchdb and when we have a 16GB+ RAM server, we can use the following steps to setup couchbase:


1. DO NOT use apt/aptitude for couchbase installation. Use the debian package directly with `dpkg -i` or `gdebi` instead with a community edition download from https://www.couchbase.com/downloads
2. a) Pay attention to the warnings during install especially for disabling transparent huge pages and swappiness.
https://docs.couchbase.com/server/6.0/install/thp-disable.html
https://docs.couchbase.com/server/6.0/install/install-swap-space.html

You can start couchbase service using `systemctl start couchbase-server`

2. b) Initialize a node with the cli:
```
couchbase-cli node-init -c 192.168.2.245 \
-u placeholdername -p placeholderpwd \
--node-init-data-path /opt/couchbase/var/lib/couchbase/data \
--node-init-index-path /opt/couchbase/var/lib/couchbase/data \
--node-init-analytics-path /opt/couchbase/var/lib/couchbase/data \
--node-init-hostname node1-devcluster.com \
--ipv4
```

3. Uninstallation:

```
sudo systemctl stop couchbase-server
sudo dpkg -r couchbase-server-community
sudo rm -rf /opt/couchbase
```