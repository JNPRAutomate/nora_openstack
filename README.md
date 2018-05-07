# [NORA]
__N__ etwork __O__ perations __R__ eference __A__ rchitecture

**Nora’s Mission**

- Provide a base topology for which to deliver a variety of demonstrations based upon customer use cases in an automatic, programmatic, and consistent fashion
- Highlight the “Simple Engineering” of Juniper and open source software; aka Simple AF (Simple Automated Framework)
- Be extensible, reproduceable and awesome
- Inspire the hearts and minds of our customers, partners and Junivators

---
# [Install]

This guide directs the user to spin up a new nora_openstackenvironment. Starting point is the setup of a so called `builder` or `seed` host.
This will be based on Ubuntu Server 16.04.4  LTS. While it is freely available, this software is OpenSourced and considered **use at your own risk and not supported**.

The builder VM will download the required packages and software (unless otherwise noted).
It will build out and install:
- the software all specified servers
- the openstack server nodes consisting of:
  + Controller Node
  + Compute Nodes
  + Wistar Server
  + Image Server
- the virtual environment consisting of:
  + instance images
  + networks
  + routers
  + instances

nora_openstack needs a total of 7 newly installed __Ubuntu Server 16.04.4 LTS__.

- 1 x NORA-builder: &nbsp;&nbsp;&nbsp;&nbsp; NORA builder virtual machine
- 1 x NORA-wistar: &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; NORA wistar create and share network topologies virtual machine
- 1 x NORA-image: &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; NORA image repository server
- 1 x NORA-controller:&nbsp; NORA controller (OpenStack based)
- 3 x NORA-computeX: NORA compute nodes (OpenStack based)

---
# [Requirements]

## [Hardware]

- 1x NORA-controller
    + CPU: &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;24 cores
    + RAM: &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;192.0GiB
    + STORAGE : 12 TB (over 3 disks)
    + NICs: &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;1x 1Gbit/s, 2x 10 Gbit/s
- 3X NORA-compute
    + CPU: &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;24 cores
    + RAM: &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;192.0GiB
    + STORAGE : 12 TB (over 3 disks)
    + NICs: &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;1x 1Gbit/s, 2x 10 Gbit/s
- 1x NORA-builder
    + Virtual Machine
    + CPU: &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;2 cores
    + RAM: &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.0 GiB
    + STORAGE : 20 GiB
    + NICs:&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; 1x 1Gbit/s, 2x 10 Gbit/s
- 1x NORA-wistar
    + Virtual Machine
     + CPU: &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;2 cores
     + RAM: &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.0 GiB
     + STORAGE : 20 GiB
     + NICs: &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;1x 1Gbit/s, 2x 10 Gbit/s
 - 1x NORA-image
    + Virtual Machine
     + CPU: &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;2 cores
     + RAM: &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.0 GiB
     + STORAGE: 30 GiB
     + NICs: &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;1x 1Gbit/s, 2x 10 Gbit/s
>Note: For better performance it's good to go with a hardware based raid controller.

## [Common]
- An internet connection
- Physical servers have to be prepared with __Ubuntu Server 16.04.4 LTS__
- Virtual Machines have to be prepared with __Ubuntu Server 16.04.4 LTS__

>Note: you must have valid name resolution between all the hosts! Add all appropriate host entries to /etc/hosts if you do not have internal DNS

---
# [Configuration]
Starting a new deployment from scratch needs to modify some configuration files. This section describes the parameters in such configuration files. The directory structure configuration files / tools / documentation files are kept in is as follows:

- `nora/ansible`
- `nora/docs`
- `nora/tools`

## [Global variables]
Global variables used during deployment are being kept in `nora/ansible/group_vars/all`. This file is split into several sections. Change into directory  `cd nora/ansible/group_vars/` and edit file `all`.

### [Kolla]
nora_openstack installer uses Openstack's Kolla to deploy cluster nodes. The `[Kolla]` section is used to obtain the right version from Kolla repository.

Example section `[KOLLA]`:
```text
# [KOLLA]
kolla_git_repo: https://github.com/openstack/kolla-ansible.git
kolla_git_branch: master
```

### [OpenStack]
OpenStack section provides configuration options being used during cluster deployment. In ```[OpenStack]``` section we define:

