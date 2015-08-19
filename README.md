## XBeeBoot: XBee Series 2 API Bootloader for Arduino and Atmel AVR ##

The XBeeBoot bootloader provides XBee Series 2 Over-The-Air firmware update capability to Atmel AVR devices, as well as supporting direct firmware update via the standard Optiboot protocol.

It provides the following features:

  * Bootloader fits within 1kB: This is larger than [Optiboot's]
    (https://github.com/Optiboot/optiboot) smallest build, but is still half
    the size of a standard Arduino bootloader.

  * Reasonably fast Over-The-Air firmware updates: Less than three minutes to
    write a 20kB firmware image.

  * Natively supports XBee API-mode protocol: No need to use AT-mode XBee
    firmware.

  * Over-The-Air updates are secure: XBee security models apply - so firmware
    updates are encrypted and authenticated with AES encryption if the Zigbee
    network is configured with encryption.

  * Over-The-Air updates are robust: The protocol uses direct addressing, so
    the other devices on the Zigbee network are unaffected.  You could even
    Over-The-Air update firmware on multiple devices concurrently.

  * Full access to Optiboot-supported bootloader facilities with a direct
    connection via standard [avrdude] (http://www.nongnu.org/avrdude/)
    software: This bootloader detects an attempt to program via the standard
    Arduino/avrdude firmware update protocol and automatically switches to the
    standard Optiboot protocol.

  * Full access to Optiboot-supported bootloader facilities Over-The-Air via
    patched avrdude software.


#### What hardware support does it need? ####

The Atmega device needs to be able to communicate serially with the XBee.
This requires connecting the XBee DOUT (pin 2) to the Atmega RXD (Atmega328P
pin 2), and connecting the XBee DIN (pin 3) to the Atmega TXD (Atmega328P pin
3).  You've probably already done that.

Apart from the serial link, there is only one additional connection required:
We need a mechanism to hard-reset the Atmega to enter the bootloader.  This is
supported by connecting the XBee DIO3 pin (pin 17) to the Atmega RESET pin
(Atmega328P pin 1).  Optionally, a 0.1uF capacitor can be included in the
reset pin connection - this isn't mandatory, but including the capacitor does
eliminate the possibility of the XBee accidentally holding the Atmega in a
permanent state of reset.

This is intentionally identical circuitry to [SparkFun's tutorial]
(https://www.sparkfun.com/tutorials/122) for use with XBee Series 1 devices.


#### Are there any limits on which XBee can bootload which XBee? ####

No.  In particular, it doesn't matter if the coordinator node is a separate
and unrelated node.  Any XBee address can bootload to any other
Zigbee-networked XBee-hosted AVR device, so long as they share encryption
keys.


#### Does it work with XBee modules in endpoint mode? ####

Yes it does, although it is of course a little slower getting going if the
remote device is sleeping.  Once it is going it appears to run something like
25% slower than a normal router mode device, E.g. a little over three minutes
for a 20kB update, rather than a little under three minutes.


#### Does it work with XBee modules in AT mode? ####

Not intentionally.  Because the AT firmware isn't really usable in any
environment with more than two XBee devices, I haven't looked at AT mode very
carefully.


#### Are there any limits on the number of Zigbee nodes in the network? ####

There are no known limits.  I currently have eight active XBee devices on the
same Zigbee network, meshed over some fairly complicated terrain, with a mix
of router and endpoint devices, and can update the firmware successfully and
reliably across the network.


#### Can this bootloader be used without any XBee devices? ####

Yes it can.  In two ways, in fact:

  1. You can still use it as if it were a normal Optiboot/Arduino bootloader,
     with the caveat that the default serial baud rate is 9600 baud, to match
     the XBee default.

  1. You can also bootload, with the avrdude xbee programmer directly, by not
     giving a Zigbee address.  This is useful for diagnosis and proof of
     concept testing.  It may also be useful for bootloading over unreliable
     serial links (E.g. AT mode XBee links), as the XBee bootloader protocol
     includes checksums, is packet-based and can recover from lost or
     corrupted packets in the serial data stream.
