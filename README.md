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

3. In order for NetQ to work correctly, you need to have the Cumulus Telemetry server installed within Vagrant with the name of "cumulus/ts":

   ```cumulus/ts   (virtualbox, 1.3.0)```

4. Copy the Git repo to your local machine:

    ```git clone https://github.com/chronot1995/int-ansible-training-l3-dc-hostpack-j2```

5. Change directories to the following

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

Helpful NetQ troubleshooting commands:

- netq check interfaces
- netq show interfaces
- netq show lldp
- netq show macs
- netq check bgp
- netq show bgp

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
BGP table version 5
RIB entries 9, using 1368 bytes of memory
Peers 3, using 58 KiB of memory

Neighbor        V         AS MsgRcvd MsgSent   TblVer  InQ OutQ  Up/Down State/PfxRcd
switch02(swp1)  4      65222     105     105        0    0    0 00:04:54            2
switch02(swp2)  4      65222     104     105        0    0    0 00:04:53            2
server01(swp10) 4      65333      13      14        0    0    0 00:00:23            3

Total number of neighbors 3
```

One should see that the corresponding loopback route is installed with two next hops / ECMP.

```
cumulus@switch01:mgmt-vrf:~$ net show route

show ip route
=============
Codes: K - kernel route, C - connected, S - static, R - RIP,
       O - OSPF, I - IS-IS, B - BGP, E - EIGRP, N - NHRP,
       T - Table, v - VNC, V - VNC-Direct, A - Babel, D - SHARP,
       F - PBR,
       > - selected route, * - FIB route

K>* 0.0.0.0/0 [0/0] via 10.0.2.2, vagrant, 00:05:38
C>* 10.0.2.0/24 is directly connected, vagrant, 00:05:38
C>* 10.1.1.1/32 is directly connected, lo, 00:05:38
B>* 10.2.2.2/32 [20/0] via fe80::4638:39ff:fe00:4, swp1, 00:05:36
  *                    via fe80::4638:39ff:fe00:8, swp2, 00:05:36
B>* 10.3.3.3/32 [20/0] via fe80::4638:39ff:fe00:5, swp10, 00:01:04
B>* 10.4.4.4/32 [20/0] via fe80::4638:39ff:fe00:4, swp1, 00:00:04
  *                    via fe80::4638:39ff:fe00:8, swp2, 00:00:04
B>* 192.168.200.0/24 [20/0] via fe80::4638:39ff:fe00:5, swp10, 00:01:04
```

One can also use NetQ to view all of the BGP peers across the network:

```
cumulus@switch01:mgmt-vrf:~$ netq show bgp

Matching bgp records:
Hostname          Neighbor                         VRF              ASN        Peer ASN   PfxRx        Last Changed
----------------- -------------------------------- ---------------- ---------- ---------- ------------ ----------------
server01          eth1(switch01)                   default          65333      65111      4/-/-        1m:34.731s
server02          eth1(switch02)                   default          65444      65222      4/-/-        34.73149s
switch01          swp1(switch02)                   default          65111      65222      4/-/-        6m:6.731s
switch01          swp10(server01)                  default          65111      65333      3/-/-        1m:34.731s
switch01          swp2(switch02)                   default          65111      65222      4/-/-        6m:5.731s
switch02          swp1(switch01)                   default          65222      65111      4/-/-        6m:6.731s
switch02          swp10(server02)                  default          65222      65444      3/-/-        34.73199s
switch02          swp2(switch01)                   default          65222      65111      4/-/-        6m:5.732s
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
    ansible_version=2.6.3

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

5. Within the Vagrantfile, it's important to make sure the oob-mgmt-server should have the following interface configuration:

device.vm.provider "virtualbox" do |vbox|
  vbox.customize ['modifyvm', :id, '--nicpromisc2', 'allow-all']
  vbox.customize ["modifyvm", :id, "--nictype1", "virtio"]
  vbox.customize ["modifyvm", :id, "--nictype2", "virtio"]

This will make sure that the virtio driver is used to improve the speed of downloading NetQ.

6. helper_scripts > auto_mgmt_network > dhcpd.hosts:

Enable "option routers 192.168.200.254" and 8.8.8.8 as the DNS server:

group {

  option domain-name-servers 8.8.8.8;
  option domain-name "simulation";
  option routers 192.168.200.254;
  option www-server 192.168.200.254;
  option default-url = "http://192.168.200.254/onie-installer";

7. helper_scripts > auto_mgmt_network > dhcpd.conf:

This is an ancillary configuration to #6

# OOB Management subnet
shared-network LOCAL-NET{
subnet 192.168.200.0 netmask 255.255.255.0 {
  range 192.168.200.10 192.168.200.50;
  option domain-name-servers 8.8.8.8;
  option domain-name "simulation";
  option routers 192.168.200.254;
  default-lease-time 172800;  #2 days
  max-lease-time 345600;      #4 days
  option www-server 192.168.200.254;
  option default-url = "http://192.168.200.254/onie-installer";
  option cumulus-provision-url "http://192.168.200.254/ztp_oob.sh";
  option ntp-servers 192.168.200.254;
}
