# Connect to a Raspberry Pi without any additional peripheral devices

It is assumed that you have a `Raspberry Pi` with a wireless network interface and a microSD card with a Raspbian distribution (`2020-02-13-raspbian-buster`), although this also works on other Raspbian distributions.

Insert the microSD card into your operating system to make a few preliminary changes. You should see two partitions: `boot` and `rootfs`. Inside the `boot` partition, create a file and name it `ssh`, and add the value "1" as its content. This will enable the SSH protocol, allowing you to access the system remotely when the operating system loads.

```bash
$ cd /media/myuser/boot/
$ sudo touch ssh
$ echo "1" > ssh
```

Inside the `rootfs` partition, we edit the `wpa_supplicant.conf` file located in the `/etc/wpa_supplicant` directory, adding the network configuration so that it connects to our Wi-Fi automatically when the operating system starts.

```
ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
update_config=1
country=ES

network={
   ssid="NETWORK-NAME"
   psk="NWTWORK-PASSWORD"
   key_mgmt=WPA-PSK
}
```

We remove the microSD card from our computer and insert it into the Raspberry Pi. Then, we connect the Raspberry Pi to the power supply to boot the operating system. Since we don't have a monitor, we wait approximately 2 minutes for it to finish loading.

To connect to the Raspberry Pi, we need to find out which IP address it has acquired. There are several ways to do this: we can access the router’s settings and look for the Raspberry Pi among the connected devices, or try different IP addresses until we find the correct one. In small networks, this shouldn’t take long. We will use the `nmap` command, replacing the network address with the one used by our network which in my case is 192.168.0.1
```bash
sudo nmap 192.168.0.100-255
```

Among the results returned by this command, we should see something similar to the following, which indicates that the Raspberry Pi is located at the address `192.168.0.101`.
```
Nmap scan report for 192.168.0.101
Host is up (0.019s latency).
Not shown: 999 closed ports
PORT   STATE SERVICE
22/tcp open  ssh
MAC Address: B8:27:EB:FF:3A:EC (Raspberry Pi Foundation)
```

We connect to the Raspberry Pi using SSH from our PC, with the username  `pi` and the password `raspberry`
```bash 
$ ssh pi@192.168.0.101
```

Once connected to the Raspberry Pi, we can perform the rest of the configurations using the command:
```
sudo raspi-config
```
