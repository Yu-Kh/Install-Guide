Getting Started Guide
=====================

1. Software Prerequisites
=========================

This guide is based on Debian 9 (Stretch) 64-bit `official
distributive`_ with installed GNOME GUI.

1.1 Install sudo
----------------

All commands in this guide should be run as non root user. But some
system commands need superuser privileges to be executed. By default,
sudo is not installed on Debian 9, and first, you should install it.

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

Some of used packages can be downloaded from the official Debian
repository. Install packages:

::

   $ sudo apt install -y postgresql git curl apt-transport-https build-essential

1.3 Create Apla directory
-------------------------

All software used by Apla Blockchain Platform is recommended to store in
a special directory.

Make directory and go to it:

::

   $ sudo mkdir /opt/apla && cd /opt/apla

Make your user owner of this directory:

::

   $ sudo chown user /opt/apla/

1.4 Install Node.js
-------------------

Download Node.js LTS version 8.11 from the `official site`_ or via
command line:

::

   $ curl -sL https://deb.nodesource.com/setup_8.x | sudo -E bash

Install Node.js:

::

   $ sudo apt install -y nodejs

1.5 Install Go Language
-----------------------

Download Go latest stable version 1.10 from the `official
site <https://golang.org/dl/>`__ or via command line:

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

Change user’s password postgres to Apla’s default (you can set your own
password, but also you must change it in node configuration file
‘config.toml’):

::

   $ sudo -u postgres psql -c "ALTER USER postgres WITH PASSWORD 'genesis'"

Create node current state database, for example ‘genesis1’:

::

   $ sudo -u postgres psql -c "CREATE DATABASE genesis1"

1.7 OS Firewall Requirements
----------------------------

By default, after installing Debian 9, there are no firewall rules. But,
if you want to design more secure system with firewall, next incoming
connections should be allowed: - 7078/TCP - Node’s TCP-server - 7079/TCP
- Node’s API-server - 8000/TCP - Centrifugo server

2. Backend Install
==================

Apla Blockchain Platform’s backend consists of two main components: •
Centrifugo (notification server) • Go-Genesis (kernel of the Apla’s
node, contains TCP-server and API-server)

In this guide, all of these components are deployed on one host, but in
production environment, you can deploy them on different hosts.

2.1 First Node Deployment
-------------------------

2.1.1 Install Centrifugo
~~~~~~~~~~~~~~~~~~~~~~~~

Download Centrifugo version 1.7.9 from `GitHub`_ or via command line:

::

   $ wget https://github.com/centrifugal/centrifugo/releases/download/v1.7.9/centrifugo-1.7.9-linux-amd64.zip && unzip centrifugo-1.7.9-linux-amd64.zip && mkdir centrifugo && mv centrifugo-1.7.9-linux-amd64/* centrifugo/

Remove temporary files:

::

   $ rm -R centrifugo-1.7.9-linux-amd64 && rm centrifugo-1.7.9-linux-amd64.zip

Create Centrifugo configuration file (you can set your own “secret”, but
also you must change it in node configuration file ‘config.toml’):

::

   $ echo '{"secret":"CENT_SECRET"}' > centrifugo/config.json

2.1.2 Install Go-Genesis
~~~~~~~~~~~~~~~~~~~~~~~~

Create go-genesis and node1 directories:

::

   $ mkdir go-genesis && cd go-genesis && mkdir node1

Download and buid latest release of Go-Genesis from
`GitHub <https://github.com/GenesisKernel/go-genesis/releases>`__ and
copy it into go-genesis directory:

::

   $ go get -v github.com/GenesisKernel/go-genesis && cd /opt/apla && mv bin/go-genesis go-genesis/ && rm -rf bin/ && rm -rf src/

Usage and flags of go-genesis are described in `documentation`_.

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

For starting first node you should start two services: - centrifugo -
go-genesis

If you did not create these services, you can just execute binary files
from its directories in different consoles.

First, execute centrifugo file:

::

   $ cd /opt/apla/centrifugo && ./centrifugo -a Node_IP-address --config=config.json

Then, in another console execute go-genesis file:

::

   $ cd /opt/apla/go-genesis/ && ./go-genesis start --config=node1/config.toml

Now, you can connecting to your node via Molis App.

2.2 Other Nodes Deployment
--------------------------

Deployment of the second node and others is similar to the first node,
but has some differences in creation of go-genesis ‘config.toml’ file.

2.2.1 Configuration
~~~~~~~~~~~~~~~~~~~

First, you need copy file of the first block to Node 2. For example you
can do it via scp:

::

   $ scp user@10.10.99.1:/opt/apla/go-genesis/node1/firstblock /opt/apla/go-genesis/node2/

Create Node2 configuration file: 

::

$ ./go-genesis config –dataDir=/opt/apla/go-genesis/node2 –firstBlock=node2/firstblock –dbName=genesis2 –privateBlockchain=true –centSecret=“CENT_SECRET” –centUrl=http://localhost:8000 –httpHost=10.10.99.2


.. _GitHub: https://github.com/centrifugal/centrifugo/releases/
.. _documentation: http://genesiskernel.readthedocs.io/en/latest/
.. _official distributive: https://www.debian.org/CD/http-ftp/#stable
.. _official site: https://nodejs.org/en/download/
