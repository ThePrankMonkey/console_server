# console_server

Use a Raspberry Pi 3B+ to create a cheap Console Server for accessing network gear.

## Requirements

| Requirement | Met? |
| --- | --- |
| Total cost under $100. | ??? |
| Connects to four network devices. | ??? |
| Works for Cisco and Juniper.| ??? |
| Individual users for different network equipment. | ??? |
| Logs all output for easy capture of configs. | ??? |
| Acts as wifi access point to work in different environments. | ??? |
| Accessible from homelab network. | ??? |

## Manual Steps

Complete the Raspbian config and fully update.

```bash
# Install packages
sudo apt install screen hostpad dnsmasq -y
```

Set up `/etc/network/infaces/`:

> ```bash
> auto eth0
> allow-hotplug eth0
> iface eth0 inet dhcp
> 
> iface wlan0 inet static
>   address 172.16.1.1
>   netmask 255.255.255.0
>   network 172.16.1.0
>   broadcast 172.16.1.255
> ```

Set up `/etc/hostapd/hostapd.conf`:

> ```bash
> interface=wlan0
> ssid=Console Server
> wpa_passphrase=rootroot
> wpa=3
> ```

Set up `/etc/dnsmasq.conf`

> ```bash
> interface=wlan0
> listen-address=172.16.1.1
> bind-interfaces
> server=8.8.8.8
> domain-needed
> bogus-priv
> dhcp-range=172.16.1.21,172.16.1.24,12h
> ```

Set up '/etc/sysctl.conf`: MIGHT NOT BE NEEDED

> ```bash
> net.ipv4.ip_forward=1
> ```

Set new services to start on boot:

```bash
sudo systemctl enable hostapd
sudo systemctl enable dnsmasq
```

Create network_user account:

```bash
sudo useradd -m network_user
sudo passwd network_user
```

Add the following line with `visudo`:

```bash
# User privilege specification
network_user ALL=(ALL) ALL
```



## BOM

| Item | Cost | Count | Total |
| --- | --- | --- | --- |
| Raspberry Pi 3B+ | $35 | 1 | $35 |
| 16GB MicroSD | $5 | 1 | $5 |
| Raspberry Pi Case | $8 | 1 | $8 |
| USB-A<>RJ45 Console Cable| $9 | 4 | $36 |
| 4-port USB Hub | $10 | 1 | $10 |
| | | __Total__ | $94 |

## Wiring

_TODO:_ Create a Fritzing wiring diagram.

## Custom Case

_TODO:_ Model a custom case for the raspberry pi to be printed.

## Image

_TODO_: Create a premade image to burn to SD-Cards.

## Issues

## References

* [Example](https://networklessons.com/uncategorized/raspberry-pi-as-cisco-console-server) - Similar project.
* [Example2](https://learn.sparkfun.com/tutorials/setting-up-a-raspberry-pi-3-as-an-access-point/set-up-wifi-access-point) - Another project.
* [Ser2Net](https://sourceforge.net/projects/ser2net/) - Allows Serial connections for TCP.
* [Minicom](https://www.cyberciti.biz/tips/connect-soekris-single-board-computer-using-minicom.html) - Another way to use Serial.
* [pi-gen](https://github.com/RPi-Distro/pi-gen) - Used to make raspbian images.
  * [vagrant box](https://app.vagrantup.com/adampie/boxes/pi-gen) - pi-gen in a vagrant box.
* [Raspbian Desktop](https://www.raspberrypi.org/downloads/raspberry-pi-desktop/) - for testing in virtual machine.
