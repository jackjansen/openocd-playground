# Re-flashing Bluefruit LE Sniffer

These notes were taking trying to upgrade the BLE sniffer from v1 software (which was the standard when the device was purchased in 2016 or so) to v2 (which is compatible with current wireshark versions and MacOS/Linux.

Notes on the Adafruit site <https://www.adafruit.com/product/2269> state that you need a JTAG flasher (or SWD? Some times the terms are used interchangeable, some times it seems SWD is the 1.2mm 10 pin header and JTAG the 2.54mm 20 pin header...) and the newest firmware from the Nordic website <https://www.nordicsemi.com/Products/Development-tools/nRF-Sniffer-for-Bluetooth-LE>.

I have two flashers to try:

- The Particle Debugger, which has the right 10-pin cable.
- A FT2232H MiniModule, also used for the [esp32](../esp32/readme.md) openocd experiments.

I would prefer to start with the first one (so I don't have to rewire the MiniModule).

## Finding the firmware

The Bluefruit LE Sniffer isn't listed on the Nordic firmware. Googled around and came across <https://devzone.nordicsemi.com/f/nordic-q-a/78791/nrf-sniffer-firmware-v3-0-0-doesn-t-work-on-nrf51822-based-board/326310>.

So, I'm going to download 3.0.0 first, if that works try upgrading to 3.1.0. Then we'll see if we can get to 4.x.

And it seems the firmware to try and flash is `pca10028` or `nrf51dk_nrf51422`. We'll see.

## Failed attempt using nRF connect

Downloaded nRF connect (on Windows), which also installed the Segger J-Link software. Neither of the devices are apparently J-link compatible, so this was a dead end.

## Try Particle Debugger with openocd

Read the documentation on theParticle Debugger at <https://support.particle.io/hc/en-us/articles/360039251414-JTAG-and-SWD-Guide>. Install openocd (through brew on a mac).

