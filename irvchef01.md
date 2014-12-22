# irvchef01 setup
This document describes how to build a VMWare box configured with a minimal CentOS 6.x installation for **`irvchef01.usa.hardie.win`**. This will run the Chef server software.


## Requirements
- [CentOS 6.x](http://isoredirect.centos.org/centos/6/isos/x86_64/) download the `CentOS-6.x-x86_64-minimal.iso` file (~400 MB)
- [Chef Server](https://web-dl.packagecloud.io/chef/stable/packages/el/6/chef-server-core-12.0.0-1.el6.x86_64.rpm) rpm package


## Configure VMWare
Configuration:

- [x] Custom

Name and Location:

- `irvchef01.usa.hardie.win`

Storage:

- datastore1

Virtual Machine Version:

- Virtual Machine Version: **8**

Guest Operating System:

- Linux
- Red Hat Enterprise Linux 6 (64-bit)

CPUs:

- Number of virtual sockets: **1**
- Number of cores per virtual socket: **1**

Memory:

- Memory Size: **1 GB**

Network:

- How many NICs: **1**
- NIC 1: **VM Network**
- Adapeter: **E1000**
- Connect at Power On: **[x]**

SCSI Controller:

- [x] VMWare Paravirtual

Select a Disk:

- [x] Create a new virtual disk

Create a Disk:

- Virtual disk size: **100.00 GB**
- Thick Provision Lazy Zeroed
- Store with the virtual machine

Advanced Options:

- Virtual Device Node: SCSI (0:0)


Network:

- Name: **`irvchef01.usa.hardie.win`**
- Manual IP assignment: **`172.16.0.90`**
- DNS: **`172.16.0.10`**
- Gateway: **`172.16.0.1`**


## Install CentOS 6.x
Connect CD/DVD Drive to CentOS 6.x minimal ISO.

Start VM (CentOS 6.4 used in this example):

- Welcome to CentOS 6.4!: **[ Install or upgrade an existing system ]**
- Disc Found: **[ Skip ]**
- CentOS: **[ OK ]**
- Language Selection: **English [ OK ]**
- Keyboard Selection: **us [ OK ]**
- Warning (Error processing drive): **[ Re-initialize all ]**
- Time Zone Selection: **[ OK ]** *(after adjusting settings)*

        [*] System clock uses UTC

        America/Chicago

- Root Password: **[ OK ]** *(after entering password)*

        Password: vagrant
        Password (confirm): vagrant

- Weak Password: **[ Use Anyway ]**
- Partitioning Type: **[ OK ]** *(after adjusting settings)*

        Use entire drive

        [*] sda

- Writing storage configuration to disk: **[ Write changes to disk ]**
- *(minimal package installation occurs)*

- Complete: **[ Reboot ]**


## Create/configure vagrant user
Login as root.

Enable networking:

    # ifup eth0

Create `vagrant` user:

  	# useradd vagrant
  	# passwd vagrant (vagrant)

Configure sudo via `visudo`:

  	# visudo

- Enable group `wheel`:

        %wheel  ALL=(ALL)       ALL

- Enable user `vagrant`:

        vagrant ALL=(ALL)       NOPASSWD: ALL

- Write file and exit

        <ESC>
        :w
        :q

Test login via ssh from separate terminal.

    $ ssh vagrant@<ip.ad.dr.es>

Disable SELinux:

    $ sudo setenforce permissive

Update packages:

    $ sudo yum -y update

Install useful packages:

    $ sudo yum -y install nano wget

Enable networking on boot:

    $ sudo nano /etc/sysconfig/network-scripts/ifcfg-eth0
    ONBOOT=yes


## Update BlueCoat configuration

Enable outbound http(s) traffic:

- BlueCoat > Policy > Visual Policy Manager > Web Authentication Layer
- Update the `ManufacturingDataCollection` object

Enable direct access to `packagecloud.io`:

- BlueCoat > Policy > Visual Policy Manager > ssl_intercept_layer
- Add `packagecloud.io`
    

## Install Chef server

Refer to [Chef Server Installation](http://docs.chef.io/install_server.html) instructions.

Note:

- The test step must succeed before proceeding.
- `Errno::ECONNREFUSED`: verify the DNS entry (or `/etc/hosts`).

Install Chef Server:

    $ cd /tmp
    $ wget https://web-dl.packagecloud.io/chef/stable/packages/el/6/chef-server-core-12.0.0-1.el6.x86_64.rpm
    $ sudo yum install chef-server-core-12.0.0-1.el6.x86_64.rpm
    $ sudo chef-server-ctl reconfigure
    $ sudo chef-server-ctl test
    $ sudo chef-server-ctl user-create user_name first_name last_name email password --filename FILE_NAME

Install Chef Manage package:

    $ sudo chef-server-ctl install opscode-manage
    $ sudo opscode-manage-ctl reconfigure
    $ sudo chef-server-ctl reconfigure

Install Chef Push Jobs package:

    $ sudo chef-server-ctl install opscode-push-jobs-server
    $ sudo opscode-push-jobs-server-ctl reconfigure
    $ sudo chef-server-ctl reconfigure

Install Chef Reporting package:

    $ sudo chef-server-ctl install opscode-reporting
    $ sudo opscode-reporting-ctl reconfigure
    $ sudo chef-server-ctl reconfigure

Update firewall:

Add the following lines to `/etc/sysconfig/iptables` below line containing `--dport 22`:

    -A INPUT -m state --state NEW -m tcp -p tcp --dport 80 -j ACCEPT
    -A INPUT -m state --state NEW -m tcp -p tcp --dport 443 -j ACCEPT

Reload firewall:

    $ sudo service iptables reload


## Configure Chef Server

Login to [irvchef01](https://irvchef01.usa.hardie.win).

- Create admin user
- Create organization
- Create additional user(s)
