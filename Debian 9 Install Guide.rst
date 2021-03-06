Getting Started Guide
=====================

1. Software Prerequisites
=========================

This guide is based on Debian 9 (Stretch) 64-bit `official distributive <https://www.debian.org/CD/http-ftp/#stable>`_ with installed GNOME GUI.

1.1 Install sudo
----------------

All commands in this guide should be run as non root user. But some system commands need superuser privileges to be executed. By default, sudo is not installed on Debian 9, and first, you should install it.

Become root superuser:

::

   $ su - 

Upgrade your system:

::

   # apt update -y && apt upgrade -y && apt dist-upgrade -y

Install sudo:

::

   # apt install sudo -y

Add your user to sudo group:

::

   # usermod -aG sudo user

After the reboot, the changes take effect.

1.2 Install common software
---------------------------

Some of used packages can be downloaded from the official Debian repository. Install packages:

::

   $ sudo apt install -y postgresql git curl apt-transport-https build-essential

1.3 Create Apla directory
-------------------------

All software used by Apla Blockchain Platform is recommended to store in a special directory.

Make directory and go to it:

::

   $ sudo mkdir /opt/apla && cd /opt/apla

Make your user owner of this directory:

::

   $ sudo chown user /opt/apla/

1.4 Install Node.js
-------------------

Download Node.js LTS version 8.11 from the `official site <https://nodejs.org/en/download/>`_ or via command line:

::

   $ curl -sL https://deb.nodesource.com/setup_8.x | sudo -E bash

Install Node.js:

::

   $ sudo apt install -y nodejs

1.5 Install Go Language
-----------------------

Download Go latest stable version 1.10 from the `official site <https://golang.org/dl/>`_ or via command line:

::

   $ wget https://dl.google.com/go/go1.10.1.linux-amd64.tar.gz

Install Go:

::

   $ tar -xvf go1.10.1.linux-amd64.tar.gz && sudo mv go /usr/local/

Export Go environment variables:

::

   $ export GOROOT=/usr/local/go && export GOPATH=/opt/apla/ && export PATH=$GOPATH/bin:$GOROOT/bin:$PATH

Remove temporary file:

::

   $ rm go1.10.1.linux-amd64.tar.gz

1.6 Setup PostgreSQL
--------------------

Change user’s password postgres to Apla’s default (you can set your own password, but also you must change it in node configuration file ‘config.toml’):

::

   $ sudo -u postgres psql -c "ALTER USER postgres WITH PASSWORD 'genesis'"

Create node current state database, for example ‘genesis1’:

::

   $ sudo -u postgres psql -c "CREATE DATABASE genesis1"

1.7 OS Firewall Requirements
----------------------------

By default, after installing Debian 9, there are no firewall rules. But, if you want to design more secure system with firewall, next incoming connections should be allowed:

- 7078/TCP - Node’s TCP-server 
- 7079/TCP - Node’s API-server 
- 8000/TCP - Centrifugo server

2. Backend Install
==================

Apla Blockchain Platform’s backend consists of two main components:

- Centrifugo (notification server)
- Go-Genesis (kernel of the Apla’s node, contains TCP-server and API-server)

In this guide, all of these components are deployed on one host, but in production environment, you can deploy them on different hosts.

2.1 First Node Deployment
-------------------------

2.1.1 Install Centrifugo
~~~~~~~~~~~~~~~~~~~~~~~~

Download Centrifugo version 1.7.9 from `GitHub <https://github.com/centrifugal/centrifugo/releases/>`_ or via command line:

::

   $ wget https://github.com/centrifugal/centrifugo/releases/download/v1.7.9/centrifugo-1.7.9-linux-amd64.zip && unzip centrifugo-1.7.9-linux-amd64.zip && mkdir centrifugo && mv centrifugo-1.7.9-linux-amd64/* centrifugo/

Remove temporary files:

::

   $ rm -R centrifugo-1.7.9-linux-amd64 && rm centrifugo-1.7.9-linux-amd64.zip

Create Centrifugo configuration file (you can set your own “secret”, but also you must change it in node configuration file ‘config.toml’):

::

   $ echo '{"secret":"CENT_SECRET"}' > centrifugo/config.json

2.1.2 Install Go-Genesis
~~~~~~~~~~~~~~~~~~~~~~~~

Create go-genesis and node1 directories:

::

   $ mkdir go-genesis && cd go-genesis && mkdir node1

Download and buid latest release of Go-Genesis from `GitHub <https://github.com/GenesisKernel/go-genesis/releases/>`_ and copy it into go-genesis directory:

