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

## Failed attempt with Particle Debugger with openocd

Read the documentation on the Particle Debugger at <https://support.particle.io/hc/en-us/articles/360039251414-JTAG-and-SWD-Guide>. Install openocd (through brew on a mac). Install `particle-ftdi.cfg`. Does not recognize the Particle Debugger.

Turns out I had not read things carefully and combined two sets of instructions. The Particle Debugger is a CMSIS-DAP device. Wonderful piece of hardware, see <https://www.keil.com/support/man/docs/dapdebug/>. 

But the bottom line is it can only be used to debug ARM devices. So another dead end.

## Try with MiniModule and openocd

Started by saving the wiring for [esp32](../esp32/readme.md) by creating a breadboard that has the wiring from that project (so I can get back to debugging esp32) and testing that it works.

On the MiniModule we need to connect CN3-1 to CN3-3 (USB power to MiniModule VCC) and CN2-1 to CN2-11 (MiniModule 3.3V to FT2232H I/O voltage) just like for the esp32 interface. See there for an image.

Attaching the MiniModule makes it show up in `System Information` with the correct PID and VID. Using it with openocd:

```
openocd -f interface/ftdi/minimodule-swd.cfg
```

That minimodule config file tells me I need to find the pins for GND, SWCLK, SWDIO and nRESET.

results in an error (or warning?) about `LIBUSB_ERROR_ACCESS`, using `sudo` makes that go away.

Next is finding which pins on the 10-pin SWD header I need to use. Starting to read <https://www.allaboutcircuits.com/technical-articles/getting-started-with-openocd-using-ft2232h-adapter-for-swd-debugging/>.

Turns out I need pins 1 (VCC), 2 (SWDIO), 3(GND), 4 (SWDCLK) and 10 (nRESET). Looking at the 10-pin connector (or at the top of the 10 pin plug), with the gap on the left, pin 1 is top-left, 2 top-right, and so on until pin 10 bottom-right. Created a little breakout from the 2-pin mini pug to a normal 10 pin plug.

Now we need the following connections to the MiniModule:

- CN3-1 to CN3-3 (USB power to MiniModule VCC)
- CN2-1 (or 3 or 5) VIO to CN2-11 (MiniModule 3.3V to FT2232H I/O voltage)
- CN2-1 (or 3 or 5) VIO to VCC (SWD pin 1) if you want to power the sniffer board through the MiniModule
- CN2-2 GND to GND (SWD pin 3)
- CN2-7 ADBUS0 to SWCLK (SWD pin 4)
- CN2-9 ADBUS2 to SWDIO (SWD pin 2)
- CN2-10 ADBUS1 also to SWDIO (SWD pin 2)
- CN2-14 ADBUS4 to nRESET (SWD pin 10)

There was some talk in `swd-resistor-hack.cfg` about needing a 470 ohm resistor but it seems I didn't need it. It needs to go into one or the other signal lines going to SWDIO.

I can now talk to the Adafruit board through openocd, and I see that things like `reset halt` and `regs` and `reset run` work (by noting how the LED behaves).

Start by downloading nrf sniffer firmware version 3.0.0 (because of issues reported with later versions with the Adafruit board). Will later try to upgrade to 4.1.0 (which may have those issues fixed). See "Finding the Firmware" above to see which one I flashed.

This is where the good news stops: attempting to flash the device gave an error `could not write`, and the device is now bricked.

To be continued.

## Odds and ends

List of resources used (to be cleaned up):

- <https://devzone.nordicsemi.com/f/nordic-q-a/29221/flash-nrf52-on-windows-with-openocd-ftdi>
- <https://www.allaboutcircuits.com/technical-articles/getting-started-with-openocd-using-ft2232h-adapter-for-swd-debugging/>
- JTAG pinout: <http://www.max-sperling.bplaced.net/?p=6430>
- JTAG pinout: <https://www.olimex.com/Products/ARM/JTAG/ARM-JTAG-20-10/resources/ARM-JTAG-20-10.png>
- JTAG pinout: <https://docs.platformio.org/en/stable/plus/debug-tools/blackmagic.html>
- <https://devzone.nordicsemi.com/f/nordic-q-a/78791/nrf-sniffer-firmware-v3-0-0-doesn-t-work-on-nrf51822-based-board/326310>
- openocd config file `minimodule-sdw.cfg`
- openocd config file `swd-resistor-hack.cfg`
- <https://cdn-learn.adafruit.com/downloads/pdf/introducing-the-adafruit-bluefruit-le-sniffer.pdf> especially the section on "How do I convert between sniffer and builfruit LE firmware using swd".


