# Zephyr™ Mechanical Keyboard (ZMK) Firmware

[![Discord](https://img.shields.io/discord/719497620560543766)](https://zmk.dev/community/discord/invite)
[![Build](https://github.com/zmkfirmware/zmk/workflows/Build/badge.svg)](https://github.com/zmkfirmware/zmk/actions)
[![Contributor Covenant](https://img.shields.io/badge/Contributor%20Covenant-v2.0%20adopted-ff69b4.svg)](CODE_OF_CONDUCT.md)

[ZMK Firmware](https://zmk.dev/) is an open source (MIT) keyboard firmware built on the [Zephyr™ Project](https://www.zephyrproject.org/) Real Time Operating System (RTOS). ZMK's goal is to provide a modern, wireless, and powerful firmware free of licensing issues.

Check out the website to learn more: https://zmk.dev/

You can also come join our [ZMK Discord Server](https://zmk.dev/community/discord/invite)

To review features, check out the [feature overview](https://zmk.dev/docs/). ZMK is under active development, and new features are listed with the [enhancement label](https://github.com/zmkfirmware/zmk/issues?q=is%3Aissue+is%3Aopen+label%3Aenhancement) in GitHub. Please feel free to add 👍 to the issue description of any requests to upvote the feature.

# Dongle Setup

This branch aims to allow the Glove80 to be used with an nRF52840 dongle. This will allow both Glove80s to work as peripherals where the dongle will act as the central.

Please see the instructions below for creating this setup. 

**Note: This is not an official release from MoErgo. Proceed with caution. If the steps are not followed correctly you may damage your nRF52840 and/or your Glove80. I will not be responsible for any damage as a result of following this guide.**

## Features
- Charge Glove80s once every 4 months 
    - My experience seems to be that they will last even longer than this...
- Seemless use with a KVM
- RGB Underglow

## <a id='to-address'> To be Addressed</a>
- All RGB indicators
    - Currently only see the power level of the left hand Glove80
- Dongle pairing issue
    - It can be the case that if you remove and reinsert the dongle from your USB slot, it may not connect to your Glove80s again immediately. The current fix is:
        1. Turn off both Glove80s
        2. Place the nRF52840 dongle into DFU mode by pressing reset twice
        3. Remove the nRF52840 dongle from the USB port
        4. Place the nRF52840 dongle back into the USB port
        5. Turn one Glove80 on and start typing until you see a response
        6. Turn the other Glove80 on and start typing until you see a response

        **Note: Repeat the steps if this didn't work the first time.**

## Flashing nRF52840 with Adafruit's nrf52 Bootloader

<details>
<summary> This guide will walk you through the steps to flash the Adafruit nRF52 bootloader to your Nordic nRF52840. </summary> 

### Prerequisites

Before you begin, make sure you have the following:

- nRF52840 dongle
- ST-Link-V2/J-Link
- [`OpenOCD`](https://openocd.org/pages/getting-openocd.html)
```bash
apt-get install openocd
```
- `nrf52840_bboard_bootloader-<version>.hex` firmware file from [Adafruit's Github.](https://github.com/adafruit/Adafruit_nRF52_Bootloader/releases)


### Install Adafruit nrfutil Bootloader on nRF52840 using OpenOCD

1. **Connect the Debugger**: Connect the SWD pins from the debugger to the nRF52840 chip. The SWD pins are usually labeled SWDIO and SWDCLK. If you are using a J-Link debugger, you will also need to connect the VDD (3.3V) and GND pins to power the nRF52840 during the programming process.

2. **Start OpenOCD as a Telnet Server**: Open a terminal or command prompt and start OpenOCD as a Telnet server, specifying the transport type based on your debugger:

   For ST-Link (hla_swd - Serial Wire Debug):
   ```bash
   openocd -f interface/stlink.cfg -c "transport select hla_swd" -f target/nrf52.cfg
   ```

   For J-Link (swd - Serial Wire Debug):
   ```bash
   openocd -f interface/jlink.cfg -c "transport select swd" -f target/nrf52.cfg
   ```

3. **Connect via Telnet**: Now that OpenOCD is running as a Telnet server, you can connect to it via Telnet. Open a new terminal or command prompt window and run:
   ```bash
   telnet localhost 4444
   ```

   This will establish a Telnet connection to OpenOCD running on your local machine.

4. **Erase the Flash**: Once connected via Telnet, you can issue the `nrf5 mass_erase` command to erase the flash memory of the nRF52840:
   ```bash
   nrf5 mass_erase
   ```

   **Note**: Do not disconnect the nRF52840 after performing the mass erase when using an ST-Link-V2. The mass erase operation resets the voltage register, which may interfere with further programming using the ST-Link-V2.

5. **Program the Bootloader**: Download the Adafruit nrfutil bootloader HEX file from the Adafruit GitHub repository. Then, program the bootloader onto the nRF52840 using the following command:
   ```bash
   flash write_image bootloader.hex
   ```

   Replace `bootloader.hex` with the filename of the Adafruit nrfutil bootloader HEX file you downloaded, such as:

   ```bash
   flash write_image nrf52840_bboard_bootloader-0.7.0_s140_6.1.1.hex 
   ```

6. **Verify the Image**: After programming the bootloader, you can verify the image using the following command:
   ```bash
   verify_image bootloader.hex
   ```
   For example:
   ```bash
   verify_image nrf52840_bboard_bootloader-0.7.0_s140_6.1.1.hex 
   ```

   **Note**: If the verification fails, it indicates that the write was not successful. In that case, you will need to rerun step 4 (mass erase) and then step 5 (bootloader programming) to ensure the correct flashing of the bootloader onto the nRF52840.

7. **Reset the Device**: After programming and verifying the bootloader, you can reset the device using the following command:
   ```bash
   reset run
   ```

8. **Exit Telnet**: To exit the Telnet connection, simply type:
   ```bash
   exit
   ```

   This will close the Telnet session.

Remember to adjust the filenames, paths, and configurations according to your specific setup. If you are using a J-Link, make sure to connect VDD and GND to power the nRF52840 during the programming process. If you are using an ST-Link-V2, refrain from disconnecting the nRF52840 after performing the mass erase to avoid potential communication issues. Always exercise caution when working with bootloaders and firmware.

</details>

## Installing Firmware

<details>

<summary> These are the steps to build and install the relevant firmware for your Glove80 and dongle. </summary> 

### Prerequisites 
Prior to proceeding please see the [ZMK documentation](https://zmk.dev/docs/development/setup) about installing all dependencies so you can build your firmware locally.

### Importing a Custom Keymap

To copy over an existing keymap, export your keymap from the Glove80 layout editor. If you already have your keymap file in a different location, that will work too.

This keymap file needs to be copied to:

```bash
app/boards/shields/glove80_dongle/glove80_dongle.keymap
```
**Note: Ensure that the file name is `glove80_dongle.keymap`**
### Building Firmware

There are three separate files we will build, one for each half of our Glove80 and one for the dongle. First change into the app directory by:
```bash
cd app
```
You can then run the following commands, `-d` specifies the directory so feel free to specify a different location. 
```bash
west build -p -d build/glove80_lh -b glove80_lh
```

```bash
west build -p -d build/glove80_rh -b glove80_rh
```

```bash
west build -p -d build/dongle -b nordic_nrf52840_dongle_slicemk -- -DSHIELD=glove80_dongle
```

### Installing Firmware
To install the firmware we are just required to copy over the files to our devices. First we will reset our bluetooth pairing bonds and then place our devices in DFU mode.

#### Glove80 Left Hand 
1. First turn off the Glove80 left hand side via the power switch.
2. To reset the bonds, on the default key layout, press and hold `Magic` and `3` while switching the power button on. Hold these keys for 10 seconds.
3. Now turn off the Glove80 left hand side and connect a USB from the Glove80 to your computer. 
4. To enter DFU mode, on the default key layout, press and hold `Magic` and `E`. While this is being held, switch on the power switch of the left hand side.
5. Your bootloader will then appear as USB Mass Storage Device `GLV80LHBOOT` which signifies being in DFU mode.
6. Copy the file `app/build/glove80_lh/zephyr/zmk.uf2` (or your specified location) to the root directory of the USB Mass Storage device. 

#### Glove80 Right Hand
1. First turn off the Glove80 right hand side via the power switch.
2. To reset the bonds, on the default key layout, press and hold `PgDn` and `8` while switching the power button on. Hold these keys for 10 seconds.
3. Now turn off the Glove80 right hand side and connect a USB from the Glove80 to your computer. 
2. To enter DFU mode, on the default key layout,press and hold `I` and `PgDn`. While this is being held, switch on the power switch of the right hand side.
3. Your bootloader will then appear as USB Mass Storage Device `GLV80RHBOOT` which signifies being in DFU mode.
4. Copy the file `app/build/glove80_rh/zephyr/zmk.uf2` to the root directory of the USB Mass Storage device. 

#### nRF52840 Dongle
1. Double press the reset switch in quick succession to enter DFU mode.
2. Your bootloader will then appear as USB Mass Storage Device `BBOARDBOOT` which signifies being in DFU mode.  
2. Copy the file `app/build/dongle/zephyr/zmk.uf2` to the root directory of the USB Mass Storage device. 

### <a id='post-installation'>Post Installation</a>
If all things went well you should be able to type successfully via your nRF52840 dongle. 

If you are still have some trouble: 
- Try and reset the bonds for each individual Glove80 (step 2 in the above section)
- Try the dongle pairing issue fix mentioned under [To be Addressed](#to-address)
- Check out the troubleshooting section 

</details>

## Troubleshooting
<details>
<summary> Steps to help resolve issues. </summary>

Before proceeding try some of the steps contained within [Post Installation](#post-installation) within Installing Firmware.

### Clearing Bonds
If you are finding that the Glove80 is no longer pairing with the nRF52840 dongle, and have tried the steps above for the pairing issue, you can reset the bonds of your Glove80 and your dongle. 

#### nRF52840 Dongle

1. First build the settings reset firmware via the following:
    ```bash
    west build -p -d build/settings_reset -b nice_nano -- -DSHIELD=settings_reset
    ```

2. Put the nRF52840 into DFU mode by double pressing the reset switch in quick succession.
3. Copy over the file from `app/build/settings_reset/zephyr/zmk.uf2` to the root directory of the USB Mass Storage device. Once copied over this will reset the bonds
4. Enter DFU mode by clicking reset twice in quick succession
5. Now copy over the nRF52840 dongle you previously built to `BBOARDBOOT` USB Mass Storage device

#### Glove80 Left Hand 
1. First turn off the Glove80 left hand side via the power switch.
2. To reset the bonds, on the default key layout, press and hold `Magic` and `3` while switching the power button on. Hold these keys for 10 seconds.

#### Glove80 Right Hand 
1. First turn off the Glove80 right hand side via the power switch.
2. To reset the bonds, on the default key layout, press and hold `PgDn` and `8` while switching the power button on. Hold these keys for 10 seconds.

### Viewing Logs
By default, logging is enabled on the nRF52840. Usually this is turned off for wireless keyboards as it drains the battery. To view the logs you can use `tio` via following command:

```bash
sudo tio /dev/ttyACM0
```

The information here should be used to help with further debugging. 
</details>