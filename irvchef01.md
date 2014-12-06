# irvchef01 setup
This document describes how to build a VMWare box configured with a minimal CentOS 6.x installation for **`irvchef01.usa.hardie.win`**. This will run the Chef server software.


## Requirements
- [CentOS 6.x](http://isoredirect.centos.org/centos/6/isos/x86_64/) download the `CentOS-6.x-x86_64-minimal.iso` file (~400 MB)
- [Chef Server](https://web-dl.packagecloud.io/chef/stable/packages/el/6/chef-server-core-12.0.0-1.el6.x86_64.rpm)


## Configure VMWare
General:

- Name: *(varies)*
- Type: **Linux**
- Version: **Red Hat (64 bit)**

System:

- Motherboard
	- Base Memory: **1024 MB**
	- Boot Order: **CD/DVD-ROM, Hard Disk**
	- Extended Features: **Enable IO APIC, Hardware clock in UTC time**

Display:

- *(use default settings)*

Storage:

- Controller: SATA **`centos-6.4-minimal.vdi`**
- Virtual Size: **100.00 GB**
- Details: **Dynamically allocated**

Audio:

- **Disable audio**

Network:

- Name: **`irvchef01.usa.hardie.win`**
- Manual IP assignment: **`172.16.0.90`**
- DNS: **`172.16.0.10`**
- Gateway: **`172.16.0.1`**

Ports:

- *(use default settings)*

Shared Folders:

- *(use default settings)*


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

Update BlueCoat configuration:

- Enable outbound http(s) traffic
    BlueCoat > Policy > Visual Policy Manager > Web Authentication Layer
    Update the `ManufacturingDataCollection` object
- Enable direct access to `packagecloud.io`:
    BlueCoat > Policy > Visual Policy Manager > ssl_intercept_layer
    Add `packagecloud.io`
    
Install `wget`:

    $ sudo yum -y install wget

Install Chef server:

Refer to [Chef Server Installation](http://docs.chef.io/install_server.html)

    $ cd /tmp
    $ wget https://web-dl.packagecloud.io/chef/stable/packages/el/6/chef-server-core-12.0.0-1.el6.x86_64.rpm
    $ sudo yum install chef-server-core-12.0.0-1.el6.x86_64.rpm
    $ sudo chef-server-ctl reconfigure
    $ sudo chef-server-ctl user-create user_name first_name last_name email password --filename FILE_NAME

Install Chef Manage package:

    $ sudo chef-server-ctl install opscode-manage
    $ sudo opscode-manage-ctl reconfigure
    $ sudo chef-server-ctl reconfigure
