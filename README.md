Open firmware for Waveshare 7" capacitive touchscreen
=====================================================

An open firmware of the Waveshare 7" Rev 2.1 capacitive touch display. The original firmware wasn't really HID compliant, so I decided to roll my own as Waveshare didn't reply to my support requests.
Most notably the touchscreen would not respond correctly when used with Windows 10 Anniversary Update and later because of some change in behavior of the USB HID driver (discarding malformed HID reports).

Hardware versions & other vendors
---------------------------------

Since I published this firmware and blogged about it, a few people around the world tried to flash the firmware on diffenrent screen models with similar hardware; here are a screens that are known to work so far:

- Waveshare 7" capacitive touch screen model (B / 800x600 pixel / rev 2.1)
- Waveshare 7" capacitive touch screen model (C / 1024x600 pixel / rev 1.1 and rev 2.1)
- Eleduino 7" HDMI capacitive touch screen (1024x600 pixel / version 150830)

The hardware 
------------

(only things relevant for the project)

- GigaDevice GD32F103 mcu
- Goodix GT811 touchscreen controller
	- I2C port connected to I2C2 on GD32F103
	- I2C_SCL and I2C_SDA tied high with 10k SMT resistors
	- /INT pin connected to Port B / Pin 6
	- /RST tied high with 10k resistor
- SWDP port accessible on PCB
- 8 MHz quartz (positive point for stable USB)
- USB correctly terminated on D+/D- lines
	- no CPU controllable Pull-Up/Pull-Down, so no software controlled re-attach

Prerequisites
-------------

- probably a Linux box (unless you want to configure compiler/linker scripts on your own in your favourite IDE)
- GCC / ARM toolchain (arm-none-eabi-gcc (15:4.9.3+svn231177-1) 4.9.3 20150529 (prerelease))
- SWDP/JTAG for STM32 / GD32
	- StLink v2
	- or BlackMagic probe
	- or OpenOCD + adapter

Instructions
------------

WARNING: there is no cheap was of backing up the original firmware from Waveshare so it'll be gone when you flash this firmware. I can and will not take any responsibility for a bricked screen, you must know what you are doing starting from here!! To say it clearly once again: I'm not liable for any damage to your computer system or touch screen!

The first thing you will have to do, is soldering some connction wires to the SWDP "header" (not really, they are needle contact surfaces for in production flashing). When you look at the screens PCB (so that you can read the silkscreen) the contacts are at the bottom right corner, close to the GD32F103, from top to bottom:

	- SWDIO
	- SWCLK
	- GND

I soldered three DuPont wires directly on the PCB and attached my BlackMagic Probe to them.

Next get the sources and compile everything:

	git clone https://github.com/pysco68/waveshare-hid.git
	cd waveshare-hid/libopencm3
	make 	# compile the libopencm3/GD32 fork
	cd ../src
	make	# compile the new touchscreen firmware

NOTE: I'm using a BlackMagic Probe and GDB in the following to flash the firmware

	arm-none-eabi-gdb main.elf

NOTE: when flashing the display for the first time you must reset the "readout protection bit". Using a Black Magic Probe it can be done by issuing a `monitor option erase` command in GDB once attached to the target (bascially after `attach 1` and before `load` in the listing below).

In the GDB console you can do the following sequence then:

	(gdb) target extended-remote /dev/ttyACM1
	Remote debugging using /dev/ttyACM1
	(gdb) monitor swdp_scan
	Target voltage: unknown
	Available Targets:
	No. Att Driver
	 1      STM32F1 medium density
	(gdb) attach 1
	Attaching to program: /...../main.elf, Remote target
	st_usbfs_poll (dev=0x2000014c <st_usbfs_dev>) at ../common/st_usbfs_core.c:218
	218	{
	(gdb) load
	Loading section .text, size 0x1cfc lma 0x8000000
	Loading section .data, size 0x8c lma 0x8001cfc
	Start address 0x80014cc, load size 7560
	Transfer rate: 7 KB/sec, 840 bytes/write.
	(gdb) run
	The program being debugged has been started already.
	Start it from the beginning? (y or n) y
	Starting program: /..../main.elf 

Done! 

Note: eventually you'll have to plug in / out the screen once to get it properly enumerated.

Status
------

What works:
- Five point multitouch
- working under Ubuntu 16.10
- working under Raspbian Jessie
- working under Windows 10 Anniversary Update (didn't really work with original firmware)
- **working under Windows 10 IoT Anniversary Update on RaspberryPi (didn't work with original firmware either)**
- working under Android 4.2.2

~~What doesn't work (just yet hopefully)~~
~~I'm in touch with Microsoft regarding Windows 10 IoT, hopefully I'll get that working soon!~~
