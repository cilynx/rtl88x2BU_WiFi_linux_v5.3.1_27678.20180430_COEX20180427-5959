Updated driver for rtl88x2bu wifi adaptors based on rtl88x2BU_WiFi_linux_v5.3.1_27678.20180430_COEX20180427-5959 originally downloaded from [D-Link's download page for the DWA-182 Rev D](https://support.dlink.com/ProductInfo.aspx?m=DWA-182).

Build confirmed on:

```
Linux version 4.18.0-0.bpo.1-amd64 (debian-kernel@lists.debian.org) (gcc version 6.3.0 20170516 (Debian 6.3.0-18+deb9u1)) #1 SMP Debian 4.18.6-1~bpo9+1 (2018-09-13)

gcc (Debian 6.3.0-18+deb9u1) 6.3.0 20170516
```
```
Linux version 4.19.0-rc2-amd64 (debian-kernel@lists.debian.org) (gcc version 8.2.0 (Debian 8.2.0-5)) #1 SMP Debian 4.19~rc2-1~exp1 (2018-09-03)

gcc (Debian 8.2.0-9) 8.2.0
```

# DKMS installation
```bash
cd rtl88x2BU_WiFi_linux_v5.3.1_27678.20180430_COEX20180427-5959
VER=$(sed -n 's/\PACKAGE_VERSION="\(.*\)"/\1/p' dkms.conf)
sudo rsync -rvhP ./ /usr/src/rtl88x2bu-${VER}
sudo dkms add -m rtl88x2bu -v ${VER}
sudo dkms build -m rtl88x2bu -v ${VER}
sudo dkms install -m rtl88x2bu -v ${VER}
sudo modprobe 88x2bu
```
