# Apla Getting Started Guide
## Table of contents

   * [Overview](#overview)
   * [Backend Install](#backend-install)
      * [Backend Install for Debian](#backend-install-deb)
        * [Backend Software Prerequisites](#backend-software-prerequisites-deb)
        * [First Node Deployment](#first-node-deployment-deb)
        * [Other Nodes Deployment](#other-nodes-deployment-deb)
      * [Backend Install for Windows](#backend-install-win)
        * [Backend Software Prerequisites](#backend-software-prerequisites-win)
        * [First Node Deployment](#first-node-deployment-win)
        * [Other Nodes Deployment](#other-nodes-deployment-win)
   * [Frontend Install](#frontend-install)
      * [Frontend Install for Debian](#frontend-install-deb)
        * [Frontend Software Prerequisites](#frontend-software-prerequisites-deb)
        * [Build Molis App](#build-molis-app-deb)
      * [Frontend Install for Windows](#frontend-install-win)
        * [Frontend Software Prerequisites](#frontend-software-prerequisites-win)
        * [Build Molis App](#build-molis-app-win)
   * [Launching](#launching)

## Overview <a name="overview"></a>

Apla Blockchain is a platform which was developed for building digital ecosystems. The platform includes an integrated application development environment with a multi-level system of access rights to data, interfaces and smart contracts.

Apla Blockchain Platform consists of two main components:
- Backend

  Contains:
  - Centrifugo notification service
  - Go-Apla kernel service (includes Apla TCP and API servers)
  - PostgreSQL database
  
- Frontend
  
  Consist of Molis client that can be builded as native OS application or web-application
  
In production environment, each of these components (backend and frontend) can be deployed on different hosts and OS.

In this guide we will deployed Apla Blockchain Platform based on three nodes on the test ICT-infrastructure and build Molis client. As Apla node OS we will used:
 - Debian 9 (Stretch) 64-bit [official distributive](https://www.debian.org/CD/http-ftp/#stable) with installed GNOME GUI 
 - Windows Server 2012R2/2016

For test purposes, all of these hosts are connected to each other in simple network and have IP-addresses: 10.10.99.1-10.10.99.3



## ***Backend Install*** <a name="backend-install"></a>

In  this section we will deploy Apla Backend components. All of these components are deployed on one node.

## Backend Install for Debian OS <a name="backend-install-deb"></a>

### Backend Software Prerequisites <a name="backend-software-prerequisites-deb"></a>

Before install Apla Backend components, you need install several additional software.

#### Install sudo

All commands for Debian 9 should be run as non root user. But some system commands need superuser privileges to be executed. By default, sudo is not installed on Debian 9, and first, you should install it.

1) Become root superuser:
```
$ su - 
```
2) Upgrade your system:
```
# apt update -y && apt upgrade -y && apt dist-upgrade -y
```
3) Install sudo:
```
# apt install sudo -y
```
4) Add your user to sudo group:
```
# usermod -aG sudo user
```
5) After the reboot, the changes take effect.

#### Install common software

Some of used packages can be downloaded from the official Debian repository. Install packages:
```
$ sudo apt install -y postgresql git curl apt-transport-https build-essential
```

#### Create Apla directory

For Debian 9 OS, all software used by Apla Blockchain Platform is recommended to store in a special directory. In this guide, we will use /opt/apla directory as main, but you can change it to your own.

1) Make directory and go to it:
```
$ sudo mkdir /opt/apla && cd /opt/apla
```
2) Make your user owner of this directory:
```
$ sudo chown user /opt/apla/
```

#### Install Go Language

1) Download Go latest stable version 1.10 from the official site or via command line:
```
$ wget https://dl.google.com/go/go1.10.1.linux-amd64.tar.gz
```
2) Install Go:
```
$ tar -xvf go1.10.1.linux-amd64.tar.gz && sudo mv go /usr/local/
```
3) Export Go environment variables:
```
$ export GOROOT=/usr/local/go && export GOPATH=/opt/apla/ && export PATH=$GOPATH/bin:$GOROOT/bin:$PATH
```
4) Remove temporary file:
```
$ rm go1.10.1.linux-amd64.tar.gz
```
#### Setup PostgreSQL

1) Change user's password postgres to Apla's default (you can set your own password, but also you must change it in node configuration file ‘config.toml’):
```
$ sudo -u postgres psql -c "ALTER USER postgres WITH PASSWORD 'apla'"
```
2) Create node current state database, for example ‘apladb’:
```
$ sudo -u postgres psql -c "CREATE DATABASE apladb"
```
#### Install Python packages
These packages should be installed only on the first node because of executing special scripts.

1) Install Python3-pip:
```
$ sudo apt install python3-pip
```

2) Install genesis_blockchain_tools:
```
$ sudo pip3 install git+https://github.com/blitzstern5/genesis-blockchain-tools
```

#### OS Firewall Requirements

By default, after installing Debian 9, there are no firewall rules. But, if you want to design more secure system with firewall, next incoming connections should be allowed:

-	7078/TCP - Node's TCP-server
-	7079/TCP - Node's API-server
-	8000/TCP - Centrifugo server

### First Node Deployment <a name="first-node-deployment-deb"></a>

#### Install Centrifugo

1) Download Centrifugo version 1.7.9 from [GitHub](https://github.com/centrifugal/centrifugo/releases/) or via command line:
```
$ wget https://github.com/centrifugal/centrifugo/releases/download/v1.7.9/centrifugo-1.7.9-linux-amd64.zip && unzip centrifugo-1.7.9-linux-amd64.zip && mkdir centrifugo && mv centrifugo-1.7.9-linux-amd64/* centrifugo/
```

2) Remove temporary files:
```
$ rm -R centrifugo-1.7.9-linux-amd64 && rm centrifugo-1.7.9-linux-amd64.zip
```

3) Create Centrifugo configuration file (you can set your own "secret", but also you must change it in node configuration file ‘config.toml’):
```
$ echo '{"secret":"CENT_SECRET"}' > centrifugo/config.json
```
#### Install Go-Apla

1) Create go-apla and node1 directories:
```
$ mkdir go-apla && cd go-apla && mkdir node1
```
2) Download and buid latest release of Go-Apla from [GitHub](https://github.com/AplaProject/go-apla/releases) and copy it into go-apla directory:
```
$ go get -v github.com/AplaProject /go-apla && cd /opt/apla && mv bin/go-apla go-apla/ && rm -rf bin/ && rm -rf src/
```
Usage and flags of go-apla are described in [documentation](http://genesiskernel.readthedocs.io/en/latest/)

3) Create Node 1 configuration file:
```
$ ./go-apla config --dataDir=/opt/apla/go-apla/node1 --dbName=apladb --privateBlockchain=true --centSecret="CENT_SECRET" --centUrl=http://10.10.99.1:8000 --httpHost=10.10.99.1 --tcpHost=10.10.99.1
```
4) Generate Node 1 keys:
```
$ ./go-apla generateKeys --config=node1/config.toml
```
5) Generate first block:
```
$ ./go-apla generateFirstBlock --config=node1/config.toml
```
6) Initialize database:
```
$ ./go-apla initDatabase --config=node1/config.toml
```
#### Start First Node