- Physical interface assignment on controller and compute nodes
```text
network_interface: eno1
external_interface: ens2f1
tunnel_interface: ens2f0
```
- **network_interface**: `eno1`
  +  Used for external management and  communication between control and compute nodes
  + Assign IP address to this interface from management network. In this guide we use `10.86.9.0/27` subnet
- **tunnel_interface**: `ens2f0`
  + Carries the overlay traffic (VXLAN)
  + This interface needs jumbo frame support
  + Assign IP address to this interface from tunnel interface network. In this guide we use `10.254.253.0/24` subnet
- **external_interface**: `ens2f1`
  +  Connects overlay network with bare metal network
  + IP address will be assigned during deployment process

>Note: **Change the interface assignment according to your environment**

>Note: If multiple topologies will be created it is important to change __os_external_router_ip__ address.

- Overlay interfaces MTU size. In this case set to support jumbo frames
```text
os_br_int_mtu: 9000
os_br_tun_mtu: 9000
```

- Overlay network to bare metal network settings
```text
os_external_network_name: external_254
os_external_network_subnet_name: external_254_subnet
os_external_network_subnet: 10.254.254.0/24
os_external_network_pool_start: 10.254.254.100
os_external_network_pool_end: 10.254.254.110
os_external_router_name: wistar_router
os_external_router_ip: 10.254.254.253
os_external_gateway_ip: 10.254.254.254
```

- Horizon login credentials
```text
keystone_admin_user: admin
keystone_admin_password: contrail1
keystone_project_name: admin
```

<details><summary>Example of section [OPENSTACK]:</summary>
<pre><code>
#[OPENSTACK]
keystone_admin_user: admin
keystone_admin_password: contrail1
keystone_project_name: admin
network_interface: eno1
external_interface: ens2f1
tunnel_interface: ens2f0
os_br_int_mtu: 9000
os_br_tun_mtu: 9000
os_external_network_name: external_254
os_external_network_subnet_name: external_254_subnet
os_external_network_subnet: 10.254.254.0/24
os_external_network_pool_start: 10.254.254.100
os_external_network_pool_end: 10.254.254.110
os_external_router_name: wistar_router
os_external_router_ip: 10.254.254.253
os_external_gateway_ip: 10.254.254.254
</code></pre>
</details>

### [Wistar]
In ```[Wistar]``` section we define:

- Overlay network settings
```text
wistar_mgmt_network_name: wistar_mgmt
wistar_mgmt_network_subnet_name: wistar_mgmt_subnet
wistar_mgmt_network_subnet: 192.168.128.0/24
wistar_mgmt_network_subnet_pool_start: 192.168.128.3
wistar_mgmt_network_subnet_pool_end: 192.168.128.251
wistar_mgmt_network_prefix: 192.168.128.
wistar_mgmt_network_gateway: 192.168.128.1
```
- Default instance user credentials
```text
wistar_ssh_user: juniper
wistar_public_ssh_key: ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDFbCFkF2qc8f0cgkP2oBN0i8/l5zqgrHEcWbm6Poekh0J/amK/ZWUKUuc1bAH5uXCeHpdpCcYcFMa1mprDftaUvRZaXh9yTOo7drIkLS8ZslbtsMxVGrynu7JMMtfm1tT0BIo7qUFsDbvKqmAIBB4ui0jzLBbYMkFRvo6JFAf47OoG/y5mY5ovwYM0aJ6X7o4QZkXPc5zM90xITtoHwXkUdpNEFMW4AF8ZcLJSZsdQTlqyKxoTHIRMmo2EOqynCvujrczJFoCoYGwfksVxFt3ddRSzLzELO9yV4ksIcNU30CzkYY3igUO/2KpsQY1dJu/OIS2rYJsNFt1B6To/aW1b juniper@coe-srv-01
wistar_default_instance_password: juniper123
```
- Wistar service related settings
```text
wistar_port: 8080
wistar_git_branch: openstack
```

