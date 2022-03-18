- FBTFT-ST7735R on kvim3l over SPI build steps 

NOTE - Purpose of the source code is to provide knowledge base

------------------------------------------------------------------------
step 1: set enviroment parameters (source side)
------------------------------------------------------------------------

-> from fenix direcory
	$ source env/setenv.sh -q -s  KHADAS_BOARD=VIM3L LINUX=4.9 UBOOT=2015.01 DISTRIBUTION=Debian DISTRIB_RELEASE=buster DISTRIB_TYPE=lxde DISTRIB_ARCH=arm64 INSTALL_TYPE=SD-USB COMPRESS_IMAGE=yes
	
------------------------------------------------------------------------
step 2: make kernel debs (source side)
------------------------------------------------------------------------

->	from fenix directory
	$ NO_GIT_UPDATE=1 make kernel 
	$ NO_GIT_UPDATE=1 make kernel-deb
	
-------------------------------------------------------------------------	
step 3: send files over deb files over ssh (source side)
------------------------------------------------------------------------

-> from fenix directory
	$ cd build/images/debs/1.0.7/VIM3L/
	$ scp -r linux-dtb-amlogic-4.9_1.0.7_arm64.deb linux-image-amlogic-4.9_1.0.7_arm64.deb linux-headers-amlogic-4.9_1.0.7_arm64.deb root@<board_ip_address>:/home/khadas
	
------------------------------------------------------------------------
step 4: install deb files (board side)
------------------------------------------------------------------------
	->from /home/khadas where deb file are sent install deb packages
	$ sudo dpkg -i linux-*.deb
	$ sync
	$ sudo reboot	
	
------------------------------------------------------------------------
step 5: display setup steps (board side)
------------------------------------------------------------------------

