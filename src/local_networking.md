# Local networking

## Overview

TODO add a network diagram

* Clients on local network should be able to access the Unikernel on your computer.
* The setup is supposed to play nice with an existing installation of Docker.
* Network setup of interfaces and firewall rules are not persistent and will be gone after a reboot

## Network setup

### Enable IP forwarding

* This turns your computer into a router and makes it forward packets between network interfaces
* Might already be enabled if you have Docker installed

```sh
sudo sysctl -w net.ipv4.ip_forward=1
```

### Setup network interfaces

* We setup a bridge `mirageos` on the host system as a point for connecting the Unikernel to the host system
* A bridge is technically not necessary for the use case in this chapter, but we'll do it anyways for consistency
  with interface namings and other conventions in this guide.
* The Unikerel is attached to `tap0`, which is in turn plugged into the bridge to communicate with the host system
* `10.0.0.1` is configured as IP address on the bridge interface and acts as the default gateway for the Unikernel

```sh
sudo ip link add mirageos type bridge
sudo ip tuntap add dev tap0 mode tap
sudo ip link set dev tap0 master mirageos
sudo ip link set dev mirageos up
sudo ip link set dev tap0 up
sudo ip addr add 10.0.0.1/24 dev mirageos
```

### Setup firewall rules

* In the first part we create firewall rules for the `FORWARD` chain, which controls traffic flowing between network interfaces
* NAT rules are required to make the Unikernel reachable on the local network via the IP address of the host system

```sh
sudo iptables --new-chain MIRAGEOS
sudo iptables --append MIRAGEOS --in-interface mirageos --jump ACCEPT
sudo iptables --append MIRAGEOS --out-interface mirageos --jump ACCEPT
sudo iptables --append MIRAGEOS --jump RETURN
sudo iptables --append FORWARD --jump MIRAGEOS

sudo iptables --table nat --new-chain MIRAGEOS
sudo iptables --table nat --append MIRAGEOS --protocol tcp --dport 80 --jump DNAT --to-destination=10.0.0.2
sudo iptables --table nat --append MIRAGEOS --protocol tcp --dport 443 --jump DNAT --to-destination=10.0.0.2
sudo iptables --table nat --append MIRAGEOS --jump RETURN
sudo iptables --table nat --append PREROUTING --jump MIRAGEOS

sudo iptables --table nat --append POSTROUTING --out-interface !mirageos --source 10.0.0.0/24 --jump MASQUERADE
```

### Starting the Unikernel

```sh
sudo solo5-hvt --net:service=tap0 dist/https.hvt --ipv4-gateway=10.0.0.1
```

### Delete firewall setup

```sh
sudo iptables --delete FORWARD --jump MIRAGEOS
sudo iptables --flush MIRAGEOS
sudo iptables --delete-chain MIRAGEOS
sudo iptables --table nat --delete PREROUTING --jump MIRAGEOS
sudo iptables --table nat --flush MIRAGEOS
sudo iptables --table nat --delete-chain MIRAGEOS
```

### Delete network interfaces

```sh
sudo ip link delete dev tap0
sudo ip link delete dev mirageos
```


## Troubleshooting

...