<details><summary>Example of  [Wistar] section</summary>
<pre><code>
#[WISTAR]
wistar_ssh_user: juniper
wistar_public_ssh_key: ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDFbCFkF2qc8f0cgkP2oBN0i8/l5zqgrHEcWbm6Poekh0J/amK/ZWUKUuc1bAH5uXCeHpdpCcYcFMa1mprDftaUvRZaXh9yTOo7drIkLS8ZslbtsMxVGrynu7JMMtfm1tT0BIo7qUFsDbvKqmAIBB4ui0jzLBbYMkFRvo6JFAf47OoG/y5mY5ovwYM0aJ6X7o4QZkXPc5zM90xITtoHwXkUdpNEFMW4AF8ZcLJSZsdQTlqyKxoTHIRMmo2EOqynCvujrczJFoCoYGwfksVxFt3ddRSzLzELO9yV4ksIcNU30CzkYY3igUO/2KpsQY1dJu/OIS2rYJsNFt1B6To/aW1b juniper@coe-srv-01
wistar_port: 8080
wistar_git_branch: openstack
wistar_default_instance_password: juniper123
wistar_mgmt_network_name: wistar_mgmt
wistar_mgmt_network_subnet_name: wistar_mgmt_subnet
wistar_mgmt_network_subnet: 192.168.128.0/24
wistar_mgmt_network_subnet_pool_start: 192.168.128.3
wistar_mgmt_network_subnet_pool_end: 192.168.128.251
wistar_mgmt_network_prefix: 192.168.128.
wistar_mgmt_network_gateway: 192.168.128.1
</code></pre>
</details>



### [Images]
Instance images will be kept on `Image Server` and are accessible through NFS.
For example:

- Juniper vSRX
  + Version: 15.1X49-D120.3
- Juniper vMX
  + Version: 17.4R116
- Juniper vQFX
  + Version: 17.3R17
- Juniper vRR
  + Version: 17.4
- Ubuntu
  + Versions: 16.04 LTS
 - Centos
   + Versions: 6 and 7

We define were to find all the images with following parameters:
- os_image_src: `Image Server IP`
- os_image_src_path: `NFS server export path`
- os_image_local_path: `local file system mount point`

#### [Adding new image entry]
A new entry will  be defined by a `key` which is the filename without file suffix and three attributes:

- image_descr
- image_type
- min_disk

Here is a example for a new entry:

```text
__IMAGE_FILE_NAME__:
    image_descr: __useful_description__
    image_type: __look_at_available_image_types_below__
    min_disk: __min_disk_size_to_boot_the_instance__
```

<details><summary>Available image types</summary>
<pre><code>

	"name": "linux"
	"description": "Linux"

	"name": "ubuntu16"
	"description": "Ubuntu 16"

	"name": "junos_vmx"
	"description": "Junos vMX 14.x"

	"name": "junos_vre"
	"description": "Junos vMX RE Latest"

	"name": "junos_vre_15"
	"description": "Junos vMX RE 15.x"

	"name": "junos_vre_15"
	"description": "Junos vMX RE 16.x"

	"name": "junos_vpfe"
	"description": "Junos vMX vFPC"

	"name": "junos_vpfe_haswell"
	"description": "Junos vMX vFPC (Haswell)"

	"name": "junos_vqfx_re"
	"description": "Junos vQFX RE"

	"name": "junos_riot",
	"description": "Junos vMX RIOT"

	"name": "junos_vrr"
	"description": "Junos Virtual Route Reflector"

	"name": "junos_vqfx_cosim"
	"description": "Junos vQFX PFE"

	"name": "generic"
	"description": "Other"

	"name": "junos_vsrx"
	"description": "Junos vSRX"

	"name": "junos_vmx_hdd"
	"description": "Junos vMX HDD"

	"name": "space"
	"description": "Junos Space"
</code></pre>
</details>


<details><summary>Example of [IMAGES] section:</summary>
<pre><code>
#[IMAGES]
os_image_src: 10.86.9.12
os_image_src_path: /opt/docker/gostatic
os_image_local_path: /tmp/images
os_glance_images:

  vFPC-2017121:
    image_descr: vFPC
    image_type: junos_vpfe
    min_disk:
  vmx174R116:
    image_descr: vMX
    image_type : junos_vre
    min_disk:
  vqfx-173R17:
    image_descr: vQFX
    image_type : junos_vqfx_re
    min_disk:
  vqfx-cosim-2017081510:
    image_descr: vqfx-cosim
    image_type : junos_vqfx_cosim
    min_disk:
  vrr-174:
    image_descr: vRR 17.4
    image_type : junos_vrr
    min_disk:
  vsrx151X49-D120.3:
    image_descr: vSRX 15.1X49-D120.3
    image_type : junos_vsrx
    min_disk:
  centos6:
   image_descr: centos6
   image_type: linux
   min_disk:
  ubuntu1604:
   image_descr: ubuntu1604
   image_type: ubuntu16
   min_disk:
  ubuntu-160402-100G:
   image_descr: ubuntu1604-100G
   image_type: ubuntu16
   min_disk:
