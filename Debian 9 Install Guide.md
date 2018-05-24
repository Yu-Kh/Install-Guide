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
- Go-genesis (kernel of the Apla's node, contains TCP-server and API-server)
- Molis (frontend client)
In this guide all of this components are deployed on one node, but in production environment you can deploy them on different hosts.
## Deploy First Node
### Install Centrifugo
### Install Go-Genesis
### Build Molis App
### Create Services
### Start First Node
## Deploy Second Node
### Configuration
### Add keys
### Create connection between nodes
## Work with system
### Login as Founder
