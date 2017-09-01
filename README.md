# CoreNIC Getting Started Guide

Removing Existing Software
When installing coreNIC 1.0 it is necessary to remove any previous installations of Agilio OVS and CoreNIC prior to starting the install procedure.

Uninstalling AOVS
The recommended method to remove AOVS is by running the uninstall script:

/opt/netronome/bin/agilio-ovs-uninstall.sh -y 


Uninstalling Existing CoreNIC
Remove all BSP and Core NIC: apt-get -y remove ns-agilio-core-nic ; apt-get -y remove nfp-bsp.*

apt-get -y remove ns-agilio-core-nic
apt-get -y remove nfp-bsp.*


Installing CoreNIC 1.1 on Ubuntu 16.04
Install Dependencies 
The following dependencies must be installed before installing coreNIC.


Install the prerequisite packages

apt-get install -y make autoconf automake libtool \
  gcc g++ bison flex hwloc-nox libreadline-dev libpcap-dev dkms libftdi1 libjansson4 \
  libjansson-dev guilt pkg-config libevent-dev ethtool libssl-dev \
  libnl-3-200 libnl-3-dev libnl-genl-3-200 libnl-genl-3-dev psmisc gawk \
  libzmq3-dev protobuf-c-compiler protobuf-compiler python-protobuf \
linux-headers-$(uname -r)

Install Netronome Packages

NFP BSP package

Install the NFP BSP rpm package provided by Netronome Support.
dpkg -i nfp-bsp-6000-*.deb


CoreNIC deb package

Next, install the Agilio Core NIC package with the following command

dpkg -i ns-agilio-core-nic_1.*.deb


The installation process does the following: 

Checks the running kernel has the fix for the PCIe quirk-(upstreamed from kernel versions 4.5+)
Builds the ‘nfp’ driver modules for all kernels with quirk_nfp6000 fix on them 
Copies the right firmware into /lib/firmware/netronome 
Checks the flash version in the NFP and prints the commands to update NFP flash if required.


If you see the message below at the end of the installation your NFP device’s flash needs to be updated. Execute the given commands and reboot your machine.

======== NFP requires a new ARM flash image ========
        /opt/netronome/bin/nfp-flash --preserve-media-overrides -w /opt/netronome/flash/flash-nic.bin
        /opt/netronome/bin/nfp-one
        reboot


*Note: If your attempt to update the flash fails with the following message, :

/opt/netronome/bin/nfp-flash: Failed to open NFP device 0 (No such device)

execute the following commands to enable the required card access and rerun the commands (if executing the commands below does not resolve the issue, a reboot is needed):  

rmmod nfp
modprobe nfp nfp_pf_netdev=0 nfp_dev_cpp=1

 

Running Core NIC
View Interfaces 
Once the package has been installed and the card has the correct flash version, new netdev interfaces named nfp_pX will be created e.g. : 

ip link 

18: nfp_p0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc mq state DOWN mode DEFAULT qlen 1000
    link/ether 00:15:4d:12:2c:ce brd ff:ff:ff:ff:ff:ff
19: nfp_p1: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc mq state DOWN mode DEFAULT qlen 1000
    link/ether 00:15:4d:12:2c:cf brd ff:ff:ff:ff:ff:ff


Configure Interfaces
Port Speed

To configure the port speed of the NFP use the following commands. A reboot is required to take affect.

#view current status
nfp-media

#change media mode
nfp-media phy0=10G #set port 0 in 10G mode
nfp-media phy1=25G #set port 1 in 25G mode
nfp-media phy2=4x10G #set port 2 in 4x10G fanout mode

Allocate IP addresses

Assign IP addresses to the respective interfaces on the NFP.

ip address add 10.0.0.1/24 dev nfp_p0 #host_0
ip link set nfp_p0 up  #host_0
ip address add 10.0.0.2/24 dev nfp_p0 #host_1
ip link set nfp_p0 up  #host_1



Checking Connectivity
After you have successfully assigned IP addresses to the NFP interfaces perform a standard ping test to confirm L1-L3 connectivity.

#from host_0
ping 10.0.0.2

PING 10.0.0.2 (10.0.0.2) 56(84) bytes of data.
64 bytes from 10.0.0.2: icmp_seq=3 ttl=64 time=0.319 ms
64 bytes from 10.0.0.2: icmp_seq=4 ttl=64 time=0.317 ms

Basic Performance Test
IPerf3 is a basic traffic generator and network performance measuring tool that allows one to quickly determine the throughput achievable by a device. 

Installing IPerf
apt-get install -y iperf3


Running IPerf
Server

Run IPerf3 on the server.
iperf3 -s 


Client

Execute the following on the client to connect to the server and start running the test.
iperf3 -c 10.0.0.2 -P 10