::

   $ go get -v github.com/GenesisKernel/go-genesis && cd /opt/apla && mv bin/go-genesis go-genesis/ && rm -rf bin/ && rm -rf src/

Usage and flags of go-genesis are described in `documentation <http://genesiskernel.readthedocs.io/en/latest/>`_.

Create Node1 configuration file:

::

   $ ./go-genesis config --dataDir=/opt/apla/go-genesis/node1 --firstBlock=node1/firstblock --dbName=genesis1 --privateBlockchain=true --centSecret="CENT_SECRET" --centUrl=http://localhost:8000 --httpHost=10.10.99.1 --tcpHost=10.10.99.1

Generate Node1 keys:

::

   $ ./go-genesis generateKeys --config=node1/config.toml

Generate first block:

::

   $ ./go-genesis generateFirstBlock --config=node1/config.toml

Initialize database:

::

   $ ./go-genesis initDatabase --config=node1/config.toml

2.1.3 Create Services
~~~~~~~~~~~~~~~~~~~~~

Under development

2.1.4 Start First Node
~~~~~~~~~~~~~~~~~~~~~~

For starting first node you should start two services:

- centrifugo
- go-genesis

If you did not create these services, you can just execute binary files from its directories in different consoles.

First, execute centrifugo file:

::

   $ cd /opt/apla/centrifugo && ./centrifugo -a Node_IP-address --config=config.json

Then, in another console execute go-genesis file:

::

   $ cd /opt/apla/go-genesis/ && ./go-genesis start --config=node1/config.toml

Now, you can connecting to your node via Molis App.

2.2 Other Nodes Deployment
--------------------------

Deployment of the second node and others is similar to the first node, but has some differences in creation of go-genesis ‘config.toml’ file.

2.2.1 Configuration
~~~~~~~~~~~~~~~~~~~

First, you need copy file of the first block to Node 2. For example you can do it via scp:

::

   $ scp user@10.10.99.1:/opt/apla/go-genesis/node1/firstblock /opt/apla/go-genesis/node2/

Create Node2 configuration file: 

::

$ ./go-genesis config –dataDir=/opt/apla/go-genesis/node2 –firstBlock=node2/firstblock –dbName=genesis2 –privateBlockchain=true –centSecret=“CENT_SECRET” –centUrl=http://localhost:8000 –httpHost=10.10.99.2

2.2.2 Adding keys
~~~~~~~~~~~~~~~~~

Errors that occurred above are caused by untrusted relationships between nodes. To fix it, you should add the second node public key to the first node.

To adding keys you should download this script updateKeys.py. All information that you are need to script execution are located in node’s directory ‘nodeN’. This scipt must be executed on the first node with founder’s privileges. Execute script with next arguments:

::

   $ python updateKeys.py PrivateKey1 Host1 Port1 KeyID2 PublicKey2 balance

Where:

- PrivateKey1 - founder private key, located in the file 'PrivateKey' of the first node 
- Host1 - IP-addres or DNS-name of the first node 
- Port1 - the first node API-server port
- KeyID2 - content of file KeyID of the second node
- PublicKey2 - content of file 'PublicKey' of the second node
- balance - set wallet balance of the second node

**Example:**

::

   $ python updatekeys.py bda1c45d3298cb7bece1f76a81d8016d33cdec18c925297c7748621c502a23f2 10.10.99.1 7079 -5910245696104921893 1812246837170b6df8609fd9d846a0984f4e5b3ee9037717e39dc38c82ea1a8e528c9e6f6acdc06b2a33f228c4d2649005bde47af857f3f756aaf64d3f1648dd 1000000000000000000000

This script will create contract which add the second node public key to the table ‘keys’ of database.

2.2.3 Create connection between nodes
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Next, you should create connection between nodes. For this, you should download this script newValToFullNodes.py. All information that you are need to script execution are located in node’s directory ‘nodeN’. This scipt must be executed on the first node with founder’s privileges. Execute script with next arguments:

::

   $ python newValToFullNodes.py PrivateKey1 Host1 Port1 'NewValue'

Where:

- PrivateKey1 - founder private key, located in the file PrivateKey of the first node
- Host1 - IP-addres or DNS-name of the first node
- Port1 - the first node API-server port
- NewValue - new value of Full_Nodes parameter

Argument **NewValue** must be written in json format:

::

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

