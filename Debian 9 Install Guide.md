# Getting Started
## Software Prerequisites
### Install sudo
All commands in this guide should be run as non root user. But some system commands need superuser privileges to be executed.
By default sudo is not installed on Debian 9, and first, you should install it.

Become root superuser:
```
$ su - 
```
Upgrade your system:
```
# apt update -y && apt upgrade -y && apt dist-upgrade -y
```
Install sudo:
```
# apt install sudo -y
```
Add your user to sudo group:
```
# usermod -aG sudo user
```
After the reboot, the changes take effect.

### Install common software
Some of used packages can be downloaded from the official Debian repository.
Install packages:
```
$ sudo apt install -y postgresql git curl apt-transport-https build-essential
```

### Create Apla directory
All software used by Apla Blockchain Platform is recommended to store in a special directory.

Make directory and go to it:
```
$ sudo mkdir /opt/apla && cd /opt/apla
```
Make your user owner of this directory:
```
$ sudo chown user /opt/apla/
```
### Install Node.js
Download Node.js LTS version 8.11 from the [official site](https://nodejs.org/en/download/) or via command line:
```
$ curl -sL https://deb.nodesource.com/setup_8.x | sudo -E bash
```
Install Node.js:
```
$ sudo apt install -y nodejs
```
### Install Go Language
Download Go latest stable version 1.10 from the [official site](https://golang.org/dl/) or via command line:
```
$ wget https://dl.google.com/go/go1.10.1.linux-amd64.tar.gz
```
Install Go:
```
$ tar -xvf go1.10.1.linux-amd64.tar.gz && sudo mv go /usr/local/
```
Export Go environment variables:
```
$ export GOROOT=/usr/local/go && export GOPATH=/opt/apla/ && export PATH=$GOPATH/bin:$GOROOT/bin:$PATH
```
Remove temporary file:
```
$ rm go1.10.1.linux-amd64.tar.gz
```
### Setup PostgreSQL
Change user's password postgres to Apla's default (you can set your own password, but also you must change it in node configuration file):
```
$ sudo -u postgres psql -c "ALTER USER postgres WITH PASSWORD 'genesis'"
```
Create node current state database (by default, the first node have database named genesis1, the second one genesis2 and etc):
```
$ sudo -u postgres psql -c "CREATE DATABASE genesis1"
```
## Install prerequisites via script
This section under development

# Apla Blockchain Platform
Apla Blockchain Platform consists of three main components:
- Centrifugo (notification server)
- Go-Genesis (kernel of the Apla's node, contains TCP-server and API-server)
- Molis (frontend client)

In this guide all of this components are deployed on one host, but in production environment you can deploy them on different hosts.
## First Node Deployment
### Install Centrifugo
Download Centrifugo version 1.7.9 from [GitHub](https://github.com/centrifugal/centrifugo/releases/) or via command line:
```
$ wget https://github.com/centrifugal/centrifugo/releases/download/v1.7.9/centrifugo-1.7.9-linux-amd64.zip && unzip centrifugo-1.7.9-linux-amd64.zip && mkdir centrifugo && mv centrifugo-1.7.9-linux-amd64/* centrifugo/
```
Remove temporary files:
```
$ rm -R centrifugo-1.7.9-linux-amd64 && rm centrifugo-1.7.9-linux-amd64.zip
```
Create Centrifugo configuration file (you can set your own "secret", but also you must change it in node configuration file):
```
$ echo '{"secret":"CENT_SECRET"}' > centrifugo/config.json
```
### Install Go-Genesis
Create go-genesis and node1 directories:
```
$ mkdir go-genesis && cd go-genesis && mkdir node1
```
Download latest release of Go-Genesis from [GitHub](https://github.com/GenesisKernel/go-genesis/releases) and copy it into go-genesis directory:
```
under development
```
Usage and flags of go-genesis are described in [documentation]().

Create Node1 configuration file:
```
$ ./go-genesis config --dataDir=/opt/apla/go-genesis/node1 --firstBlock=node1/firstblock --dbName=genesis1 --privateBlockchain=true --centSecret="CENT_SECRET" --centUrl=http://localhost:8000 --httpHost=10.10.99.1 --tcpHost=10.10.99.1
```
Generate Node1 keys:
```
$ ./go-genesis generateKeys --config=node1/config.toml
```
Generate first block:
```
$ ./go-genesis generateFirstBlock --config=node1/config.toml
```
Initialize database:
```
$ ./go-genesis initDatabase --config=node1/config.toml
```
### Build Molis App
For building Molis application you need install Yarn package manager. Download Yarn version 1.6.0 from [GitHub](https://github.com/yarnpkg/yarn/releases) or via command line:
```
$ cd /opt/apla &&  wget https://github.com/yarnpkg/yarn/releases/download/v1.6.0/yarn_1.6.0_all.deb
```
Install Yarn:
```
$ sudo dpkg -i yarn_1.6.0_all.deb && rm yarn_1.6.0_all.deb
```
Download latest release of Genesis-Front (Molis) from [GitHub](https://github.com/GenesisKernel/genesis-front) via git:
```
$ git clone https://github.com/GenesisKernel/genesis-front.git
```
Install Genesis-Front dependencies via Yarn:
```
$ cd genesis-front/ && yarn install
```
Molis client can be build via three technical implementations:
- Desktop Application
- Web Application
- Mobile Application

#### Build Molis Desktop App
Create settings.json file which contains connections information about full nodes:
```
$ cp public/settings.json.dist public/settings.json
```
Edit settings.json file by any text editor and add required settings in next format:
```
http://Node_IP-address:Node_HTTP-Port
```

**Example** settings.json for three nodes:
```
{
    "fullNodes": [
        "http://10.10.99.1:7079",
        "http://10.10.99.2:7079",
        "http://10.10.99.3:7079"
    ]
}
```
Build desktop app by Yarn:
```
$ cd /opt/apla/genesis-front && yarn build-desktop
```
Then desktop app must be packed to the AppImage:
```
$ yarn release --publish never –l
```
After that, your application will be ready to use, but its connection settings can not be changed in the future. If these settings will change, you must build a new version of the application.
#### Build Molis Web App
Create settings.json file as its described in Build Molis Desktop App section.

Build web app:
```
$ cd /opt/apla/genesis-front/ && yarn build
```
After building, redistributable files will be placed to the '/build' directory. You can serve it with any web-server of your choice. Settings.json file must be also placed there. It is woth noting that you shouldn't buld your application again if your connection settings will change. Just edit settings.json file and restart web-server.

For development or testing purposes you can simple build Yarn's web-server:
```
$ sudo yarn global add serve && serve -s build
```
After this, your Molis Web App will be accessed at: ht&#8203;tp://localhost:5000

#### Build Molis Mobile App
Under development

### Create Services
Under development

### Start First Node
For starting first node you should start two services:
- centrifugo
- go-genesis

If you did not create these services, you can just execute binary files from its directories in different consoles.

First, execute centrifugo file:
```
$ cd /opt/apla/centrifugo && ./centrifugo -a localhost --config=config.json
```
Then, in another console execute go-genesis file:
```
$ cd /opt/apla/go-genesis/ && ./go-genesis start --config=node1/config.toml
```
Now, you can connecting to your node via Molis App.

## Other Nodes Deployment
Deployment of the second node and others is similar to the first node, but has some differences in creation of go-genesis config.toml file.

### Configuration
First, you need copy file of the first block to node 2. For example you can do it via scp:
```
$ scp user@10.10.99.1:/opt/apla/go-genesis/node1/firstblock /opt/apla/go-genesis/node2/
```
Create Node2 configuration file:
```
$ ./go-genesis config --dataDir=/opt/apla/go-genesis/node2 --firstBlock=node2/firstblock --dbName=genesis2 --privateBlockchain=true --centSecret="CENT_SECRET" --centUrl=http://localhost:8000 --httpHost=10.10.99.2 --tcpHost=10.10.99.2 --nodesAddr=10.10.99.1
```
Generate Node2 keys:
```
$ ./go-genesis generateKeys --config=node2/config.toml
```

Initialize database:
```
$ ./go-genesis initDatabase --config=node2/config.toml
```
Now you can start Node2:
```
$ ./go-genesis start --config=node2/config.toml 
```
You should Ignore showed errors. If you start node with log level "INFO", you'll see that node start downloaded blocks.

### Adding keys
Errors that occurred above are caused by untrusted relationships between nodes. To fix it, you should add the second node public key to the first node.

To adding keys you should download this script [updateKeys.py](https://github.com/GenesisKernel/genesis-tests/blob/master/scripts/updateKeys.py). All keys that you are need are located in node's directory 'nodeN'. This scipt must be executed on the first node with founder's privileges. Execute script with next arguments:
```
$ python updateKeys.py PrivateKey1 Host1 Port1 KeyID2 PublicKey2 balance
```
Where:
- PrivateKey1 - founder private key, located in the file PrivateKey of the first node
- Host1 - IP-addres or DNS-name of the first node
- Port1 - the first node API-server port
- KeyID2 - content of file KeyID of the second node
- PublicKey2 - content of file PublicKey of the second node
- balance - set wallet balance of the second node

**Example:**
```
$ python updatekeys.py bda1c45d3298cb7bece1f76a81d8016d33cdec18c925297c7748621c502a23f2 10.10.99.1 7079 -5910245696104921893 1812246837170b6df8609fd9d846a0984f4e5b3ee9037717e39dc38c82ea1a8e528c9e6f6acdc06b2a33f228c4d2649005bde47af857f3f756aaf64d3f1648dd 1000000000000000000000
```
This script create contract which add the second node public key to the table 'keys' of database.

### Create connection between nodes

## Work with system

### Login as Founder
### Create wallet