step 1. include mod for our display st7735 with debug messages:
	
	-> on board go to display module folder
		$ cd /usr/lib/modules/4.9.241/kernel/drivers/staging/fbtft/
	
	-> remove modules if installed before
	$ sudo rmmod fb_st7735r fbtft_device fbtft
	
	(note: in case no module added before output would be like this)
	
		rmmod: ERROR: Module fb_st7735r is not currently loaded
		rmmod: ERROR: Module fbtft_device is not currently loaded
		rmmod: ERROR: Module fbtft is not currently loaded
	)
	
	-> clean dmesg buffer to see only changes after cleaning buffer
	$ dmesg -C
	
	-> modprobe fbtft to enable fbtft driver support
	$ sudo modprobe fbtft dma=0
	
	-> add add device specific driver
	$ sudo insmod fb_st7735r.ko 
	
	-> add generic fbtft device driver. this step also link device tree with our devce specific driver
	$ sudo insmod fbtft_device.ko name=adafruit18 busnum=1 verbose=3 debug=3
	
	-> see the dmesg to see if mod added successfully. 
	$ dmesg 
	
	-> dmesg output look like this
	
		[   55.591478] fbtft: module is from the staging directory, the quality is unknown, you have been warned.
		[   55.623072] fb_st7735r: module is from the staging directory, the quality is unknown, you have been warned.
		[   55.661070] fbtft_device: module is from the staging directory, the quality is unknown, you have been warned.
		[   55.661662] fbtft_device_init-1417
		[   55.661735] meson-fb fb: fb id=-1 pdata? no
		[   55.661791] fbtft_device_init-1463 name=adafruit18, busnum=1, cs=0
		[   55.661796] [FDEBUG]fbtft_device_init-1514 spi->max_speed_hz = 32000000
		[   55.661814] spi spi1.0: setup mode 0, 8 bits/w, 32000000 Hz max --> 0
		[   55.662106] fb_st7735r spi1.0: fbtft_gamma_parse_str() str=
		[   55.662111] fb_st7735r spi1.0: 02 1c 07 12 37 32 29 2d 29 25 2B 39 00 01 03 10
		               03 1d 07 06 2E 2C 29 2D 2E 2E 37 3F 00 00 02 10
		[   55.662150] fb_st7735r spi1.0: fbtft_request_gpios: 'reset' = GPIO461
		[   55.662162] fb_st7735r spi1.0: fbtft_request_gpios: 'dc' = GPIO463
		[   55.662173] fb_st7735r spi1.0: fbtft_request_gpios: 'led' = GPIO460
		[   55.663074] fb_st7735r spi1.0: fbtft_request_gpios: 'cs' = GPIO433
		[   55.663082] fb_st7735r spi1.0: fbtft_verify_gpios()
		[   55.663092] fb_st7735r spi1.0: fbtft_reset()
		[   55.783142] fb_st7735r spi1.0: init: write(0x01) 
		[   55.793800] fb_st7735r spi1.0: init: mdelay(150)
		[   55.944031] fb_st7735r spi1.0: init: write(0x11) 
		[   55.944112] fb_st7735r spi1.0: init: mdelay(500)
		[   56.447773] fb_st7735r spi1.0: init: write(0xB1) 0x01 0x2C 0x2D 
		[   56.448103] fb_st7735r spi1.0: init: write(0xB2) 0x01 0x2C 0x2D 
		[   56.448389] fb_st7735r spi1.0: init: write(0xB3) 0x01 0x2C 0x2D 0x01 0x2C 0x2D 
		[   56.448638] fb_st7735r spi1.0: init: write(0xB4) 0x07 
		[   56.448730] fb_st7735r spi1.0: init: write(0xC0) 0xA2 0x02 0x84 
		[   56.448803] fb_st7735r spi1.0: init: write(0xC1) 0xC5 
		[   56.448861] fb_st7735r spi1.0: init: write(0xC2) 0x0A 0x00 
		[   56.448919] fb_st7735r spi1.0: init: write(0xC3) 0x8A 0x2A 
		[   56.448977] fb_st7735r spi1.0: init: write(0xC4) 0x8A 0xEE 
		[   56.449034] fb_st7735r spi1.0: init: write(0xC5) 0x0E 
		[   56.449090] fb_st7735r spi1.0: init: write(0x20) 
		[   56.449129] fb_st7735r spi1.0: init: write(0x3A) 0x05 
		[   56.449204] fb_st7735r spi1.0: init: write(0x29) 
		[   56.449306] fb_st7735r spi1.0: init: mdelay(100)
		[   56.549323] fb_st7735r spi1.0: init: write(0x13) 
		[   56.549384] fb_st7735r spi1.0: init: mdelay(10)
		[   56.559394] set_var-111
		[   56.559486] fb_st7735r spi1.0: fbtft_update_display(start_line=0, end_line=159)
		[   56.599000] fb_st7735r spi1.0: Display update: 1011 kB/s, fps=0
		[   56.599008] set_gamma-150
		[   56.599570] graphics fb4: fb_st7735r frame buffer, 128x160, 40 KiB video memory, 4 KiB buffer memory, fps=20, spi1.0 at 32 MHz
		[   56.599576] fb_st7735r spi1.0: fbtft_backlight_update_status: polarity=1, power=0, fb_blank=0
		[   56.599615] spi_add_device-557
		[   56.599621] meson-spicc ffd15000.spi: registered child spi1.0
		[   56.599626] fbtft_device: GPIOS used by 'adafruit18':
		[   56.599630] fbtft_device: 'reset' = GPIO461
		[   56.599633] fbtft_device: 'dc' = GPIO463
		[   56.599636] fbtft_device: 'led' = GPIO460
		[   56.599639] fbtft_device: 'cs' = GPIO433
		[   56.599642] fbtft_device_init-1576
		[   56.599648] fb_st7735r spi1.0: fb_st7735r spi1.0 32000kHz 8 bits mode=0x00
		[   56.599652] fbtft_device: fbtft_device_init probe done
	
	note: on display side it should be changed to dark-grey display
	
------------------------------------------------------------------------	
step 6: start display with serverX:
------------------------------------------------------------------------

->	use newly created fb node number in below commad, replace x with newly created node.
	e.g  for /dev/fb4 node, replace x with 4)
		
		$ sudo FRAMEBUFFER=/dev/fbx startx

->  now wait for some time for display to turn on it should take near 10 seconds.
->  you should be able to control curser via connecting mouse to any of vim3l boards usb poart.
