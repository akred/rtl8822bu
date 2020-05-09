# 8822BU Wifi driver for Linux

Driver for 802.11ac USB Adapter with
RTL8822BU chipset
Only STA/Monitor Mode is supported, no AP.

A few known wireless cards that use this driver include
* [Deepow Wifi 802.11n/g/b/a/AC (2.4G/300Mbps+5G/867Mbps)](https://www.amazon.fr/Dongle-Wifi-Adaptateur-Wireless-600Mbps/dp/B07NVJLPDD/ref=sr_1_5?keywords=Moglor&qid=1556401118&s=gateway&sr=8-5&th=1)
* [Moglor Wifi AC - 600Mbps (2.4G/150Mbps + 5G/433Mbps)](https://www.amazon.fr/Moglor-Adaptateur-600Mbps-150Mbps-Compatible/dp/B07C4JQZSP/ref=sr_1_2?__mk_fr_FR=%C3%85M%C3%85%C5%BD%C3%95%C3%91&keywords=Moglor&qid=1556720371&s=gateway&sr=8-2)
* [Edimax EW-7822ULC](http://us.edimax.com/edimax/merchandise/merchandise_detail/data/edimax/us/wireless_adapters_ac1200_dual-band/ew-7822ulc/)
* [ASUS AC-53 NANO](https://www.asus.com/Networking/USB-AC53-Nano/)
* [Archer T3U](https://www.tp-link.com/fr/home-networking/adapter/archer-t3u/)

## Platform tested

Linux Mint Tricia 19.3
Currently tested on X86_64 and ARM platform(s) **only**, cross compile possible.

_Update 09/05/2020_ : Test on last LTS Kernel **4.15.0-99-generic**

Card name (lsusb) : **ID 0bda:0129 / Realtek Semiconductor Corp. RTS5129 Card Reader Controller**


## How to install

### DKMS support (recommended)

```
sudo apt-get install git dkms
mkdir ~/rtl8822
cd ~/rtl8822
git clone https://github.com/akred/rtl8822bu.git
sudo dkms add ./rtl8822bu
sudo dkms install 8822bu/1.1
sudo modprobe 8822bu
```

### Manual installation

For compiling type
`make`
in source dir

To install the firmware files
`sudo make install`


## NOTES

To Unload driver you may need to disconnect the device

If the driver fails building consult your distro how to
install the kernel sources and build an <u>external</u> module.

## Troubleshooting

### No wifi after suspend
It is due to the fact that the module is not well supported
For this, you need first to check by lauching this 2 commands :

```
sudo /sbin/modprobe -r 8822bu
sudo /sbin/modprobe  8822bu
```

If the wifi works again, you can create a service to automate this command at each suspend/wakeup

`
sudo -H xed /etc/systemd/system/rtl8822bu-reload.service
`

Put this line inside this file and save it :

```
[Unit]
Description=Reload rtl8822bu wireless lan driver after system resume
After=hibernate.target suspend.target hybrid-sleep.target

[Service]
Type=oneshot
ExecStart=/sbin/modprobe -r 8822bu
ExecStart=/sbin/modprobe 8822bu

[Install]
WantedBy=hibernate.target suspend.target hybrid-sleep.target
```

⚠️ After saving the file, do not forget to launch the command :

`
systemctl enable rtl8822bu-reload.service
`

## MISC

This driver allows use of wpa_supplicant by using the nl80211 driver
`wpa_supplicant -Dnl80211`

If installing on Rasberry Pi or other "armv71" devices, edit the Makefile and set `CONFIG_PLATFORM_ARM_RPI = y` and `CONFIG_PLATFORM_I386_PC = n`

Doc Sources :

- https://forum.ubuntu-fr.org/viewtopic.php?pid=21953042#p21953042
- https://forum.peppermintos.com/index.php/topic,6813.0.html?PHPSESSID=nmsv7brpnk9t8e9apkc6hk52m1

### ⚠️ Secure boot
If you update recently (2019) your kernel, you may have to sign again the driver.
To do it, you can follow [this stackoverflow post](https://askubuntu.com/questions/762254/why-do-i-get-required-key-not-available-when-install-3rd-party-kernel-modules?answertab=votes#tab-top) or disable the UEFI secure boot.
