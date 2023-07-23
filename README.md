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

## Features
- Charge Glove80s once every 4 months 
    - My experience seems to be that they will last even longer than this...
- Seemless use with a KVM
- RGB Underglow

## To be addressed
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