::

   $ python newValToFullNodes.py bda1c45d3298cb7bece1f76a81d8016d33cdec18c925297c7748621c502a23f2 10.10.99.1 7079 '[{"tcp_address":"10.10.99.1:7078","api_address":"http://10.10.99.1:7079","key_id":"5541394763743537703","public_key":"d26824d0e94894bae9e983e7a386a1c9e4f609990d4b635b6926b52c831d6ec28b95f75acf0c9d10ee96afc0dd02617f08fea225706f0e502d5fe26587023e3b"},{"tcp_address":"10.10.99.2:7078","api_address":"http://10.10.99.2:7079","key_id":"6404048169476933259","public_key":"afd9ed260ec65a2a294794285ad40c5edc219e3be2455a044e2444111b8525815b224fdb369aa17307434d0e6aca8f9c959f823756baeb9ccb105f96f996bf11" }, {"tcp_address":"10.10.99.3:7078","api_address":"http://10.10.99.3:7079","key_id":"-5910245696104921893","public_key":"254c38cd6d9f47ffc42a8d178bb47f9a0cbc46ec6ef4d972c05146bfe87a8da03cb3450b71b2a724fdb2184163ae91023931c9fe5f148f0bdceeeefc5a16fe58"}]'

Now, all nodes are connected to each other.

3. Frontend Install
===================

To work with the system you should build and use frontend Molis client.

3.2 Build Molis App
-------------------

For building Molis application you need install Yarn package manager. Download Yarn version 1.6.0 from `GitHub <https://github.com/yarnpkg/yarn/releases>`_ or via command line:

::

   $ cd /opt/apla &&  wget https://github.com/yarnpkg/yarn/releases/download/v1.6.0/yarn_1.6.0_all.deb

Install Yarn:

::

   $ sudo dpkg -i yarn_1.6.0_all.deb && rm yarn_1.6.0_all.deb

Download latest release of Genesis-Front (Molis) from
`GitHub <https://github.com/GenesisKernel/genesis-front>`_ via git:

::

   $ git clone https://github.com/GenesisKernel/genesis-front.git

Install Genesis-Front dependencies via Yarn:

::

   $ cd genesis-front/ && yarn install

Molis client can be build via three technical implementations: - Desktop
Application - Web Application - Mobile Application

3.2.1 Build Molis Desktop App
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Create settings.json file which contains connections information about
full nodes:

::

   $ cp public/settings.json.dist public/settings.json

Edit settings.json file by any text editor and add required settings in
next format:

::

   http://Node_IP-address:Node_HTTP-Port

**Example** settings.json for three nodes:

::

   {
       "fullNodes": [
           "http://10.10.99.1:7079",
           "http://10.10.99.2:7079",
           "http://10.10.99.3:7079"
       ]
   }

Build desktop app by Yarn:

::

   $ cd /opt/apla/genesis-front && yarn build-desktop

Then desktop app must be packed to the AppImage:

::

   $ yarn release --publish never –l

After that, your application will be ready to use, but its connection settings can not be changed in the future. If these settings will change, you must build a new version of the application.

3.2.2 Build Molis Web App
~~~~~~~~~~~~~~~~~~~~~~~~~

Create settings.json file as its described in Build Molis Desktop App section.

Build web app:

::

   $ cd /opt/apla/genesis-front/ && yarn build

After building, redistributable files will be placed to the ‘/build’
directory. You can serve it with any web-server of your choice.
Settings.json file must be also placed there. It is woth noting that you
shouldn’t buld your application again if your connection settings will
change. Just edit settings.json file and restart web-server.

For development or testing purposes you can simple build Yarn’s
web-server:

::

   $ sudo yarn global add serve && serve -s build

After this, your Molis Web App will be accessed at: ht​tp://localhost:5000

3.3.3 Build Molis Mobile App
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Under development

4. Launching
============

After building Molis App, you can obtain access to the system by
selected user.

4.1 Login as Founder
--------------------

To obtain system administrator rights on your ecosystem, you should
login as ecosystem founder (Node 1 founder). To do this, first, you must
obtain the private key of the founder that was generated during the
installation of the node. This key contains in the ‘PrivateKey’ file,
located in node configuration directory (in this guide it is located in
directory ‘/node1’).

Next, in the Molis client in account options you should choose item
‘Import existing key’.

In next window, Import account, you should copy your founder’s private
key in field of account seed and set new founder password.

Now, in accounts list you can see your founder’s account.

4.2 Create wallet
-----------------

To create wallet, in the Molis client in account options you should
choose item ‘Generate new key’.

Invent new account seed phrase or generate it and set new user password.

Now, in accounts list you can see your user’s account.