For starting first node you should start two services:

-	centrifugo
-	go-apla

If you did not create these services, you can just execute binary files from its directories in different consoles.

1) Execute centrifugo file:
```
$ cd /opt/apla/centrifugo && ./centrifugo -a Node_IP-address --config=config.json
```
2) Execute go-apla file in another console:
```
$ cd /opt/apla/go-apla/ && ./go-apla start --config=node1/config.toml
```

Now, you can connecting to your node via Molis App.

### Other Nodes Deployment <a name="other-nodes-deployment-deb"></a>

Deployment of the second node and others is similar to the first node, but has some differences in creation of go-apla ‘config.toml’ file.

#### Configuration

1) Copy file of the first block to Node 2. For example, you can do it via scp:
```
$ scp user@10.10.99.1:/opt/apla/go-apla/node1/firstblock /opt/apla/go-apla/node2/
```

2) Create Node 2 configuration file:
```
$ ./go-apla config --dataDir=/opt/apla/go-apla/node2 --dbName=apladb  --privateBlockchain=true --centSecret="CENT_SECRET" --centUrl=http://10.10.99.2:8000 --httpHost=10.10.99.2 --tcpHost=10.10.99.2 --nodesAddr=10.10.99.1
```

3) Generate Node 2 keys:
```
$ ./go-apla generateKeys --config=node2/config.toml
```

