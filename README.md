# int-ansible-training-l3-dc-hostpack-j2

### Summary:

Intermediate Automation - Two Cumulus VX switches and Two Linux Servers running Host Pack and NetQ in a L3 / BGP Unnumbered configuration using Ansible and J2 / Jinja2 templates

### Network Diagram:

![Network Diagram](https://github.com/chronot1995/int-ansible-training-l3-dc-hostpack-j2/blob/master/documentation/int-ansible-training-l3-dc-hostpack-j2.png)

### Initializing the demo environment:

First, make sure that the following is currently running on your machine:

1. Vagrant > version 2.1.2

    https://www.vagrantup.com/

2. Virtualbox > version 5.2.16

    https://www.virtualbox.org

3. Copy the Git repo to your local machine:

    ```git clone https://github.com/chronot1995/int-ansible-training-l3-dc-hostpack-j2```

4. Change directories to the following

    ```int-ansible-training-l3-dc-hostpack-j2```

6. Run the following:

    ```./start-vagrant-poc.sh```

### Running the Ansible Playbook

1. SSH into the oob-mgmt-server:

    ```cd vx-simulation```   
    ```vagrant ssh oob-mgmt-server```

2. Copy the Git repo unto the oob-mgmt-server:

    ```git clone https://github.com/chronot1995/int-ansible-training-l3-dc-hostpack-j2```

3. Change directories to the following

    ```int-ansible-training-l3-dc-hostpack-j2/automation```

4. Run the following:

    ```./provision.sh```

This will bring run the automation script and configure the two switches with BGP.

### Troubleshooting

Helpful NCLU troubleshooting commands:

- net show route
- net show bgp summary
- net show interface | grep -i UP
- net show lldp

Helpful Linux troubleshooting commands:

- ip route
- ip link show
- ip address <interface>

The BGP Summary command will show if each switch has formed an IPv6 and an l2vpn neighbor relationship:

```
cumulus@switch01:mgmt-vrf:~$ net show bgp summary

show bgp ipv4 unicast summary
=============================
BGP router identifier 10.1.1.1, local AS number 65111 vrf-id 0
BGP table version 3
RIB entries 5, using 760 bytes of memory
Peers 2, using 39 KiB of memory

Neighbor        V         AS MsgRcvd MsgSent   TblVer  InQ OutQ  Up/Down State/PfxRcd
switch02(swp1)  4      65222      41      41        0    0    0 00:01:36            2
switch02(swp2)  4      65222      41      43        0    0    0 00:01:36            2

Total number of neighbors 2


show bgp ipv6 unicast summary
=============================

show bgp l2vpn evpn summary
===========================
BGP router identifier 10.1.1.1, local AS number 65111 vrf-id 0
BGP table version 0
RIB entries 3, using 456 bytes of memory
Peers 2, using 39 KiB of memory

Neighbor        V         AS MsgRcvd MsgSent   TblVer  InQ OutQ  Up/Down State/PfxRcd
switch02(swp1)  4      65222      41      41        0    0    0 00:01:36            2
switch02(swp2)  4      65222      41      43        0    0    0 00:01:36            2

Total number of neighbors 2

```

One should see that the corresponding loopback route is installed with two next hops / ECMP.

```
cumulus@switch01:mgmt-vrf:~$ net show route

show ip route
=============
Codes: K - kernel route, C - connected, S - static, R - RIP,
       O - OSPF, I - IS-IS, B - BGP, P - PIM, E - EIGRP, N - NHRP,
       T - Table, v - VNC, V - VNC-Direct, A - Babel,
       > - selected route, * - FIB route

K>* 0.0.0.0/0 [0/0] via 10.0.2.2, vagrant, 00:02:36
C>* 10.0.2.0/24 is directly connected, vagrant, 00:02:36
C>* 10.1.1.1/32 is directly connected, lo, 00:02:36
B>* 10.2.2.2/32 [20/0] via fe80::4638:39ff:fe00:4, swp1, 00:02:33
  *                    via fe80::4638:39ff:fe00:8, swp2, 00:02:33
```

One can also view the MAC addresses of the two switches within the EVPN instance by running the following command:

```
cumulus@switch01:mgmt-vrf:~$ net show evpn mac vni 11

Number of MACs (local and remote) known for this VNI: 2
MAC               Type   Intf/Remote VTEP      VLAN
44:38:39:00:00:05 local  swp10                 11
44:38:39:00:00:01 remote 10.2.2.2
```

### Errata

1. To shutdown the demo, run the following command from the vx-simulation directory:

    ```vagrant destroy -f```

2. This topology was configured using the Cumulus Topology Converter found at the following URL:

    https://github.com/CumulusNetworks/topology_converter

3. The following command was used to run the Topology Converter within the vx-simulation directory:

    ```python2 topology_converter.py int-ansible-training-l3-dc-hostpack-j2.dot -c```

    After the above command is executed, the following configuration changes are necessary:

4. Within ```vx-simulation/helper_scripts/auto_mgmt_network/OOB_Server_Config_auto_mgmt.sh```

The following stanza:

    #Install Automation Tools
    puppet=0
    ansible=1
    ansible_version=2.3.1.0

Will be replaced with the following:

    #Install Automation Tools
    puppet=0
    ansible=1
    ansible_version=2.6.2

The following stanza will replace the install_ansible function:

```
install_ansible(){
echo " ### Installing Ansible... ###"
apt-get install -qy ansible sshpass libssh-dev python-dev libssl-dev libffi-dev
sudo pip install pip --upgrade
sudo pip install setuptools --upgrade
sudo pip install ansible==$ansible_version --upgrade
}```

Add the following ```echo``` right before the end of the file.

    echo " ### Adding .bash_profile to auto login as cumulus user"
    echo "sudo su - cumulus" >> /home/vagrant/.bash_profile
    echo "exit" >> /home/vagrant/.bash_profile

    echo "############################################"
    echo "      DONE!"
    echo "############################################"
