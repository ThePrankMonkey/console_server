# console_server

Use a Raspberry Pi 3B+ to create a cheap Console Server for accessing network gear.

- [console_server](#consoleserver)
  - [Requirements](#requirements)
  - [Manual Steps](#manual-steps)
  - [BOM](#bom)
  - [Wiring](#wiring)
  - [Custom Case](#custom-case)
  - [Image](#image)
  - [Issues](#issues)
    - [Binding same dev path to device on reboot](#binding-same-dev-path-to-device-on-reboot)
  - [References](#references)

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
sudo apt install screen hostapd dnsmasq -y
```

Set up `/etc/network/interfaces`:

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
> # Change these values
> interface=wlan0
> ssid=Console Server
> wpa_passphrase=rootroot
> wpa=3
> ```

Set up `/etc/dnsmasq.conf`

> ```bash
> # Change these values
> interface=wlan0
> listen-address=172.16.1.1
> dhcp-range=172.16.1.21,172.16.1.24,12h
> # Add these values
> bind-interfaces
> server=8.8.8.8
> domain-needed
> bogus-priv
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
PASSWORD='rootroot'
sudo useradd -s /bin/bash -m network_user
echo -e "$PASSWORD\n$PASSWORD" | sudo passwd network_user
```

Add the following line with `visudo`:

```bash
# User privilege specification
network_user ALL=(ALL) ALL
```

Collect the idVendor, idProduct, and serial properties of the console cables. With one cable plugged in at a time, run:

```bash
#NOTE /dev/ttyUSB0 and head -4 may need to change if your have different hardware.
udevadm info -a -p  $(udevadm info -q path -n /dev/ttyUSB0) | grep -E '(idProduct|idVendor|serial)' | head -4
```

Use the previous values to label each cable.

Create `/etc/udev/rules.d/99-usb-serial.rules` with the following information:

```text
SUBSYSTEM=="tty", ATTRS{idVendor}=="0403", ATTRS{idProduct}=="6001", ATTRS{serial}=="REPLACE_ME1", SYMLINK+="console_cable0"
SUBSYSTEM=="tty", ATTRS{idVendor}=="0403", ATTRS{idProduct}=="6001", ATTRS{serial}=="REPLACE_ME2", SYMLINK+="console_cable1"
SUBSYSTEM=="tty", ATTRS{idVendor}=="0403", ATTRS{idProduct}=="6001", ATTRS{serial}=="REPLACE_ME3", SYMLINK+="console_cable2"
SUBSYSTEM=="tty", ATTRS{idVendor}=="0403", ATTRS{idProduct}=="6001", ATTRS{serial}=="REPLACE_ME4", SYMLINK+="console_cable3"
```

Copy the contents of `newuser.sh` into network_user's `.bash_profile`.

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

### Binding same dev path to device on reboot

Using `usb-devices`, I can see the Serial Numbers for my usb console cables.

Should be able to configure udevadm rules via [this](https://unix.stackexchange.com/questions/66901/how-to-bind-usb-device-under-a-static-name).

## References

* [Example](https://networklessons.com/uncategorized/raspberry-pi-as-cisco-console-server) - Similar project.
* [Example2](https://learn.sparkfun.com/tutorials/setting-up-a-raspberry-pi-3-as-an-access-point/set-up-wifi-access-point) - Another project.
* [Ser2Net](https://sourceforge.net/projects/ser2net/) - Allows Serial connections for TCP.
* [Minicom](https://www.cyberciti.biz/tips/connect-soekris-single-board-computer-using-minicom.html) - Another way to use Serial.
* [pi-gen](https://github.com/RPi-Distro/pi-gen) - Used to make raspbian images.
  * [vagrant box](https://app.vagrantup.com/adampie/boxes/pi-gen) - pi-gen in a vagrant box.
* [Raspbian Desktop](https://www.raspberrypi.org/downloads/raspberry-pi-desktop/) - for testing in virtual machine.