4) Initialize database:
```
$ ./go-apla initDatabase --config=node2/config.toml
```

5) Start Node 2:
```
$ ./go-apla start --config=node2/config.toml 
```
You should ignore showed errors. If you start node with log level "INFO", you'll see that node start downloaded blocks.

#### Adding keys

Errors that occurred above are caused by untrusted relationships between nodes. To fix it, you should add the second node public key to the first node.

To adding keys you should download this script [updateKeys.py](https://github.com/GenesisKernel/genesis-tests/blob/master/scripts/updateKeys.py). All information that you are need to script execution are located in node's directory 'nodeN'. This scipt must be executed on the first node with founder's privileges. Execute script with next arguments:
```
$ python3 updateKeys.py PrivateKey1 Host1 Port1 KeyID2 PublicKey2 balance
```

Where:
-	PrivateKey1 - founder private key, located in the file PrivateKey of the first node
-	Host1 - IP-addres or DNS-name of the first node
-	Port1 - the first node API-server port
-	KeyID2 - content of file KeyID of the second node
-	PublicKey2 - content of file PublicKey of the second node
-	balance - set wallet balance of the second node

**Example**:
```
$ python3 updatekeys.py bda1c45d3298cb7bece1f76a81d8016d33cdec18c925297c7748621c502a23f2 10.10.99.1 7079 -5910245696104921893 1812246837170b6df8609fd9d846a0984f4e5b3ee9037717e39dc38c82ea1a8e528c9e6f6acdc06b2a33f228c4d2649005bde47af857f3f756aaf64d3f1648dd 1000000000000000000000
```

This script will create contract, which add the second node public key to the table 'keys' of database.

#### Create connection between nodes

Next, you should create connection between nodes. For this, you should download this script [newValToFullNodes.py](https://github.com/GenesisKernel/genesis-tests/blob/master/scripts/newValToFullNodes.py). All information that you are need to script execution are located in node's directory 'nodeN'. This scipt must be executed on the first node with founder's privileges.

Execute script with next arguments:
```
$ python3 newValToFullNodes.py PrivateKey1 Host1 Port1 'NewValue'
```

Where:
-	PrivateKey1 - founder private key, located in the file PrivateKey of the first node
-	Host1 - IP-addres or DNS-name of the first node
-	Port1 - the first node API-server port
-	NewValue - new value of Full_Nodes parameter

Argument **NewValue** must be written in json format:
```
[
 {
  "tcp_address":"Host1:tcpPort1", 
  "api_address":"http://Host1:httpPort1", 
  "key_id":"KeyID1", 
  "public_key":"NodePubKey1"
 },
 {
  "tcp_address":"Host2:tcpPort2", 
  "api_address":"http://Host2:httpPort2", 
  "key_id":"KeyID2", 
  "public_key":"NodePubKey2"
 },
 {
  "tcp_address":"HostN:tcpPortN", 
  "api_address":"http://HostN:httpPortN", 
  "key_id":"KeyIDN", 
  "public_key":"NodePubKeyN"
 }
]
```

Where:
-	Host1 - IP-addres or DNS-name of the first node
-	tcpPort1 - the first node TCP-server port
-	httpPort1 - the first node API-server port
-	KeyID1 - content of file KeyID of the first node
-	NodePubKey1 - content of file NodePublicKey of the first node
-	Host2 - IP-addres or DNS-name of the second node
-	tcpPort2 - the second node TCP-server port
-	httpPort2 - the second node API-server port
-	KeyID2 - content of file KeyID of the second node
-	NodePubKey2 - content of file NodePublicKey of the second node
-	HostN - IP-addres or DNS-name of node N
-	tcpPortN - node N TCP-server port
-	httpPortN - node N API-server port
- KeyIDN - content of file KeyID of node N
-	NodePubKeyN - content of file NodePublicKey of node N

**Example:**
```
$ python3 newValToFullNodes.py bda1c45d3298cb7bece1f76a81d8016d33cdec18c925297c7748621c502a23f2 10.10.99.1 7079 '[{"tcp_address":"10.10.99.1:7078","api_address":"http://10.10.99.1:7079","key_id":"5541394763743537703","public_key":"d26824d0e94894bae9e983e7a386a1c9e4f609990d4b635b6926b52c831d6ec28b95f75acf0c9d10ee96afc0dd02617f08fea225706f0e502d5fe26587023e3b"},{"tcp_address":"10.10.99.2:7078","api_address":"http://10.10.99.2:7079","key_id":"6404048169476933259","public_key":"afd9ed260ec65a2a294794285ad40c5edc219e3be2455a044e2444111b8525815b224fdb369aa17307434d0e6aca8f9c959f823756baeb9ccb105f96f996bf11" }, {"tcp_address":"10.10.99.3:7078","api_address":"http://10.10.99.3:7079","key_id":"-5910245696104921893","public_key":"254c38cd6d9f47ffc42a8d178bb47f9a0cbc46ec6ef4d972c05146bfe87a8da03cb3450b71b2a724fdb2184163ae91023931c9fe5f148f0bdceeeefc5a16fe58"}]'
```
Now, all nodes are connected to each other.

### ***Backend Install for Windows Server OS*** <a name="backend-install-win"></a>

### Backend Software Prerequisites <a name="backend-software-prerequisites-win"></a>

Before install Apla Backend components, you need install several additional software. To do this, you should have administrators privileges.

#### Install Go Language

1) Download Go latest stable version 1.10 for Windows from the [official site](https://golang.org/dl/).

2) Install Go without any specific settings.

#### Install PostgreSQL

1) Download PostgreSQL 10.4 installer for Windows x86-64 from the [official site](https://www.enterprisedb.com/downloads/postgres-postgresql-downloads).

2) During installation process, you should:

