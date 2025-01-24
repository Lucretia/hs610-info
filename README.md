# HS610

![HUION HS610](./imgs/hs610.png)

I have this tablet to learn to draw with, but it is currently unusable as the drivers do not work. So, in the process of learning what I need to re Linux USB HID drivers, this repo contains the information I have discovered about this device.

## USB ID on Linux

```bash
$ lsusb|grep -i huion
Bus 001 Device 065: ID 256c:006d HUION Huion Tablet_HS610
```

If you have a device with the same VID:PID as above, can you open an issue here with the details of the device?

## Flashing the firmware on Windows

On flashing the device under a Windows VM, you are prompted to press buttons 1 and 5 on the device as you plug it in, this puts the device into a new mode for flashing.

I flashed to the latest version, it all seemed a bit weird the way it worked, but the following is the version I have, I don't know if it's the same as it was before, as according to the HUION website.

![Firmware Version](./imgs/screenshots/firmware.png)

### USB ID's

This prompted me to do the same on Linux and see what lsusb produced, interestingly, it gave the following:

```bash
$ lsusb
Bus 001 Device 041: ID 28e9:0189 GDMicroelectronics GD32 DFU Bootloader (Longan Nano)
```

The output of dmesg shows this on adding the device in flash mode:

```bash
[  +4.934932] usb 1-4.4: USB disconnect, device number 45
[  +4.517608] usb 1-4.4: new full-speed USB device number 46 using ehci-pci
[  +0.292254] usb 1-4.4: New USB device found, idVendor=28e9, idProduct=0189, bcdDevice= 1.00
[  +0.000006] usb 1-4.4: New USB device strings: Mfr=1, Product=2, SerialNumber=3
[  +0.000002] usb 1-4.4: Product: GD32 USB DFU in FS Mode
[  +0.000002] usb 1-4.4: Manufacturer: GigaDevice
[  +0.000001] usb 1-4.4: SerialNumber: 䌵䜸
```

If you cannot read the SerialNumber, that's because they are CJK characters, I don't know what that translates to, Google was useless here.

I don't know what it means by FS mode, unless it means the extensions added by ST Micro-Electronics in DfuSe?

```bash
# lsusb -v -d 28e9:0189

Bus 001 Device 048: ID 28e9:0189 GDMicroelectronics GD32 DFU Bootloader (Longan Nano)
Device Descriptor:
  bLength                18
  bDescriptorType         1
  bcdUSB               2.00
  bDeviceClass            0
  bDeviceSubClass         0
  bDeviceProtocol         0
  bMaxPacketSize0        64
  idVendor           0x28e9 GDMicroelectronics
  idProduct          0x0189 GD32 DFU Bootloader (Longan Nano)
  bcdDevice            1.00
  iManufacturer           1 GigaDevice
  iProduct                2 GD32 USB DFU in FS Mode
  iSerial                 3 䌵䜸
  bNumConfigurations      1
  Configuration Descriptor:
    bLength                 9
    bDescriptorType         2
    wTotalLength       0x001b
    bNumInterfaces          1
    bConfigurationValue     1
    iConfiguration          0
    bmAttributes         0x80
      (Bus Powered)
    MaxPower              100mA
    Interface Descriptor:
      bLength                 9
      bDescriptorType         4
      bInterfaceNumber        0
      bAlternateSetting       0
      bNumEndpoints           0
      bInterfaceClass       254 Application Specific Interface
      bInterfaceSubClass      1 Device Firmware Update
      bInterfaceProtocol      2
      iInterface              0
      Device Firmware Upgrade Interface Descriptor:
        bLength                             9
        bDescriptorType                    33
        bmAttributes                       11
          Will Detach
          Manifestation Intolerant
          Upload Supported
          Download Supported
        wDetachTimeout                    255 milliseconds
        wTransferSize                    2048 bytes
        bcdDFUVersion                   1.1a
can't get device qualifier: Resource temporarily unavailable
can't get debug descriptor: Resource temporarily unavailable
Device Status:     0x0002
  (Bus Powered)
  Remote Wakeup Enabled
```

The USB descriptor above shows the DFU version to be 1.1a, which matches that of the DfuSe, as there is no USB DFU 1.1a document that I can find.

### DFU-Util

