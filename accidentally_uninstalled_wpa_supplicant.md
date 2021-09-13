Scenario
--------
- Accidentally uninstalled `wpa_supplicant` because I didn't know it's the one used to do networking in Ubuntu (20.04.3 LTS (Focal Fossa)).
```
$ sudo apt uninstall wpa_supplicant
$ sudo apt autoremove
```
- My laptop cannot connect to network anymore because NetworkManager is down due to `wpa_supplicant` being removed, and the laptop got no LAN cable interface too >_<.

Pre-requisites
--------------
- From a separate computer with internet download the following files:
  * `wpasupplicant_2.9-1ubuntu4_amd64.deb`
  * `libpcsclite1_1.8.26-3_amd64.deb`

Steps
-----

- Install the required `deb` packages:
```
$ sudo dpkg -i libpcsclite1_1.8.26-3_amd64.deb
$ sudo dpkg -i wpasupplicant_2.9-1ubuntu4_amd64.deb
```

- Restart NetworkManager to autodetect the `wpa_supplicant`:
```
$ sudo systemctl restart NetworkManager.service
```

- Check the name of WIFI interface:
```
$ sudo iw dev
```

- It should show something like `wlp1s0`. Then try to scan for available WIFI nearby.
```
$ sudo ip link show wlp1s0
$ sudo iw wlp1s0 link
$ sudo iw wlp1s0 scan
```

- If not connected or down then turn it on using:
```
$ sudo ip link set wlp1s0 up
$ sudo iw wlp1s10 link
```

- Then search for available WIFI nearby again and note its SSID. The list might be long so you can put it in a file first then search from there:
```
$ sudo iw wlp1s0 scan > ssids.txt
```

- Note the SSID. For the commands below let's assume it's `ZTE_5G_TYeASQ` for example. Restart NetworkManager for now:
```
$ sudo systemctl restart NetworkManager.service
```

- Log in to root then connect to the SSID with passphrase:
```
$ sudo su
$ wpa_passphrase ZTE_5G_TYeASQ >> /etc/wpa_supplicant.conf YOUR_WIFI_PASSWORD_HERE
$ cat /etc/wpa_supplicant.conf
$ lsmod | grep iwlagn
$ lshw -C network
```

- Show the available WIFI drivers:
```
$ wpa_supplicant
```

- Then it should show the ff drivers for example:
```
drivers:
  nl80211 = Linux nl80211/cfg80211
  wext = Linux wireless extensions (generic)
  wired = Wired Ethernet driver
  macsec_linux = MACsec Ethernet driver for Linux
  none = no driver (RADIUS server/WPS ER)
```

- Then enable the WIFI driver:
```
$ wpa_supplicant -B -D nl80211 -i wlp1s0 -c /etc/wpa_supplicant.conf
```

- Request for IP Address from WIFI's DHCP server:
```
$ sudo dhclient wlp1s0 -v -r wlp1s0 // to release first
$ sudo dhclient wlp1s0 -v -4 wlp1s0 // option -4 is to request for v4 IP Address scheme
```

- On a separate terminal check if you have IP Address:
```
$ sudo ifconfig
```

- For troubleshooting, you can restart your WIFI interface:
```
$ sudo ifconfig wlp1s0 down
$ sudo ifconfig wlp1s0 up
```

- If IP Address is received, you are now connected to internet. You can download NetworkManager:
```
$ sudo apt-install network-manager
$ sudo ifconfig wlp1s0 down
$ sudo ifconfig wlp1s0 up
$ sduo systemctl restart NetworkManager.service
```

- By this time Network and Wifi in Settings GUI should show up.

- Install VPN Manager too if you need:
```
$ sudo apt-get install network-manager-openvpn network-manager-openvpn-gnome
$ sudo systemctl restart NetworkManager.service
```
