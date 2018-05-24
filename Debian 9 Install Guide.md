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
Download and install Node.js LTS version 8.11:
```
$ curl -sL https://deb.nodesource.com/setup_8.x | sudo -E bash && sudo apt install -y nodejs
```

### Install Go

### Setup PostgreSQL
### Remove temporary files
# Apla Blockchain Platform
## Install First Node
### Install Centrifugo
### Install Go-Genesis
### Build Molis App
### Create Services
### Start First Node
## Install Second Node
### Configuration
### Add keys
### Create connection between nodes
## Work with system
### Login as Founder
