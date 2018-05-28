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

.. _official distributive: https://www.debian.org/CD/http-ftp/#stable
.. _official site: https://nodejs.org/en/download/
