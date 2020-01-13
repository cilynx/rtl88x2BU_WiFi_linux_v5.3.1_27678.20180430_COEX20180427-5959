# Driver for rtl88x2bu wifi adaptors

Updated driver for rtl88x2bu wifi adaptors based on rtl88x2BU_WiFi_linux_v5.3.1_27678.20180430_COEX20180427-5959 originally downloaded from [D-Link's download page for the DWA-182 Rev D](https://support.dlink.com/ProductInfo.aspx?m=DWA-182).

Build confirmed on:

```bash
Linux Q4 5.3.0-12-generic #13-Ubuntu SMP Tue Sep 17 12:35:50 UTC 2019 x86_64 x86_64 x86_64 GNU/Linux

gcc (Ubuntu 9.2.1-8ubuntu1) 9.2.1 20190909
```
```
Linux xps 5.2.0-2-amd64 #1 SMP Debian 5.2.9-2 (2019-08-21) x86_64 GNU/Linux

gcc (Debian 9.2.1-8) 9.2.1 20190909
```
```bash
Linux 5.0.0-13-generic #14-Ubuntu SMP Mon Apr 15 14:59:14 UTC 2019 GNU/Linux

gcc (Ubuntu 8.3.0-6ubuntu1) 8.3.0
```

## DKMS installation

```bash
cd rtl88x2BU_WiFi_linux_v5.3.1_27678.20180430_COEX20180427-5959
VER=$(sed -n 's/\PACKAGE_VERSION="\(.*\)"/\1/p' dkms.conf)
sudo rsync -rvhP ./ /usr/src/rtl88x2bu-${VER}
sudo dkms add -m rtl88x2bu -v ${VER}
sudo dkms build -m rtl88x2bu -v ${VER}
sudo dkms install -m rtl88x2bu -v ${VER}
sudo modprobe 88x2bu
```

## Raspberry Pi Access Point

```bash
# Update all packages per normal
sudo apt update
sudo apt upgrade

# Install prereqs
sudo apt install git dnsmasq hostapd bc build-essential dkms raspberrypi-kernel-headers

# Reboot just in case there were any kernel updates
sudo reboot

# Pull down the driver source
git clone https://github.com/cilynx/rtl88x2BU_WiFi_linux_v5.3.1_27678.20180430_COEX20180427-5959.git
cd rtl88x2BU_WiFi_linux_v5.3.1_27678.20180430_COEX20180427-5959/

# Configure for RasPi
sed -i 's/I386_PC = y/I386_PC = n/' Makefile
sed -i 's/ARM_RPI = n/ARM_RPI = y/' Makefile

# DKMS as above
VER=$(sed -n 's/\PACKAGE_VERSION="\(.*\)"/\1/p' dkms.conf)
sudo rsync -rvhP ./ /usr/src/rtl88x2bu-${VER}
sudo dkms add -m rtl88x2bu -v ${VER}
sudo dkms build -m rtl88x2bu -v ${VER} # Takes ~3-minutes on a 3B+
sudo dkms install -m rtl88x2bu -v ${VER}

# Plug in your adapter then confirm your new interface name
ip addr

# Set a static IP for the new interface (adjust if you have a different interface name or preferred IP)
sudo tee -a /etc/dhcpcd.conf <<EOF
interface wlan1
    static ip_address=192.168.4.1/24
    nohook wpa_supplicant
EOF

# Clobber the default dnsmasq config
sudo tee /etc/dnsmasq.conf <<EOF
interface=wlan1
  dhcp-range=192.168.4.100,192.168.4.199,255.255.255.0,24h
EOF

# Configure hostapd
sudo tee /etc/hostapd/hostapd.conf <<EOF
interface=wlan1
driver=nl80211
ssid=pinet
hw_mode=g
channel=7
wmm_enabled=0
macaddr_acl=0
auth_algs=1
ignore_broadcast_ssid=0
wpa=2
wpa_passphrase=CorrectHorseBatteryStaple
wpa_key_mgmt=WPA-PSK
wpa_pairwise=TKIP
rsn_pairwise=CCMP
EOF

sudo sed -i 's|#DAEMON_CONF=""|DAEMON_CONF="/etc/hostapd/hostapd.conf"|' /etc/default/hostapd

# Enable hostapd
sudo systemctl unmask hostapd
sudo systemctl enable hostapd

# Reboot to pick up the config changes
sudo reboot
```

If you want to setup masquerading or bridging, check out [the official Raspberry Pi docs](https://www.raspberrypi.org/documentation/configuration/wireless/access-point.md).


## Troubleshooting

### WiFi don't connect
#### Symptoms
- NetworkManager repeatedly says "Authentication required by the wireless network". 
- `journalctl` shows something like
```log
jan 10 13:32:01 galadriel NetworkManager[781]: <warn>  [1578673921.9003] sup-iface[0x55fff7e02323,wlxd01212121212]: connection disconnected (reason 2)
jan 10 13:32:01 galadriel wpa_supplicant[792]: wlxd037453470f5: CTRL-EVENT-SSID-TEMP-DISABLED id=0 ssid="FooBarBaz" auth_failures=1 duration=10 reason=WRONG_KEY
```
#### Solution
Disable new random MAC address generation, like in:

```console
$ cat /etc/NetworkManager/conf.d/no_mac_random.conf
[no-random-mac-88x2bu]
# replace wlxd01212121212 with the name of your device.
match-device=wlxd01212121212
wifi.scan-rand-mac-address=0
wifi.cloned-mac-address=preserve
ethernet.cloned-mac-address=preserve
```
[More info](https://bugs.launchpad.net/ubuntu/+source/network-manager/+bug/1681513)