</code></pre>
</details>

## [Inventory]
Change into directory  `cd nora/ansible`  and edit file `inventory/hosts`. Make changes to fit your environment.

 ```text
coe-srv-03 ansible_host=10.86.9.14 ansible_connection=ssh
coe-srv-04 ansible_host=10.86.9.15 ansible_connection=ssh
coe-srv-05 ansible_host=10.86.9.16 ansible_connection=ssh
coe-srv-06 ansible_host=10.86.9.17 ansible_connection=ssh
nora-wistar-01 ansible_host=192.168.122.66 ansible_connection=ssh

[seeds]
seed-01 ansible_host=192.168.122.12 ansible_connection=local

[wistar]
nora-wistar-01

[nora-os-cluster]
coe-srv-03
coe-srv-04
coe-srv-05
coe-srv-06

[nora-os-controller]
coe-srv-03

[nora-os-compute]
coe-srv-03
coe-srv-04
coe-srv-05
coe-srv-06
```

>Note: you must have valid name resolution between all the hosts! Add all appropriate host entries to /etc/hosts if you do not have internal DNS

## [Custom Configuration]
Override OpenStack service configuration with a custom config:
- Create `service name` directory in `roles/deploy-nora-seed/templates/custom_conf`
- Create `servicename.conf or .ini` in that directory
- Examples:
    + `roles/deploy-nora-seed/templates/custom_conf/neutron/neutron-server.conf`
    + `roles/deploy-nora-seed/templates/custom_conf/nova/nova-compute.conf`
 - List of service names to map to:

 <details><summary>List of service name mappings:</summary>
<pre><code>
cron
fluentd
glance-api
glance-registry
heat-api
heat-api-cfn
heat-engine
horizon
keystone
kolla-toolbox
mariadb
memcached
neutron-dhcp-agent
neutron-l3-agent
neutron-metadata-agent
neutron-openvswitch-agent
neutron-server
nova-api
nova-compute
nova-conductor
nova-consoleauth
nova-libvirt
nova-novncproxy
nova-scheduler
nova-ssh
openvswitch-db-server
openvswitch-vswitchd
placement-api
rabbitmq
</code></pre>
</details>

---
# [The Build]

Build process consists of four steps as shown in below picture:

- Deploy Builder host
- Deploy OpenStack cluster
- Deploy Wistar host
- Build a topology!

Before the actual deployment can be started we have to prepare:

- Controller node
- Compute nodes
- Wistar host
- Image server host
- Builder host

and all the required configuration settings described in the `[Configuration]` chapter.

## [Prepare]

### [Controller]
Prepare one new `controller` node.

#### [Storage]
- Create a single partition across the first disk `(typically sda)`
- Create RAID 1 across second and third disk and format later with EXT4
  + If no hardware based RAID controller being used create a software RAID 1 and format it with EXT4 `typically a mdX` device

#### [Operating System]
Install `Ubuntu Server 16.04.4 LTS`

#### [Network]
Prepare NICs according to:
- **eno1**
  + IP: __network_interface__ subnet
  + VLAN: untagged
- **ens2f0**
  + IP: __tunnel_interface__ subnet
  + VLAN: untagged
- **ens2f1**
  + IP: __external_interface__ subnet
  + VLAN: untagged

### [Compute]
Prepare three new `compute` nodes.

#### [Storage]
- Create a single partition across the first disk `(typically sda)`
- Create RAID 1 across second and third disk and format later with EXT4
  + If no hardware based RAID controller being used create a software RAID 1 and format it with EXT4 `typically a mdX` device

#### [Operating System]
Install `Ubuntu Server 16.04.4 LTS`

#### [Network]
Prepare NICs according to:
- **eno1**
  + IP: __network_interface__ subnet
  + VLAN: untagged
- **ens2f0**
  + IP: __tunnel_interface__ subnet
  + VLAN: untagged
- **ens2f1**
  + IP: __external_interface__ subnet
  + VLAN: untagged

### [Wistar]
Prepare a new `wistar` virtual machine.

#### [Storage]
Create a single 20 GiB partition and format it with EXT4.

#### [Operating System]
- Install `Ubuntu Server 16.04.4 LTS`
- Create user `juniper` with password `juniper123`
- Login as user `juniper`
- Create user `juniper` ssh keys with `ssh-keygen -t rsa`

