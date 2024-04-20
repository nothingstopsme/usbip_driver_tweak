# usbip_driver_tweak

This tweak is aimed at circumventing the error "(attach_device) cannot find device" occuring when trying to attach an exported device from a remote client.

## My Case
My printer, which used to be exported by a usbip host (host a) on one machine running kernel v4.4 and attachable via a usbip windows client, can not be attached by the same client when being exported by another usbip host (host b) on a different machine running kernel v5.5; specifically, the client will thwart the attempt to attach complaining "(attach_device) cannot find device".

## Cause
The root cause remains obscure, but the immediate cause has been spotted:
unlike the list of exported devices returned from host a, the one from host b showed no usb interfaces of the printer existed

<table>
<tr><th>Host</th><th>Device list (partial)</th></tr>
<tr><td>host a (the successful case)</td>        
<td><pre>
2-1.6: Canon, Inc. : unknown product (04a9:173a)
        : /sys/devices/pci0000:00/0000:00:1d.0/usb2/2-1/2-1.6
        : (Defined at Interface level) (00/00/00)
        :  0 - Vendor Specific Class / unknown subclass / unknown protocol (ff/00/ff)
        :  1 - Printer / Printer / Bidirectional (07/01/02)
</pre></td></tr>
        <tr></tr>
<tr><td>host b (the failing case)</td>
<td><pre>
1-1.2: Canon, Inc. : unknown product (04a9:173a)
        : /sys/devices/pci0000:00/0000:00:1a.0/usb1/1-1/1-1.2
        : (Defined at Interface level) (00/00/00)
</pre></td></tr>
</table>

A further examination on the sysfiles in the folder "/sys/devices/pci0000:00/0000:00:1a.0/usb1/1-1/1-1.2" reveals that the content of the "bConfigurationValue" was empty and there were no interface folders located inside either. That implied this device was not configured at the moment. Simply assigning bConfigurationValue with a proper value made the device attachable again.

Interestingly, this device only became unconfigured after binding; once it had been unbound from the usbip host, it got configured as usual.

## Solution
Since it is just devices not being configured, the direct way to address it is to feed the sysfile "bConfigurationValue" with a valid configuration value, either by hand manually or via some uevent rules automatically, to configure the target devices; alternatively and as done in this tweak, that setting can be conducted at the driver level, which allows more flexibility in deciding when the configuration is needed and what configuration value should be used for each newly-bound device.

## Build/Install
Execute the following commands (Assuming on a Debian-based Linux)

```bash
make
sudo make install
```

That will only replace the module "usbip-host.ko" on the system with the tweaked version. You can also update all drivers making up the whole usbip facility if you like, by running

```bash
make all
sudo make install_all
```

Note that if you find this driver (based on the code from the Linux repository up to this [commit](https://github.com/torvalds/linux/commit/66cce9e73ec61967ed1f97f30cee79bd9a2bb7ee)) not compatible with your kernel, as the tweak only consists of a few lines of code you could:

1. Clone [the Linux repository](https://github.com/torvalds/linux) and check-out the desired version.
2. Replace the Makefile with the one from this tweak (might need to update the .o files/modules listed in the Makefile accordingly).
3. Copy/Paste the changes made in this [commit](https://github.com/nothingstopsme/usbip_driver_tweak/commit/cb496aff309cf6655b15c58bbb3cdb15c31cf0fa) to the corresponding files/function bodies.

That should allow you to build your own version against the kernel you prefer.
