# Pi Router
These are some instructions and references that I used to set up a Raspberry Pi 3 as a DNS server, DHCP server, and
router for a group of other Pi's.  The idea was to have a single Pi manage the network for a cluster of other Pis over
its eth0 interface and share its internet connection from its WiFi interface.

## Requirements
This project assumes that you are using a Raspberry Pi 3 with Raspbian installed.  For reference, here are the kernel
and pi firmware versions that were running on my Pi.
```
pi@raspberrypi:~ $ vcgencmd version
Aug 30 2016 17:02:01 
Copyright (c) 2012 Broadcom
version c60d00082957b5072ae3de71484f03f0aec21628 (clean) (release)
```
```
pi@raspberrypi:~ $ uname -a
Linux raspberrypi 4.4.17-v7+ #902 SMP Mon Aug 15 12:21:29 BST 2016 armv7l GNU/Linux
```

### DNS and DHCP Configuration
`dnsmasq` is the tool that I used to provide DNS and DHCP services to the Pi network.  Install dnsmasq with the
following commands:
```shell
sudo apt-get update
sudo apt-get install dnsmasq
```

Next, we need to reconfigure the eth0 interface on the Pi to a use a static IP address.  Open the network interfaces
configuration file:
```shell
sudo nano /etc/network/interfaces
```
This file should be updated to match the configuration in this project's configuration file.  We basically configure the
that configures the eth0 interface:
```
auto eth0
iface eth0 inet static
address 192.168.4.1
netmask 255.255.255.0
network 192.168.4.0
broadcast 192.168.4.255
gateway 192.168.4.1
```
Because we want to share the internet connection of the wlan0 interface, we must update that interface's configuration 
so that it can connect to our wireless network:
```
allow-hotplug wlan0
iface wlan0 inet dhcp
  wpa-ssid "Your network name here"
  wpa-psk "Your passcode here"
  wpa-group TKIP CCMP
  wpa-key-mgmt WPA-PSK

```

There is also a block for configuring the loopback interface that allows the Pi to talk to itself:
```
auto lo
iface lo inet loopback
```

After reconfiguring the network interfaces, the networking service will need to be restarted:
```shell
sudo service networking restart
```

Next, we need to reconfigure the dnsmasq service to support DHCP and DNS.  Open the dnsmasq config file with the
following command:
```shell
sudo nano /etc/dnsmasq.conf
```

Update the contents of the file to match the configuration file in this project:
```
interface=eth0

# Specify the address range and lease time for DHCP (leave 2-50 for static IPs)
dhcp-range=192.168.4.51,192.168.4.254,255.255.255.0,12h

# Hand out DNS server information at the same time as IP addresses
dhcp-option=6,192.168.4.1

# Use a text file for DNS host name storage
no-hosts  # ignore /etc/hosts
addn-hosts=/etc/hosts.dnsmasq

# Fallback to Google DNS after checking hosts.dnsmasq
server=8.8.8.8
server=8.8.4.4
```

This config file specifies that the file `/etc/hosts.dnsmasq` will contain a list of hostnames for the DNS service.  An
example is included in this project.

After saving the file, restart the dnsmasq service:
```shell
sudo service dnsmasq restart
```

The file `/etc/resolve.conf` needs to be updated so that DNS requests are forwarded to the local DNS service.  Update
the file to match the one in the project.  It should only have the following line:
```
nameserver 127.0.0.1
```

Reboot the network service again.

`setupiptables.sh` should set up some iptables rules to allow the traffic from the eth0 interface to make its way
through the wlan0 interface to the internet.  

You might also need to reconfigure the kernel routes for traffic to flow. You should see output similar to the following for the `route` command:
```
pi@raspberrypi:~ $ route
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
default         dsldevice.attlo 0.0.0.0         UG    303    0        0 wlan0
192.168.1.0     *               255.255.255.0   U     303    0        0 wlan0
192.168.4.0     *               255.255.255.0   U     0      0        0 eth0
```

## References
[Raspberry Pi DHCP Tutorial](https://www.raspberrypi.org/learning/networking-lessons/lesson-3/plan/)
[Raspberry Pi DNS Tutorial](https://www.raspberrypi.org/learning/networking-lessons/lesson-4/plan/)
[WiFi to ethernet adapter for an ethernet-ready TV](https://rbnrpi.wordpress.com/project-list/wifi-to-ethernet-adapter-for-an-ethernet-ready-tv/)