#### [Network]
Connect the Wistar virtual machine to the management network on NIC __network_interface__.

### [Builder]
Prepare a new `builder` virtual machine.

#### [Storage]
Create a single 20 GiB partition and format it with EXT4.

#### [Operating System]
- Install `Ubuntu Server 16.04.4 LTS`
  + Create user `juniper` with password `juniper123`
  + Login as user `juniper`
  + Create user `juniper` ssh keys with `ssh-keygen`
  + Update apt cache with `sudo apt-get update`
  + Install `python` with `sudo apt-get install python python-pip -y`
  + Install `ansible` with `sudo apt-get install ansible -y`
  + Update pip `sudo pip install --upgrade pip`
  + Update ansible `sudo pip install --upgrade ansible==2.4.4.0`
    * Current supported ansible version is ```2.4.4.0```
  +  `git clone https://github.com/Juniper/nora.git`
  + Change directory
    * `cd nora/ansible`
  + Modify `group_vars/all` file to fit your environment
  + Modify `inventory/hosts` file to fit your environment

#### [Network]
Connect the Builder virtual machine to the management network on NIC __network_interface__.

## [Deploy]
The deploy process consists of four main steps:

- Provision Builder host
- Provision controller / compute nodes
- Provision network topology elements
- Provision network topology

 First three steps will be initiated on the `Builder` host. Step four will be done with Wistar UI.
 >**Note: All steps on Builder host will be run as none root user. In this example we use user 'juniper'.**

### [Builder]
Deploy new nora_openstack environment with:

>Note: Make sure all the configuration files have been changed to fit your environment before starting the deployment.

- run `ansible-playbook -i inventory/hosts provision-seed.yml`
  + at the end of this play you will be asked for remote user password once to install public ssh keys on all remote hosts
- run `ansible-playbook -i inventory/hosts provision-os.yml`
  + Depending on your internet connection and hosts compute power this step can take several hours
- run `source /etc/kolla/admin-openrc.sh`
- run `ansible-playbook -i inventory/hosts provision-topology.yml`

If deployment was successful OpenStack Horizon UI should be available at `[nora-os-controller] <controller-ip>`
Login credentials are:

- Username set in file  `<group_vars/all>` section `[OPENSTACK]` parameter `keystone_admin_user`
-  Password set in file `<group_vars/all>` section `[OPENSTACK]` parameter `keystone_admin_password`

### [Wistar]
After the third deployment step Wistar host / UI will be accessible with ```http://__wistar_ip__:__wistar_port__``` set in file ```nora/ansible/group_vars/all```  section ```[Wistar]```.


## [Re-Deploy]

### [Builder]
- Rerun deployment from scratch
  + If deployment has to be rerun we need to take care about
    * `known_hosts` file
    * `/etc/hosts` file
    * These files have to be cleaned up manually so far
  + If changes were made to file e.g. `globals.yml` or `multinode` inventory file then
    * run `kolla-ansible destroy -i /etc/kolla/multinode --yes-i-really-really-mean-it`
      * Be aware this destroys the hole Openstack cluster environment
    * run `ansible-playbook -i inventory/hosts provision-seed.yml`
    * run `ansible-playbook -i inventory/hosts provision-os.yml`
    * run `source /etc/kolla/admin-openrc.sh`
    * run `ansible-playbook -i inventory/hosts provision-topology.yml`

## [Deploy Changes]
 - Deploy changes to custom configs we have to options doing so
    + Option 1
      * Make changes to custom config in `/etc/kolla/custom_config`
      * run `kolla-ansible reconfigure -i /etc/kolla/multinode`
    * Option 2
      * Make change to custom config files in `roles/deploy-nora-seed/templates/custom_configs/`
      * run `ansible-playbook -i inventory/hosts provision-seed.yml`
      * Deploy changes by running `kolla-ansible reconfigure -i /etc/kolla/multinode`

- Add additional compute node to environment
  + add new compute node to `/etc/kolla/multinode`
  + check if new node is in `/etc/hosts`
  + run `kolla-ansible -i /etc/kolla/multinode deploy --tags nova --limit <host>`

 ---
# [Version]

- 0.01 - 10-17-17 initial project creation
- 0.02 - 04-11-18 updated installer / builder part

# [Contributing]

Contact Juniper CoE to gain access to #projectnora slack channel, mailing lists, and github repositories.
