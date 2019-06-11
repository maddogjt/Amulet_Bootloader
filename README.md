# Amulet of Rune nRF52 Bootloader

[![Build Status](https://travis-ci.com/maddogjt/Amulet_Bootloader.svg?branch=master)](https://travis-ci.com/maddogjt/Amulet_Bootloader)

This is a CDC/DFU/UF2 bootloader for nRF52 boards.

- [Adafruit Feather nRF52832](https://www.adafruit.com/product/3406)
- Amulet Proto Revision
- Amulet EVT Revision
- Amulet MP Revision

UF2 is an easy-to-use bootloader that appears as a flash drive. You can just copy `.uf2`-format
application images to the flash drive to load new firmware.
See https://github.com/Microsoft/uf2 and https://github.com/adafruit/uf2-samdx1
for more information.

[adafruit-nrfutil](https://github.com/adafruit/Adafruit_nRF52_nrfutil),
a modified version of [Nordic nrfutil](https://github.com/NordicSemiconductor/pc-nrfutil),
is required to perform DFU.
Install `python3` if it is not installed already and run this command to install adafruit-nrfutil from PyPi:

```
$ pip3 install --user adafruit-nrfutil
```

This repository depends on the following submodules:

- [tinyusb](https://github.com/hathach/tinyusb)
- [nrfx](https://github.com/NordicSemiconductor/nrfx)

Note that `tinyusb` also includes `nrfx` as a submodule, so you need
to initialize and update  submodules with the `--recursive`` flag.

Clone this repo with following commands, or fork it for your own development

```
git clone https://github.com/maddogjt/Amulet_Bootloader
cd Amulet_Bootloader
git submodule update --init --recursive
```

## Features

- DFU over Serial and OTA ( application, Bootloader+SD )
- Self-upgradable via Serial and OTA
- DFU using UF2 (https://github.com/Microsoft/uf2) (application only)
- Auto-enter DFU briefly on startup for DTR auto-reset trick (832 only)

## How to use

There are two pins, `DFU` and `FRST` that bootloader will check upon reset/power:

- `Double Reset` Reset twice within 500 ms will enter DFU with UF2 and CDC support (only works with nRF52840)
- `DFU = LOW` and `FRST = HIGH`: Enter bootloader with UF2 and CDC support
- `DFU = LOW` and `FRST = LOW`: Enter bootloader with OTA, to upgrade with a mobile application such as Nordic nrfConnect/Toolbox
- `DFU = HIGH and `FRST = LOW`: Factory Reset mode: erase firmware application and its data
- `DFU = HIGH and `FRST = HIGH`: Go to application code if it is present, otherwise enter DFU with UF2
- The `GPREGRET` register can also be set to force the bootloader can enter any of above modes (plus a CDC-only mode for Arduino).
`GPREGRET` is set by the application before performing a soft reset.

On Amulet boards, `DFU` is assigned to the labeled `DFU` button.  `FRST` is assigned to Pin6, which is broken out on Proto0 and is the front unpopulated button on EVT/MP

For other boards, please check the board definition for details.

## Burn & Upgrade with pre-built binaries

You can burn and/or upgrade the bootloader with either a J-link or DFU (serial) to a specific pre-built binary version
without the hassle of installing a toolchain and compiling the code.
This is preferred if you are not developing/customizing the bootloader.
Pre-builtin binaries are available on GitHub [releases](https://github.com/maddogjt/Amulet_Bootloader/releases)

Note: The bootloader can be downgraded. Since the binary release is a merged version of
of both bootloader and the Nordic SoftDevice, you can freely upgrade/downgrade to any version you like.

## How to compile and build

You should only continue if you are looking to develop bootloader for your own.
You must have have  a J-Link available to "unbrick" your device.

### Option 1: Build with Makefile

Prerequisites

- ARM GCC
- Nordic's [nRF5x Command Line Tools](http://infocenter.nordicsemi.com/index.jsp?topic=%2Fcom.nordic.infocenter.tools%2Fdita%2Ftools%2Fnrf5x_command_line_tools%2Fnrf5x_installation.html)

To build:

```
make BOARD=amulet_evt all combinehex
```

To flash the bootloader with JLink:

```
make BOARD=amulet_evt flash
```

To upgrade the bootloader using DFU Serial via port /dev/ttyACM0

```
make BOARD=amulet_evt SERIAL=/dev/ttyACM0 dfu-flash
```

To flash SoftDevice (and chip erase):

```
make BOARD=amulet_evt sd
```

To erase all of flash:

```
make BOARD=amulet_evt erase
```

For the list of supported boards, run `make` without `BOARD=` :

```
$ make
You must provide a BOARD parameter with 'BOARD='
Supported boards are: amulet_evt amulet_mp amulet_proto0 feather_nrf52832
Makefile:90: *** BOARD not defined.  Stop
```

### Common makefile problems

#### 1. `arm-none-eabi-gcc`: No such file or directory

If you get the following error ...

```
$ make BOARD=amulet_evt all 
Compiling file: main.c
/bin/sh: /usr/bin/arm-none-eabi-gcc: No such file or directory
make: *** [_build/main.o] Error 127
```

... you may need to edit the `Makefile` and update `GNU_INSTALL_ROOT` to point to the root path of your GCC ARM toolchain.

#### 2. `mergehex: No such file or directory`

Make sure that `mergehex` is available from the command-line. This binary is
part of Nordic's nRF5x Command Line Tools.

#### 3. `make: nrfjprog: No such file or directory`

Make sure that `nrfjprog` is available from the command-line. This binary is
part of Nordic's nRF5x Command Line Tools.

On POSIX-type systems you can temporarily add the path to `nrfjprog` via a
variation on the following command:

```
$ export PATH=$PATH:/location/of/nRF5x-Command-Line-Tools_9_7_2_OSX/nrfjprog
```

### Option 2: Build using Segger Embedded Studio

For easier debugging you can also use [SES](https://www.segger.com/products/development-tools/embedded-studio/).
The project file is located at `src/segger/Adafruit_nRF52_Bootloader.emProject`.

> **Note**: SES only flashes the bootloader when you click download, not the SoftDevice.
You need to flash the SoftDevice beforehand if you haven't already done so.
As mentioned above do something like:

```
make BOARD=amulet_evt sd
```