- specify default installation directory
- specify all selected components
- specify default data directory
- set a password for the database superuser (postgres), for example ‘apla’
- specify default port 5432 the server should listen on (you can set your own port, but also you must change it in node configuration file ‘config.toml’)
- select default locale to be used by the new database cluster
- after setup wizard completed, don’t launch stack builder

3) Run pgAdmin4 app and create node current state database, for example ‘apladb’

#### Install Git

1) Download the latest 64-bit Git for Windows from the [official site](https://git-scm.com/download/win).

2) Install Git without any specific settings.

#### Install MinGW

You should install MinGW software only if you want build Apla backend from source code.

1) Download the latest MinGW-W64 from its [site](https://sourceforge.net/projects/mingw-w64/).

2) During installation process, you should specify setup settings:

- Version: from drop list select latest version
- Architecture: from drop list select ‘x86_64’
- Threads: from drop list select ‘win32’

Leave the other settings by default.

3) Add absolute path of directory ‘mingw64/bin’ to the system environment variable PATH by command line, for example “C:\mingw64\bin”:
```
> setx PATH “C:\mingw64\bin”
```

4) For Windows Server 2016 you must restart your system.

#### Install Python 3

1) Python 3 should be installed only on the first node because of executing special scripts.

2) Download latest Python 3 Release from the [official site](https://www.python.org/downloads/).

3) During installation process, select “Add python.exe to Path” in features tree. Leave the other settings by default.

4) For script execution install additional package:
```
> py -m pip install requests
```

Install genesis_blockchain_tools:
```
> py -m pip install git+https://github.com/blitzstern5/genesis-blockchain-tools
```

#### OS Firewall Requirements

In Windows Server firewall settings, you should allow next incoming connections:

-	7078/TCP - Node's TCP-server
-	7079/TCP - Node's API-server
-	8000/TCP - Centrifugo server

### First Node Deployment <a name="first-node-deployment-win"></a>

#### Install Centrifugo

1) Download Centrifugo-1.7.9-windows-amd64.zip from [GitHub](https://github.com/centrifugal/centrifugo/releases/).

2) Unzip archive to your Centrifugo folder inside Apla directory

3) By any text editor, create Centrifugo configuration file ‘config.json’ in centrifugo directory. Add the following line to ‘config.json’ file:
```
{"secret":"CENT_SECRET"}
```

