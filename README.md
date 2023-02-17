# Openwrt Wavlink Quantum QAX
Configure Openwrt in Wavlink Quantum DAX (almost the same as Phicomm K3)

![image](https://user-images.githubusercontent.com/47162315/219511130-f9022e69-64c0-4c48-abee-db2a2f43987c.png)

## Some notes: ##
- THIS PROCEDURE WILL VOID YOUR WARRANTY AND CAN BRICK YOUR ROUTER!!
- This document explains the procedure I followed to install Openwrt in my router and I'm don't encourage you to do the same if you're not comfortable with the possibility of damaging your router. Every piece of hardware can be different...
- I'm not responsible for any problem occurred when following this steps
- This instructions were made to run in a Linux based PC
- This seems to be a nice piece of hardware for the price (you can find it for around €30)
- This device seems to be a rebrand of Phicomm K3 (with some protection against changing the firmware, of course...)
  - https://openwrt.org/toh/phicomm/k3_a1
  - https://openwrt.org/toh/hwdata/phicomm/phicomm_k3_a1
- After flashing Openwrt the screen options will not be available anymore (only Wavlink logo with progress bar will be shown)
- Please remember: THIS PROCEDURE WILL VOID YOUR WARRANTY AND CAN BRICK YOUR ROUTER!!

## Connection Diagram ##
To transfer the openwrt firmware file, we'll need to connect to the router in 2 ways:
- A serial port connection to access the router terminal
- a TFTP connection to transfer the firmware file (file transfer via serial is painfuly slow...)

![image](https://user-images.githubusercontent.com/47162315/219512908-44c15271-a1ba-4dbe-84af-b80f89e99f2c.png)

## Download latest Openwrt firmware and prepare PC ##
- Create a folder to serve files over tftp (ie. /tmp/tftp)
- Download latest TRX file from (ie. 22.03.3): 
  - https://downloads.openwrt.org/releases/22.03.3/targets/bcm53xx/generic/openwrt-22.03.3-bcm53xx-generic-phicomm_k3-squashfs.trx
- Copy previously downloaded file to tftp folder
- Rename file to openwrt.trx (to simplify process)

## Setup PC connection ##
- Connect PC via ethernet cable to router LAN port (NOT WAN!!!)
- Setup PC with static IP 192.168.10.X (ie. 192.168.10.100)

## Setup tftp Server ##
- Start dnsmasq tftp server: 
  - `sudo dnsmasq --port=0 --enable-tftp --tftp-root=/tmp/tftp --tftp-no-blocksize --user=root --group=root`

## Setup minicom in PC: ##
- Install minicom tool: 
  - `sudo apt install minicom`
- Edit configs: 
  - `sudo minicom -s`
- Choose "Serial Port Setup"
- Set the following configs ("Hardware Flow Control": No - is really important!!):
```
    +-----------------------------------------------------------------------+
    | A -    Serial Device      : /dev/ttyUSB0                              |
    | B - Lockfile Location     : /var/lock                                 |
    | C -   Callin Program      :                                           |
    | D -  Callout Program      :                                           |
    | E -    Bps/Par/Bits       : 115200 8N1                                |
    | F - Hardware Flow Control : No                                        |
    | G - Software Flow Control : No                                        |
    | H -     RS485 Enable      : No                                        |
    | I -   RS485 Rts On Send   : No                                        |
    | J -  RS485 Rts After Send : No                                        |
    | K -  RS485 Rx During Tx   : No                                        |
    | L -  RS485 Terminate Bus  : No                                        |
    | M - RS485 Delay Rts Before: 0                                         |
    | N - RS485 Delay Rts After : 0                                         |
    |                                                                       |
    |    Change which setting?                                              |
    +-----------------------------------------------------------------------+
```
- Save configs: Choose "Save setup as dfl"

## Dismantle router (AND VOID WARRANTY!!!!!) ##
- Remove router rubber foot
- Remove the four phillips screws in the bottom of the router:
![image](https://user-images.githubusercontent.com/47162315/219514520-5ad1b067-a631-4aaf-8926-11291493ff59.png)

- Lay router down with the wording at the bottom facing right way up
- Remove the side cover by pulling it (beware not to break the supports)
- Unscrew 3 philips screws and remove PCBs out of back shell (you can take of the antenna plastic stuff to make it easier)
![image](https://user-images.githubusercontent.com/47162315/219515734-51482cfd-da7f-4bbf-920f-25e03f0d1085.png)

- Locate the PCB connections:

![image](https://user-images.githubusercontent.com/47162315/219516002-48374c47-e2ca-44b4-8bd2-b7c404d9be0a.png)
- Connect the FDTI to the router following the scheme (connections are clearly labelled on router PCB):

|Router  | FDTI  |
|--------|----- |
|        |  VCC |
|  GND   |  GND |
|  TX    |  RX  |
|  RX    |  TX  |

- *NOTE: VCC should not be connected

## Connect to router via serial connection ##
- Execute: 
  - `sudo minicom`

## Backup current image: ## 
- Open nc server on destination machine: 
  - `nc -l 4444 > original_firmware.trx`
- Copy current image via nc: 
  - `nc 192.168.10.100 4444  < /dev/mtdblock6`

## Install openwrt: ## 
- Copy openwrt image via tftp: 
  - `tftp -g -r openwrt.trx 192.168.10.100`                                                                                                            
- Override firmware: 
  - `cat openwrt.trx  > /dev/mtdblock6`
- Reboot: 
  - `reboot`
- Set PC back as DHCP
- On a browser open 192.168.1.1 et voilà!
- Installing irqbalance bring some improvements to the router: 
  - https://openwrt.org/docs/guide-user/services/irqbalance

## Troubleshooting ##

### 5Ghz Wifi Signal: ###
If the WIFI signal is bad, you can try to install the firmware "brcmfmac4366c-pcie.bin_ac88_3" present in the following page:
- https://github-com.translate.goog/xiangfeidexiaohuo/Phicomm-K3_Wireless-Firmware?_x_tr_sl=zh-CN&_x_tr_tl=en&_x_tr_hl=pt-PT&_x_tr_pto=wapp

How to replace the driver?
1. Log in to the router with software such as WinSCP
2. Go to the /lib/firmware/brcm/ directory
3. Change brcmfmac4366c-pcie.bin to brcmfmac4366c-pcie.bin.bak
4. Pass the driver you want to replace to the /lib/firmware/brcm/ directory
5. Then rename the replaced driver to brcmfmac4366c-pcie.bin
6. Restart the router

### No WAN interface: ###
After the installation my WAM interface wasn't working (no signal when cable is connected). The workaround found for this was to setup of the LAN ports to act as the WAN port.
To do so:
- Go to "Network -> Interfaces -> WAN -> Edit -> Device" and change "wan" to "Swicth port: lan3"
- Go to "Network -> Interfaces -> WAN6 -> Edit -> Device" and change "wan" to "Swicth port: lan3"
- Go to "Network -> Interfaces -> Devices -> (br-lan) Configure -> Bridge ports" and remove "lan3" (leaving only lan1 and lan2)
- Reboot

------
## References: ##
This tutorial was only possible because of the amazing tips found in the following forum entries:

- https://forum.openwrt.org/t/phicomm-k3-a1-wavlink-quantum-dax-how-to-install-openwrt/131384/9
- https://forum.openwrt.org/t/install-openwrt-on-wavlink-quantum-dax-via-ttl-where-am-i-wrong/137351/3
- https://forum.openwrt.org/t/openwrt-support-for-wavlink-quantum-dax/148323
- https://forum.openwrt.org/t/openwrt-19-07-and-bcm53xx-target-flow-offloading/52356/8
