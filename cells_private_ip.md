When starting `cells`, you see this warning in your logs:

```sh
Warning: no private IP detected for binding broker. Will bind to <YOUR PUBLIC IP ADDRESS>, which may give public access to the broker.
```

You should solve this issue by adding a virtual NIC  on your machine or you might be at risk (depending onn how secure is your network).

## Overview

Internally, Pydio Cells is implemented with a microservice oriented architecture: each simple feature is implemented as an independant brick that exposes a set of internal APIs inside Cells.

All microservices communicate together via gRPC, a HTML2 based protocol. It is important that this communication happens on a private network for better security.

On single instance servers, this is done by using a private IP of the server.

### How to fix

There are many ways to declare a private virtual interface. A few examples:

#### With docker

If you have `docker` installed on your machine, launch the service:  
the docker daemon creates a virtual private network that can be used by Cells.

#### On Debian like

If your main interface is called `eno1`, make a backup of your `/etc/network/interfaces` file and append this:

```conf
# virtual IP on eno1
auto eno1:0
iface eno1:0 inet static
address 10.0.0.1
netmask 255.255.255.0
network 10.0.0.0
broadcast 10.0.0.255
```

From your comand line, mount the virtual NIC and list all:

```sh
sudo ifup eno1:0
sudo ip address
```

#### On RHEL like

Your main interface is called `eth0`, adapt otherwise.

```sh
sudo tee /etc/sysconfig/network-scripts/ifcfg-eth0:0 << EOF
DEVICE="eth0:0"
ONBOOT="yes"
BOOTPROTO="static"
IPADDR="10.0.0.1"
NETMASK="255.255.255.0"
BROADCAST="10.0.0.1"
EOF

# Start the interface and check new IP
sudo ifup eth0:0
sudo ip address
```