You can set your own "secret", but also you must change it in node configuration file ‘config.toml’.

#### Install Go-Apla

1) In Apla directory, create go-apla directory and node folder inside it.

2) Download Go-Apla from [GitHub](https://github.com/AplaProject/go-apla/releases) or build latest release by command line:
```
> cd C:\Apla\go-apla
> go get –v github.com/AplaProject/go-apla
> go build github.com/AplaProject/go-apla
```

After that, ‘go-apla.exe’ file will appear in ‘go-apla’ directory.

Usage and flags of ‘go-apla.exe’ file are described in [documentation](http://genesiskernel.readthedocs.io/en/latest/).

3) Create Node 1 ‘config.toml’ configuration file:
```
> go-apla.exe config --dataDir=C:\Apla\go-apla\node --dbName=apladb --privateBlockchain=true --centSecret="CENT_SECRET" --centUrl=http://10.10.99.1:8000 --httpHost=10.10.99.1 --tcpHost=10.10.99.1
```
4) Generate Node 1 keys:
```
> go-apla.exe generateKeys --config=node\config.toml
```

5) Generate first block:
```
> go-apla.exe generateFirstBlock --config=node\config.toml
```

6) Initialize database:
```
> go-apla.exe initDatabase --config=node\config.toml
```

#### Start First Node

For starting first node you should start two services:

-	centrifugo
-	go-apla

If you did not create these services, you can just execute .exe files from its directories in different command prompts.

1) Run centrifugo.exe:
```
> centrifugo.exe -a 10.10.99.1 --config=config.json
```

2) Run go-apla.exe:
```
> go-apla.exe start --config=node\config.toml
```

Now, you can connecting to your node via Molis App.

### Other Nodes Deployment <a name="other-nodes-deployment-win"></a>

Deployment of the second node and others is similar to the first node, but has some differences in creation of go-apla ‘config.toml’ file.

#### Configuration

1) Copy file of the first block to Node 2 in the same directory. Default location of the first block file you can see in ‘config.toml’ file.

2) Create Node 2 ‘config.toml’ configuration file:
```
> go-apla.exe config --dataDir=C:\Apla\go-apla\node --dbName=apladb --privateBlockchain=true --centSecret="CENT_SECRET" --centUrl=http://10.10.99.2:8000 --httpHost=10.10.99.2 --tcpHost=10.10.99.2 --nodesAddr=10.10.99.1
```

3) Generate Node 2 keys:
```
> go-apla.exe generateKeys --config=node\config.toml
```

4) Initialize database:
```
> go-apla.exe initDatabase --config=node\config.toml
```

5) Start Node 2 services:
```
> centrifugo.exe -a 10.10.99.2 --config=config.json 
> go-apla.exe start --config=node\config.toml
```

You should ignore showed errors. If you start node with log level "INFO", you'll see that node start downloaded blocks.

#### Adding keys

Errors that occurred above are caused by untrusted relationships between nodes. To fix it, you should add the second node public key to the first node.

