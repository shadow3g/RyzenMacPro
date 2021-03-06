# Ryzen Mac Pro - OpenCore EFI for ASRock X570 ITX



This repository provides the basic EFI folder to run macOS Catalina on an ASRock Phantom Gaming ITX/TB3 motherboard. The default provided currently using a Ryzen 9 3900X 12 Core CPU and a Radeon RX 5500 XT. For a short guide to using different CPUs and GPUs see below (all kexts specific to those are named explicitely).
This is intended as a reference and to share improvements for similar build, not as an out of the box EFI to download. It is highly recommended to start with a vanilla OpenCore and following [OpenCore Vanilla Guide](https://khronokernel-2.gitbook.io/opencore-vanilla-desktop-guide/) first.

## Ryzen Mac Pro build

**Prozessor:** AMD Ryzen 9 3900X  
**Mainboard:** AsRock X570 Phantom Gaming ITX/TB3 (BIOS 2.0)  
**Memory:** Corsair Vengeance RGB Pro (2x, 16GB) DDR4-3200  
**Storage:** Corsair MP600 (1000GB) M.2 NVMe PCIe 4.0  
**Video Card:** Sapphire Radeon RX 5500 XT Pulse 8G  
**Power Supply:** Corsair SF600 Platinum  
**Case:** Phanteks Enthoo Evolv Shift (Mini-ITX)

## Versions
**OpenCore:** 0.5.7   
**MacOS:** 10.15.4  

## Content

### ACPI

The following SSDT files are for setting up HPET and EC.

- SSDT-EC.aml
- SSDT-HPET.aml
- SSDT-PLUG.aml
- SSDT-SBRG.aml

The following SSDT files are for USB Power and properly name USB controllers and ports. Note: to fix sleep issues the internal HS10 port connected to bluetooth is **not** configured as internal. Currently also the internal USB 2 header is not mapped (the SSDT-XHC-full.aml contains all even unconnected ports to do a custom mapping).

- SSDT-USBX.aml
- SSDT-XHC.aml
- SSDT-XHC-XHC0.aml (either this or above - see section about sleep)

### Kexts

Besides the default kexts the following are noteworthy:

For enabling the integrated Intel Bluetooth

- IntelBluetoothFirmware.kext
- IntelBluetoothInjector.kext

The SMCAMDProcessor.kext is used to provide CPU temperature and frequency information, the AMD Power Gadget can be downloaded from https://github.com/trulyspinach/SMCAMDProcessor/releases. Other monitoring tools can also access and display this information.

The VoodooTSCSyncAMD kext is used to sync the cores and required the correct number of threads (cores * 2). Either update the Info.plist of the kext or download the generator from [insanelymac](https://www.insanelymac.com/forum/files/file/744-voodootscsync-configurator/).

The RadeonBoost kext is used to inject proper power management (AGPMInjector) and fixes some performance issues. It support SMBIOS iMacPro1,1/all MacPro and RX 480, 580, 590, 5500 (XT), 5600 (XT), 5700 (XT) and Radeon VII. See  [RadeonBoost](https://www.hackintosh-forum.de/forum/thread/47791-radeonboost-kext-benchmark-scores-wie-am-echten-mac-unter-windows/?pageNo=1) for more detailed informations and latest versions.

The AMD-USB-Map kext is depending on the SMBIOS and can be created with the  [Hackintool](https://www.hackintosh-forum.de/forum/thread/38316-hackintool-ehemals-intel-fb-patcher/).
See section about sleep for the other variant.



## Setup

### BIOS settings

Everything is tested with ASRocks latest BIOS v2.0:

- CSM: disabled
- Above 4G decoding: disabled (must be enabled for certain older graphics card)
- Thunderbolt: enabled
  - Security Level: No Security
- Fast boot: disabled

## USB port mapping
![Back I/O](./back_io.png)
The front USB ports on the internal USB 3 header are SS5/HS5 and SS6/HS6.
The port of the internal USB header is mapped to HS9, the internal Bluetooth module to HS10.
In case the XHC0 controller is disabled, the ports 3/4 on the back I/O are USB 2 only.


| XHC0 -> XHCI | | |
| --- | --- | --- |
| PRT1 | HS4 | USB 2 |
| PRT4 | HS9 | internal USB 2 |
| PRT5 | HS2 | USB 2 |
| PRT6 | HS1 | USB 2 |
| PRT9 | SS1 | USB 2 |
| PRT10 | SS2 | USB 2 |

| XHC1 -> XHC | | |
| --- | --- | --- |
| PRT1 | HS6 | USB 2 |
| PRT2 | HS10 | Bluetooth |
| PRT3 | HS5 | USB 2 |
| PRT4 | SS10 | USB Type C |
| PRT7 | SS6 | USB 3 |
| PRT8 | SS5 | USB 3 |

| XHC0 -> XHC2 | | |
| --- | --- | --- |
| PRT7 | SS3 | USB 3 |
| PRT8 | SS4 | USB 3 |

_(The last is the problematic controller causing wake up issues)_



## Known issues

Thunderbolt controller is not detected by MacOS unless device is already connected during boot. Hot plug does not work and changing devices requires a reboot.

The integrated Intel Wifi is not yet supported.

Microphone is not yet working through integrated audio codec.

### Sleep

Sleep can be a difficult topic with little things breaking either entering or leaving sleep. A major source issues for waking up (causing a reboot) is the XHC0 USB Controller, with it disabled and the hibernate fixup kext waking up from sleep seems to work so far.

Lucky us only two USB 3 ports are connected to this controller (the outer ones on the I/O plane) and will be reduced to USB 2 speed. 
To do this replace

- SSDT-XHC.aml -> SSDT-XHC-XHC0.aml
- AMD-USB-Map-MacPro7,1.kext -> AMD-USB-Map-XHC0-MacPro7,1.kext

And make sure boot flag -hbfx-disable-patch-pci is set.

If it does not entering sleep properly there are some things to be tried:

- Disable "Allow Bluetooth devices to wake this computer" in Advanced Bluetooth settings (may require the use of the power button if keyboard and mouse are connected through standard bluetooth)
- Disconnect specific bluetooth devices (like monitor speakers and the like)
- Turn off monitor before entering sleep


### Bluetooth

The bluetooth firmware may fail to load and disable Bluetooth completely. This case can usually be fixed with a reboot.
With deep sleep in S5 enabled in the BIOS this seems to happen regurarely, disabling may help reducing this issue. Reseting the NVRAM also leads to this issue but manually setting the the value in the NVRAM section of the `config.plist` will solve this:

```
<key>bluetoothActiveControllerInfo</key>
<data></data>
```
You can extract the proper data value with the Hackintool.

## Credits

Many thanks to all the help from AMD-OS X and the german Hackintosh Forums.
