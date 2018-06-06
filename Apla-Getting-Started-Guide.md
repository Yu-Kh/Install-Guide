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
      * [Frontend Install for Debian](#backend-install-debian)
        * [Frontend Software Prerequisites](#frontend-software-prerequisites-deb)
        * [Build Molis App](#build-molis-app-deb)
      * [Frontend Install for Windows](#backend-install-win)
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

### Other Nodes Deployment <a name="other-nodes-deployment-deb"></a>

### ***Backend Install for Windows Server OS*** <a name="backend-install-win"></a>

### Backend Software Prerequisites <a name="backend-software-prerequisites-win"></a>

### First Node Deployment <a name="first-node-deployment-win"></a>

### Other Nodes Deployment <a name="other-nodes-deployment-win"></a>

## Frontend Install <a name="frontend-install"></a>

### Frontend Software Prerequisites <a name="frontend-software-prerequisites"></a>

### Build Molis App <a name="build-molis-app"></a>

## Launching <a name="launching"></a>
