# Amulet of Rune nRF52 Bootloader

[![Build Status](https://github.com/maddogjt/Amulet_Bootloader/workflows/Build/badge.svg)](https://github.com/maddogjt/Amulet_Bootloader/actions)

This is a CDC/DFU/UF2 bootloader for nRF52 boards.

- [Adafruit Feather nRF52832](https://www.adafruit.com/product/3406)
- Amulet MP Revision

UF2 is an easy-to-use bootloader that appears as a flash drive. You can just copy `.uf2`-format
application images to the flash drive to load new firmware. See https://github.com/Microsoft/uf2 and https://github.com/adafruit/uf2-samdx1 for more information.

[adafruit-nrfutil](https://github.com/adafruit/Adafruit_nRF52_nrfutil), a modified version of [Nordic nrfutil](https://github.com/NordicSemiconductor/pc-nrfutil), is required to perform DFU. Install `python3` if it is not installed already and run this command to install adafruit-nrfutil from PyPi:

```
$ pip3 install --user adafruit-nrfutil
```

Clone this repo with following commands, or fork it for your own development

```
git clone https://github.com/maddogjt/Amulet_Bootloader
cd Amulet_Bootloader
git submodule update --init
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
- <s>`DFU = HIGH` and `FRST = LOW`: Factory Reset mode: erase firmware application and its data</s>
- `DFU = HIGH` and `FRST = HIGH`: Go to application code if it is present, otherwise enter DFU with UF2
- The `GPREGRET` register can also be set to force the bootloader can enter any of above modes (plus a CDC-only mode for Arduino).
`GPREGRET` is set by the application before performing a soft reset.

```c
#include "nrf_nvic.h"
void reset_to_uf2(void) {
  NRF_POWER->GPREGRET = 0x57; // 0xA8 OTA, 0x4e Serial
  NVIC_SystemReset();         // or sd_nvic_SystemReset();
}
```
On Amulet boards, `DFU` is assigned to the labeled `DFU` button.  `FRST` is assigned to Pin6, which is broken out on Proto0 and is the front unpopulated button on EVT/MP

On the Nordic PCA10056 DK board, `DFU` is connected to **Button1**, and `FRST` is connected to **Button2**.
So holding down **Button1** while clicking **RESET** will put the board into USB bootloader mode, with UF2 and CDC support.
Holding down **Button2** while clicking **RESET** will put the board into OTA (over-the-air) bootloader mode.

On the Nordic PCA10059 Dongle board, `DFU` is connected to the white button.
`FRST` is connected to pin 1.10. Ground it to pull `FRST` low, as if you had pushed an `FRST`  button.
There is an adjacent ground pad.

For other boards, please check the board definition for details.

### Making your own UF2

To create your own UF2 DFU update image, simply use the [Python conversion script](https://github.com/Microsoft/uf2/blob/master/utils/uf2conv.py) on a .bin file or .hex file, specifying the family as **0xADA52840**. If using a .bin file with the conversion script you must specify application address with the -b switch, this address depend on the SoftDevice size/version e.g S140 v6 is 0x26000

To create a UF2 image from a .bin file:
```
uf2conv.py firmware.bin -c -b 0x26000 -f 0xADA52840
```

To create a UF2 image from a .hex file:
```
uf2conv.py firmware.hex -c -f 0xADA52840
```

To create a UF2 image for bootloader from a .hex file using separated family of **0xd663823c**

```
uf2conv.py bootloader.hex -c -f 0xd663823c
```

## Burn & Upgrade with pre-built binaries

You can burn and/or upgrade the bootloader with either a J-link or DFU (serial) to a specific pre-built binary version
without the hassle of installing a toolchain and compiling the code.
This is preferred if you are not developing/customizing the bootloader.
Pre-builtin binaries are available on GitHub [releases](https://github.com/maddogjt/Amulet_Bootloader/releases)

Note: The bootloader can be downgraded. Since the binary release is a merged version of
both bootloader and the Nordic SoftDevice, you can freely upgrade/downgrade to any version you like.

## How to compile and build

You should only continue if you are looking to develop bootloader for your own.
You must have have a J-Link available to "unbrick" your device.

Prerequisites

- ARM GCC

To install for macos

```bash
brew tap ArmMbed/homebrew-formulae
brew install arm-none-eabi-gcc
brew link --overwrite arm-none-eabi-gcc # if a prior version was present
```

- Nordic's [nRF5x Command Line Tools](https://www.nordicsemi.com/Software-and-Tools/Development-Tools/nRF-Command-Line-Tools)
- [Python IntelHex](https://pypi.org/project/IntelHex/)

To build:

```
make BOARD=amulet_mp all
```

To flash the bootloader with JLink:

```
make BOARD=amulet_mp flash
```

To upgrade the bootloader using DFU Serial via port /dev/ttyACM0

```
make BOARD=amulet_mp SERIAL=/dev/ttyACM0 dfu-flash
```

To flash SoftDevice (and chip erase):

```
make BOARD=amulet_mp sd
```

For the list of supported boards, run `make` without `BOARD=` :

```
$ make
You must provide a BOARD parameter with 'BOARD='
Supported boards are: amulet_mp feather_nrf52832
Makefile:90: *** BOARD not defined.  Stop
```

### Common makefile problems

#### 1. `arm-none-eabi-gcc`: No such file or directory

If you get the following error ...

```
$ make BOARD=amulet_mp all
Compiling file: main.c
/bin/sh: /usr/bin/arm-none-eabi-gcc: No such file or directory
make: *** [_build/main.o] Error 127
```

... you may need to pass the location of the GCC ARM toolchain binaries to `make` using
the variable `CROSS_COMPILE` as below:
```
$ make CROSS_COMPILE=/opt/gcc-arm-none-eabi-9-2019-q4-major/bin/arm-none-eabi- BOARD=feather_nrf52832 all
```

For other compile errors, check the gcc version with `arm-none-eabi-gcc --version` to insure it is at least 9.x.

#### 2. `ModuleNotFoundError: No module named 'intelhex'`

Install python-intelhex with

```
pip install intelhex
```


#### 3. `make: nrfjprog: No such file or directory`

Make sure that `nrfjprog` is available from the command-line. This binary is
part of Nordic's nRF5x Command Line Tools.

