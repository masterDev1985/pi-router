# Pi Router
These are some instructions and references that I used to set up a Raspberry Pi 3 as a DNS server, DHCP server, and
router for a group of other Pi's.  The idea was to have a single Pi manage the network for a cluster of other Pis over
its eth0 interface and share its internet connection from its WiFi interface.

## Requirements
This project assumes that you are using a Raspberry Pi 3 with Raspbian installed.  For reference, here are the kernel
and pi firmware versions that were running on my Pi.
```
gencmd output here
```
```
uname -a output here
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
put config text here
```
Because we want to share the internet connection of the wlan0 interface, we must update that interface's configuration 
so that it can connect to our wireless network:
```
put config block here
```

There is also a block for configuring the loopback interface that allows the Pi to talk to itself:
```
put config block here
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
config here
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


## Finished?
If everything went smoothly, you should see output similar to the following for the `route` command:
```
route output here
```

## References
[Raspberry Pi DHCP Tutorial](https://www.raspberrypi.org/learning/networking-lessons/lesson-3/plan/)
[Raspberry Pi DNS Tutorial](https://www.raspberrypi.org/learning/networking-lessons/lesson-4/plan/)
[WiFi to ethernet adapter for an ethernet-ready TV](https://rbnrpi.wordpress.com/project-list/wifi-to-ethernet-adapter-for-an-ethernet-ready-tv/)