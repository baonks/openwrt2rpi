# lede2rpi
Linux shell script for downloading and converting LEDE image to Raspberry Pi NOOBS/PINN installer format.

Script generates files:
- [BOOT_PART_LABEL].tar.xz
- [ROOT_PART_LABEL].tar.xz
- partition_setup.sh
- partitions.json
- os.json
- [LEDE_OS_NAME].png

in directory [WORKING_DIR]/lede2R[RASPBERRY_MODEL]_[LEDE_RELEASE]/LEDE

for example: lede2RPi3_snapshot/LEDE for Pi3 snapshot

Directory LEDE can be copied to SD card into /os folder for NOOBS/PINN installer.

Optionally script downloads selected modules to root partition and creates initial configuration script.

Tested on Ubuntu 16.04 with Raspberry Pi 3 and NOOBS 2.4.0 / PINN 2.3.1a BETA

## Usage
```
$ lede2rpi.sh -m RASPBERRY_MODEL -r LEDE_RELEASE [OPTIONS]

OPTIONS:

-m RASPBERRY_MODEL
   RASPBERRY_MODEL=Pi|Pi2|Pi3, mandatory parameter

-r LEDE_RELEASE
   LEDE_RELEASE=snapshot|17.01.0|17.01.0|future_release_name, mandatory parameter

-d WORKING_DIR
   WORKING_DIR=<working_directory_path>, optional parameter, default=/tmp/
   Directory to store temporary and final files.

-p
   optional parameter
   Pause after boot and root partitions mount. You can add/modify files on both partitions in /media/$USER/[UUID] directories.

-a MODULES_LIST
   MODULES_LIST='module1 module2 ...', optional parameter
   List of modules to download and copy to root image into MODULES_DESTINATION directory

-b MODULES_DESTINATION
   MODULES_DESTINATION=<ipk_directory_path>, optional parameter, default=/root/ipk/
   Directory on LEDE root partition to copy downloaded modules from MODULES_LIST

-s INITIAL_SCRIPT_PATH
   INITIAL_SCRIPT_PATH=<initial_script_path>, optional parameter, default=none
   Path to store initial configuration script on LEDE root partition. Example: /root/init_config.sh

-i INCLUDE_INITIAL_FILE
   MODULES_LIST=<include_initial_script_path>, optional parameter
   Path to local script, to be included in initial configuration script INITIAL_SCRIPT_PATH.

-c
   optional parameter, default=no autorun initial script
   Run initial script INITIAL_SCRIPT_PATH once. Path to initial script will be added do /etc/rc.local and removed after first run.

-k LEDE_BOOT_PART_SIZE
   LEDE_BOOT_PART_SIZE=<boot_partition_size_in_mb>, optional parameter, default=25
   LEDE boot partition size in MB.

-l LEDE_ROOT_PART_SIZE
   LEDE_ROOT_PART_SIZE=<root_partition_size_in_mb>, optional parameter, default=300
   LEDE root partition size in MB.

-n BOOT_PART_LABEL
   BOOT_PART_LABEL=<boot_partition_label>, optional parameter, default=LEDE_boot
   LEDE boot partition label.

-e ROOT_PART_LABEL
   ROOT_PART_LABEL=<root_partition_label>, optional parameter, default=LEDE_root
   LEDE root partition label.

-o LEDE_OS_NAME
   LEDE_OS_NAME=<lede_os_name>, optional parameter, default=LEDE
   LEDE os name in os.json

-q
   optional parameter, default=no quiet
   Quiet mode.

-v
   optional parameter, default=no verbose
   Verbose mode.
```

### Examples

Download LEDE release 17.01.1 for RPi 3 and convert to NOOBS with default parameters:
```
$ lede2rpi.sh -m Pi3 -r 17.01.1
```

Download LEDE release 17.01.0 for RPi 2 and convert to NOOBS with default parameters:
```
$ lede2rpi.sh -m Pi2 -r 17.01.0
```

Download LEDE development snapshot for RPi 1 and convert to NOOBS with quiet mode:
```
$ lede2rpi.sh -m Pi -r snapshot -q
```

Download LEDE release 17.01.1 for Raspberry Pi 3, be verbose, use working dir ~/tmp/, create initial config script in /root/init_config.sh and run it once (through rc.local), include local init script ~/tmp/my_lede_init.sh, download modules for HiLink modem and nano to /root/ipk directory and pause befor making final files. Boot partition will have a size of 30 MB and the root partition will have a size of 400 MB. Final files will be created in the directory ~/tmp/lede2RPi3_17.01.1/LEDE.
```
$ lede2rpi.sh -m Pi3 -r 17.01.1 -d ~/tmp/ -v -p -c -s /root/init_config.sh -i ~/tmp/my_lede_init.sh -b /root/ipk -a "kmod-usb2 librt libusb-1.0 usb-modeswitch kmod-mii kmod-usb-net kmod-usb-net-cdc-ether terminfo libncurses nano"  -k 30 -l 400
```

Sample local init file ~/tmp/my_lede_init.sh. Script sets local IP address, timezone (Warsaw), enables WPA2 secured Wifi AP and sets USB HiLink modem as wan interface. Finally waits 30 sec and reboots RPi.
```
LAN_IP="192.168.10.1"

uci set system.@system[0].timezone=CET-1CEST,M3.5.0,M10.5.0/3
uci commit system

uci set wireless.@wifi-device[0].disabled=0
uci set wireless.@wifi-device[0].channel=11
uci set wireless.@wifi-iface[0].ssid=DarthVader
uci set wireless.@wifi-iface[0].encryption=psk2
uci set wireless.@wifi-iface[0].key=secretpassword
uci commit wireless

uci set network.lan.ipaddr=${LAN_IP}
uci del network.wan
uci set network.wan=interface
uci set network.wan.proto=dhcp
uci set network.wan.ifname=eth1
uci commit network

echo "
${LAN_IP} ruter ruter.lan ruter.local
" >> /etc/hosts

sync
sleep 30
sync
reboot
```

## Requirements
```
sudo apt install kpartx
```
## To do / roadmap
- script to upgrade existing LEDE instalation on NOOBS/PINN SD card.

## License

This project is licensed under the GPLv3 License - see the [LICENSE](LICENSE) file for details

