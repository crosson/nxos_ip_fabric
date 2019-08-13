# IP Fabric Ansible Deploy Demo

This repository demonstrates a series of playbooks that automates the managemnet
of a standard CLOS styled IP-Fabric. I hope to demonstrate an ansible based techniques in deploying an nxos based IP fabric. This
lab is built on eve-ng using nxosv images. Configuration will need to be
modified to support your specific 9k model. LACP Does not work in eve-ng. bond interfaces will need to be non LACP based.

## Example Topology

![Topology](example_topology.JPG?raw=true "Topology")

### Summary of Technologies used

* Cisco's anycast gateway
* OSPF with an ip-unnumbered underlay
* MP BGP with EVPN and vxlan
* Support for VPC configured pairs and non VPC switches


## This Demo can:

* Deploy Spines
* Deploy Leaves
* Deploy new VLANS
* Deploy VPC Domains between leaf pairs
* Deploy new VPCs down to server clients

## Methods
This repository has partitioned desired config into seperate roles. This config
is built from ansibles jinja2 templating engine. We then call these roles from
the desired playbook.

## Building a new Fabric

Before building a new fabric this demo assumes that for each switch the
**mgmt0** interface has been configured, that an admin account exists, and that
the control server has SSH access to these switches.

Pre Reqs

* Admin account created
* mgmt0 address added
* ansible control host has ssh access to each leaf

Deployment is done in order

1. Define Leaves and Spines inventory files
2. Create the desired vlans in groupvars fabric file
3. Deploy Spines
4. Deploy Leaves
5. Deploy VPC Domains
6. Add any specific VPC Ports

## Playbooks

### Deploy Spines

Name: deploy_spine.yml

This playbook generates the config for each spine and pushes that config to the
spine.

#### Variables

_HOST_: This variable you define the desired spines to configure. You can
isolate a single spine int his way. Or call all _spines_

Example: spines, spine1

_role_: This hostvar is used to distinguish leaves from spines and dictates that
devices base configuration.

Example: spine

_lo0_: This is a hostvar variable unique to each spine. It defines the spines
lo0 ipv4 address. This address is assumed a host address. A netmask is not
required.

Example: 10.255.1.1

_fabric_links_: This groupvar is a simple list of fabric ports which will be
configured to peer with downlink leaves. This is located in the spines groupvar
file.
Example:

```
fabric_links:
  - "Ethernet1/1"
  - "Ethernet1/2"
  - "Ethernet1/3"
  - "Ethernet1/4"
  - "Ethernet1/5"
  - "Ethernet1/6"
  - "Ethernet1/7"

```

### Deploy Leaves

Name: deploy_leaf.cfg

This playbook generates the config for each leaf and pushes that config to the
leaf.

#### Variables

_HOST_: This variable you define the desired spines to configure. You can
isolate a single leaf int his way. Or call all _leaves_

Example: leaves, leaf4

_role_: This hostvar is used to distinguish leaves from spines and dictates that
devices base configuration.

Example: leaf

_lo0a_: This hostvar defines the leaf's unique lo0 ip address. This address is
unique for each leaf. This address is assumed a host address. A netmask is not
required.

Example: 10.255.2.1

_lo0b_: This hostvar defines the leaf's unique secondary lo0 address used by
vxlan and other processes. Normally this address is unique except for VPC domain
 pairs. This address is assumed a host address. A netmask is not
required.

Example: 10.255.3.1

_fabric_links_: This groupvar is a simple list of fabric ports which will be
configured to peer with uplink spines. This is located in the leaves groupvar
file.

Example:

```
fabric_links:
  - "Ethernet1/1"
  - "Ethernet1/2"
  - "Ethernet1/3"
```

_VLANS_: This fabric wide groupvar defines the list of tenant specific vlans for
the fabric. These are located in the fabric groupvar file.

Example:
```
VLANS:
  - vlan_descr: client_a_l3_vni
    vlan_id: 100
    vn_segment: 16100
    has_svi: false
    vrf: TENANT-1
    svi_address:
    has_dhcp_relay: false
    dhcp_server:
    l3_vnid: true
  - vlan_descr: clients_a_hosts_a
    vlan_id: 101
    vn_segment: 16101
    has_svi: true
    vrf: TENANT-1
    svi_address: "192.168.0.1/24"
    has_dhcp_relay: true
    dhcp_server: "192.168.253.10"
    l3_vnid: false
```

### Setup VPC Domains

Name: deploy_vpc_domains.yml

This playbook connects to each VPC pair in the vpc group. Unique configuration
is assigned based on a subgroup, vpca and vpcb. Your VPC Pairs should be
appropriately organized in these groups.

#### Variables

_local_keepalive_address_: This group var is the A or B side local keepalive
address.

Example: 169.254.255.1

_remote_peer_keepalive_address_: This group var is the A or B side remote
keep alive address.

Example: 169.254.255.2

_keepalive_interface_: This groupvar is the VPC devices peer keepalive
interface.

Example: Ethernet1/5

_peer_link_interfaces_: This groupvar is the VPC peer-link port-channel interface
list.

Example:

```
peer_link_interfaces:
  - "Ethernet1/6"
  - "Ethernet1/7"
```

### Create new VPC Port-channel

Name: add_vpc.yml

This playbook creates new VPC's for vpc domain pairs.

_HOST_: This variable you define the desired VPC pair groups to configure. You
may want to create a VPC for leaf3a and leaf3b. You could just call the group
leaf3.

Example: leaf3

_vpc_port_channels_: This groupvar file defines the list of port-channels for a
vpc domain.

Example:
'''
vpc_port_channels:
  - name: ethernet1/14
    vlan: 102
    channel_group: 10
'''