As I now know the chip in use here, this sent me looking for information on it, there are many by this company and I originally thought it was a RISC-V, as the dfu-util from [gd32-utils](https://github.com/riscv-mcu/gd32-dfu-utils) shows:

```bash
$ ~/opt/gd32-dfu-utils/bin/dfu-util -l
dfu-util 0.9

Copyright 2005-2009 Weston Schmidt, Harald Welte and OpenMoko Inc.
Copyright 2010-2016 Tormod Volden and Stefan Schmidt
This program is Free Software and has ABSOLUTELY NO WARRANTY
Please report bugs to http://sourceforge.net/p/dfu-util/tickets/

Found Runtime: [0a5c:21e8] ver=0112, devnum=18, cfg=1, intf=3, path="1-2.3", alt=0, name="UNKNOWN", serial="5CF37093A853"
Found DFU: [28e9:0189] ver=0100, devnum=43, cfg=1, intf=0, path="1-4.4", alt=0, name="UNKNOWN", serial="5C8GNGN"
Found Runtime: [1235:800a] ver=0125, devnum=2, cfg=1, intf=5, path="8-1", alt=0, name="Scarlett 2i4 USB-DFU", serial="UNKNOWN"
Found Runtime: [1235:800a] ver=0125, devnum=2, cfg=1, intf=5, path="8-1", alt=0, name="Scarlett 2i4 USB-DFU", serial="UNKNOWN"
```

The other two devices were also interesting to find, the last two devices are the same just listed twice, again for some unknown reason.

There are multiple versions of this tool, not one of them can download the firmware from the device as can be seen here:

```bash
 ~/opt/gd32-dfu-utils/bin/dfu-util -a 0 -v -U fw.dfu -d 256c:006d
dfu-util 0.9

Copyright 2005-2009 Weston Schmidt, Harald Welte and OpenMoko Inc.
Copyright 2010-2016 Tormod Volden and Stefan Schmidt
This program is Free Software and has ABSOLUTELY NO WARRANTY
Please report bugs to http://sourceforge.net/p/dfu-util/tickets/

Opening DFU capable USB device...
ID 28e9:0189
Run-time device DFU version 011a
Claiming USB DFU Interface...
Setting Alternate Setting #0 ...
dfu-util: Cannot set alternate interface
```

## GD32 DFU Tool

This led to a bit of searching and I managed to grab the tool shown [here](https://github.com/riscv-mcu/gd32-dfu-utils/issues/2) and run it on Windows:

![GD32 DFU Tool - Main window](./imgs/screenshots/gd32_dfu_tool.png)
![GD32 DFU Tool - Option bytes window](./imgs/screenshots/gd32_dfu_tool_option_bytes.png)
![GD32 DFU Tool - Flash pages window 1](./imgs/screenshots/gd32_dfu_tool_flash_pages_0_30.png)
![GD32 DFU Tool - Flash pages window 2](./imgs/screenshots/gd32_dfu_tool_flash_pages_30_60.png)
![GD32 DFU Tool - Flash pages window 3](./imgs/screenshots/gd32_dfu_tool_flash_pages_34_63.png)

As can be see by the first window, gives us the MCU part number, [GD32F350C8T6](https://www.gigadevice.com/microcontroller/gd32f350c8t6/)

* [DataSheet](./docs/GD32F350xx_Datasheet_Rev1.4.pdf)
* [User Manual](./docs/GD32F3x0_User_Manual_EN_v2.1.pdf)
* [Firmware Library User Guide](./docs/GD32F3x0_Firmware_Library_User_Guide_Rev1.0.pdf)
* [GD32 DFU Tool and DFU Drivers](http://www.gd32mcu.com/en/download?kw=dfu&lan=en)

## Finding the firmware

I have sniffed the http/s traffic from the firmware application and got a number of requests sent:

```bash
POST http://zyz.huion.cn/api/device.php HTTP/1.1
nReferer: zyz.huion.cn
Content-Type: application/x-www-form-urlencoded
Accept: */*
User-Agent: HuionFirmware
Host: zyz.huion.cn
Content-Length: 6
Pragma: no-cache

id=all
```

```bash
GET http://zyz.huion.cn/api/search.php?token= HTTP/1.1
nReferer: zyz.huion.cn
Content-Type: application/x-www-form-urlencoded
Accept: */*
User-Agent: HuionFirmware
Host: zyz.huion.cn
Pragma: no-cache
```

```bash
GET http://wx.huion.cn/api/Propelling/getInfo/?region=GB&sys_type=win HTTP/1.1
nReferer: wx.huion.cn
Content-Type: application/x-www-form-urlencoded
Accept: */*
Host: wx.huion.cn
Pragma: no-cache
Cookie: FIFMWARE=8001e659212cd459f063ea7691e0dae7
```

By sending the following command:

```bash
$ wget --no-cookies --header "Cookie: FIFMWARE=8001e659212cd459f063ea7691e0dae7" http://zyz.huion.cn/api/search.php?token=
```

I was able to get a list of firmware binaries with url's for various devices. I have downloaded that file and the gd32-prefix/suffix tools don't recognise the files:

```bash
$ ~/opt/gd32-dfu-utils/bin/dfu-prefix -c c831532b0621e8ff9510fdc18ebe00d1.bin
dfu-prefix (dfu-util) 0.9

Copyright 2011-2012 Stefan Schmidt, 2014 Uwe Bonnes
This program is Free Software and has ABSOLUTELY NO WARRANTY
Please report bugs to http://sourceforge.net/p/dfu-util/tickets/

dfu-prefix: Invalid DFU suffix signature
dfu-prefix: A valid DFU suffix will be required in a future dfu-util release!!!

$ ~/opt/gd32-dfu-utils/bin/dfu-suffix -c c831532b0621e8ff9510fdc18ebe00d1.bin
dfu-suffix (dfu-util) 0.9

Copyright 2011-2012 Stefan Schmidt, 2013-2014 Tormod Volden
This program is Free Software and has ABSOLUTELY NO WARRANTY
Please report bugs to http://sourceforge.net/p/dfu-util/tickets/

dfu-suffix: Invalid DFU suffix signature
dfu-suffix: Valid DFU suffix needed
```

So, it could be an encrypted file, or they have a modified toolset for their devices.

### MD5

The JSON file returned from the query above has a 30-digit md5sum hash contained, it misses off the last 2 digits of the hash, i.e.:

```bash
$ md5sum HS610_HUION_T194_190307.bin
b552a20d0dab09dcf1dbfafea3ab4b21  HS610_HUION_T194_190307.bin
b552a20d0dab09dcf1dbfafea3ab4b    # Hash from the JSON file for this binary.

$ md5sum HS610_HUION_T194_191023.bin
9257fe18bf62426a6c5a412cc74e2bd0  HS610_HUION_T194_191023.bin
9257fe18bf62426a6c5a412cc74e2b    # Hash from the JSON file for this binary.
```

## Libinput data

```bash
$ sudo libinput list-devices
event4  - HUION Huion Tablet_HS610 Touch Ring: libinput bug: missing tablet capabilities: pen btn-stylus resolution. Ignoring this device.
event5  - HUION Huion Tablet_HS610 Dial: libinput bug: missing tablet capabilities: pen btn-stylus resolution. Ignoring this device.
event1  - HUION Huion Tablet_HS610 Keyboard: libinput bug: missing tablet capabilities: xy pen btn-stylus resolution. Ignoring this device.

# Non-related devices removed.

Device:           HUION Huion Tablet_HS610
Kernel:           /dev/input/event2
Group:            7
Seat:             seat0, default
Size:             254x159mm
Capabilities:     tablet
Tap-to-click:     n/a
Tap-and-drag:     n/a
Tap drag lock:    n/a
Left-handed:      n/a
Nat.scrolling:    n/a
Middle emulation: n/a
Calibration:      n/a
Scroll methods:   none
Click methods:    none
Disable-w-typing: n/a
Accel profiles:   none
Rotation:         n/a

Device:           HUION Huion Tablet_HS610 Pad
Kernel:           /dev/input/event3
Group:            7
Seat:             seat0, default
Capabilities:     tablet-pad
Tap-to-click:     n/a
Tap-and-drag:     n/a
Tap drag lock:    n/a
Left-handed:      n/a
Nat.scrolling:    n/a
Middle emulation: n/a
Calibration:      n/a
Scroll methods:   none
Click methods:    none
Disable-w-typing: n/a
Accel profiles:   n/a
Rotation:         n/a
Pad:
	Rings:   0
	Strips:  0
	Buttons: 13
	Mode groups: 1 (1 modes)
```

## Notes

So, what's the point of all this? Well, it would be nice to be able to update firmware without a Windows or Mac machine. Yes I can do it in a virtual machine (KVM), and it works, but some people don't and won't.

The issue is this, the firmware isn't distributed with the firmware tool, it's downloaded and from a (now) known location. It would be nice to be able to have this location publicly available or even just a link to the DFU binary.

The manufacturer also doesn't want people to reverse engineer their firmware, well, unfortunately, that is beyond their control, people will do it, 1) because they can, 2) because it's a challenge and 3) because at some point the tablet will be unsupported and people will still own one and there may be bugs that need fixing.

Any help in writing the firmware with open tools to the device will be gratefully received and merged into this repository.

## Useful Links

* [Touch ring PID only](https://github.com/DIGImend/digimend-kernel-drivers/issues/275#issuecomment-691860908)
* [HS610 touch ring user-level driver](https://github.com/leiserfg/w2w)

## Contributing

If anyone does this, I'll need to change the name of the repository.

### Your own tablet, other than any already listed

If you want to see if your tablet enters a new mode for flashing, do the following:

1. ```dmesg -Hw``` in a terminal.
2. Check the site for firmware instructions, if any.
3. Hold down the buttons one at a time and insert the USB cable, try combinations of buttons.
4. Note down what dmesg prints if it prints anything.
5. Create an issue here with those details and I'll create a new page for that tablet.
6. Find links to the datasheets I can link into a new document.
7. Trace networking/USB on Windows or Mac using a tool as you insert the cable into the tablet in case the driver is checking for firmware updates.
8. Note down any URL's grabbed.
9. Test these URL's with wget, see above, and see what comes down, add these files to the issue.

I'm interested in anything you can find relating to flashing the ROM or reading from the ROM on the MCU inside the tablet, datasheets of the MCU, etc.

## Links

* [FrOSCon DIGImend presentation](https://youtu.be/Qi73_QFSlpo)
*