To adding keys you should download this script [updateKeys.py](https://github.com/GenesisKernel/genesis-tests/blob/master/scripts/updateKeys.py). All information that you are need to script execution are located in node's directory 'node'. This script must be executed on the first node with founder's privileges. Execute script with next arguments:
```
> py updateKeys.py PrivateKey1 Host1 Port1 KeyID2 PublicKey2 balance
```

Where:
-	PrivateKey1 - founder private key, located in the file PrivateKey of the first node
-	Host1 - IP-addres or DNS-name of the first node
-	Port1 - the first node API-server port
-	KeyID2 - content of file KeyID of the second node
-	PublicKey2 - content of file PublicKey of the second node
-	balance - set wallet balance of the second node

**Example:**
```
>py updateKeys.py 0f1aaf0c76716f189a295a0edbeed05ae760c4cd0009bd337f19aea6a0d37d89 10.10.99.1 7079 839301472950762263 2ce37e8a3fbeadd3862e962267fa29c43c02b6d2fbab9360f7d2e988e1477c333aa06fd6d85a8999779f1314063bf2bd2a298ea1284d0284b1c1ea69870d3ba 1000000000000000000000
```

This script will create contract, which add the second node public key to the table 'keys' of database.

#### Create connection between nodes

Then you need to create a connection between the nodes. For this, you should download this script [newValToFullNodes.py](https://github.com/GenesisKernel/genesis-tests/blob/master/scripts/newValToFullNodes.py). All information that you are need to script execution are located in node's directory. This script must be executed on the first node with founder's privileges. Execute script with next arguments:
```
> py newValToFullNodes.py PrivateKey1 Host1 Port1 “NewValue”
```

Where:
-	PrivateKey1 - founder private key, located in the file PrivateKey of the first node
-	Host1 - IP-addres or DNS-name of the first node
-	Port1 - the first node API-server port
-	NewValue - new value of Full_Nodes parameter

Argument **NewValue** must be written in json format:
```
[
 {
  "tcp_address":"Host1:tcpPort1", 
  "api_address":"http://Host1:httpPort1", 
  "key_id":"KeyID1", 
  "public_key":"NodePubKey1"
 },
 {
  "tcp_address":"Host2:tcpPort2", 
  "api_address":"http://Host2:httpPort2", 
  "key_id":"KeyID2", 
  "public_key":"NodePubKey2"
 },
 {
  "tcp_address":"HostN:tcpPortN", 
  "api_address":"http://HostN:httpPortN", 
  "key_id":"KeyIDN", 
  "public_key":"NodePubKeyN"
 }
]
```

Where:

- Host1 - IP-addres or DNS-name of the first node
- tcpPort1 - the first node TCP-server port
- httpPort1 - the first node API-server port
- KeyID1 - content of file KeyID of the first node
- NodePubKey1 - content of file NodePublicKey of the first node
- Host2 - IP-addres or DNS-name of the second node
- tcpPort2 - the second node TCP-server port
- httpPort2 - the second node API-server port
- KeyID2 - content of file KeyID of the second node
- NodePubKey2 - content of file NodePublicKey of the second node
- HostN - IP-addres or DNS-name of node N
- tcpPortN - node N TCP-server port
- httpPortN - node N API-server port
- KeyIDN - content of file KeyID of node N
- NodePubKeyN - content of file NodePublicKey of node N

**Example:**
```
>py updateFullNode.py 0f1aaf0c76716f189a295a0edbeed05ae760c4cd0009bd337f19aea6a0d37d89 10.10.99.1 7079 "[{\"tcp_address\":\"10.10.99.1:7078\",\"api_address\":\"http://10.10.99.1:7079\",\"key_id\":\"4053339477525839986\",\"public_key\":\"d708bda3734e17822245d6477810ff28c150380abb9ae0271c5a49eca05be92fa7d80d0043e476ef936971288dd5df08b83370488182f524d789b919b398e70b\"},{\"tcp_address\":\"10.10.99.2:7078\",\"api_address\":\"http://10.10.99.2:7079\",\"key_id\":\"1647233376862283221\",\"public_key\":\"ed1bd2a29f607ca5529b353e8d6a9d998ebaca30a129a775b738b1bb0da4ff8afde9c6abdf208fd41fa4541cec471417705c7786ab69c359d36d822e8840815a\"},{\"tcp_address\":\"10.10.99.3:7078\",\"api_address\":\"http://10.10.99.3:7079\",\"key_id\":\"5700268145718545990\",\"public_key\":\"f653dd110e19abc259865397f14b1d866215fc4ea9abcaa246e620e750a3e26a46c5565bd6f2d19b4f18af10ccd8088bc62b7b2687b869bd542e91f2203ec164\"}]"
```

Now, all nodes are connected to each other.

## ***Frontend Install*** <a name="frontend-install"></a>

## Frontend Install for Debian <a name="frontend-install-deb"></a>

### Frontend Software Prerequisites <a name="frontend-software-prerequisites-deb"></a>

### Build Molis App <a name="build-molis-app-deb"></a>

## Frontend Install for Windows <a name="frontend-install-win"></a>

### Frontend Software Prerequisites <a name="frontend-software-prerequisites-win"></a>

### Build Molis App <a name="build-molis-app-win"></a>

## Launching <a name="launching"></a>
