# An IP Fabric NXOS and ansible demonstration

This repo will demonstrate a traditional leaf and spine network topology with
Cisco NXOS as the OS ansible as the orchestration tool for deployment.

## Fabric Design

This fabric uses the following technology.

* Any-cast gateway
* VXLAN
* Ingress Replication
* IP Unnumbered
* OSPF
* MPBGP EVPN

Using an ip-Unnumbered approach with the underlay radically simplifies our
configs. Each switch can now be identified by a single ip address. This means we
have a unified entry in our DNS that will always get us to the Lo0 of the
switch. This address is also the router ID.

Additionally we avoid having to fabricate a complicated IP scheme and we
improve our troubleshooting time since switches become easily identifiable by
a single address. Spines will have only two addresses. The Lo0 address and The
mgmt0 interface. Leaves will have 3 addresses. Lo0 for routing, a secondary for
the tep, and its mgmt0.

To make the fabric example more realistic we add several hosts in varying VLANS.
We add a DHCP server and a VPC leaf scenario as well as a non VPC switch
scenario.

## Ansible High Level

This example uses roles to process the configuration jinja2 templates. We then
push this config with the nxos_config module of ansible.

# Example deploy
