# Rpi_AccessPoint_and_Client
Rpi3B configuration to use it as WiFi AP and Client simultaneously on the same onboard adapter of the Pi, this also allows the pi to share its internet conection (as a repeater), you can connect to your RPi without a router nor ethernet, or operate "headless" or "WirelessHead" with the use of VNC server and have access to the desktop with a tablet or phone with VNC connect

## Configure WiFi client
open /etc/wpa_supplicant/wpa_supplicant.conf and set ssid and pass of your router

## Install hostapd and dnsmasq
in shell: 
```
sudo apt-get update
sudo apt-get install hostapd dnsmasq
```

## Configure static ip in uap0 
Content of /etc/dhcpcd.conf
```
interface uap0
 static ip_address=192.168.50.1/24
```

## Configure HostAP

Content of /etc/hostapd/hostapd.conf

```
interface=uap0
#driver=nl80211
ssid=PiAP
hw_mode=g
channel=1
wmm_enabled=0
macaddr_acl=0
auth_algs=1
ignore_broadcast_ssid=0
wpa=2
wpa_passphrase=yourPassword
wpa_key_mgmt=WPA-PSK
wpa_pairwise=TKIP
rsn_pairwise=CCMP
```


** Set the SSID and pass you want the default would be "PiAP" and "yourPassword" respectively

## Configure the daemon for the host AP
Add the following to /etc/default/hostapd

`DAEMON_CONF="/etc/hostapd/hostapd.conf"`

## Configure AP dns
Content of /etc/dnsmasq.conf
```
interface=lo,uap0
dhcp-range=192.168.50.50,192.168.50.150,12h 
```

## Configure forwarding
Uncomment the next line in /etc/sysctl.conf

`net.ipv4.ip_forward=1`

## Configure Startup
Add to /etc/rc.local 
```
service hostapd stop
service dnsmasq stop
service dhcpcd stop
iw dev wlan0 interface add uap0 type __ap
iptables -t nat -A POSTROUTING -o wlan0 -j MASQUERADE
service hostapd start
service dnsmasq start
service dhcpcd start
```

## Reboot
`sudo reboot`
