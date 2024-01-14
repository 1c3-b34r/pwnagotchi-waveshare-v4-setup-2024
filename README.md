# pwnagotchi-waveshare-v4-setup-2024

## pick right version

use latest release from here : https://github.com/aluminum-ice/pwnagotchi/releases

## add config.toml file

config.toml file:

main.name = "pwnagotchi"
main.lang = "en"
main.whitelist = [
  "EXAMPLE_NETWORK",
  "ANOTHER_EXAMPLE_NETWORK",
  "fo:od:ba:be:fo:od",
  "fo:od:ba"
]

main.plugins.grid.enabled = true
main.plugins.grid.report = false
main.plugins.grid.exclude = [
  "YourHomeNetworkHere"
]

ui.display.enabled = true
ui.display.type = "waveshare213inb_v4"
ui.display.color = "black"

## configure network sharing

### on pi
ssh into pi

edit /etc/resolv.conf from 127.0.0.1 to any DNS (8.8.8.8)

### on macbook:

1. Plug in Pwnagotchi
2. System Preferences > Network > RNDIS/Ethernet Gadget > Details
3. TCP/IP
Configure IPv4: Manually
IP address: 10.0.0.1
Subnet Mask: 255.255.255.0
Router: 10.0.0.1
4. System Preferences > General > Sharing > Internet Sharing: Toggle OFF
5. HERE'S THE TRICK: System Preferences > Network > Hit the "... V" looking button and choose "Set Service Order". Make sure Wi-Fi is above RNDIS/Ethernet Gadget
6. Run sudo ./macOS_connection_share.sh en0 en7

macos_connection_share.sh file:

#!/usr/bin/env bash

UPSTREAM_IFACE=${1:-en0}
USB_IFACE=''
USB_IP=${2:-10.0.0.1}

for i in $(ifconfig -lu); do
  if ifconfig $i | grep -q "${USB_IP}" ; then USB_IFACE=$i; fi;
done

if [ -z "$USB_IFACE" ]
then
  echo "can't find usb interface with ip $USB_IP"
  exit 1
fi

echo "sharing connecting from upstream interface $UPSTREAM_IFACE to usb interface $USB_IFACE ..."

sysctl -w net.inet.ip.forwarding=1
pfctl -e
echo "nat on ${UPSTREAM_IFACE} from ${USB_IFACE}:network to any -> (${UPSTREAM_IFACE})" | pfctl -f -

./connection_sharing.sh en0 en8

