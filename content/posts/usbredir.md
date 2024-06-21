+++
title = "Using usbredir with qemu"
date = 2024-06-21
+++

## The why

So assume that you have some USB device that is connected to a machine on your network that isn't your primary machine. But you want to use QEMU's [USB passthrough capabilities](https://qemu-project.gitlab.io/qemu/system/devices/usb.html#connecting-usb-devices) with `usb-host`, perhaps to develop drivers for [managarm](https://github.com/managarm/managarm). The device might just be very inaccessible, or built-in to that other machine. The point is, connecting it to your primary machine is not feasible. What do we do? Unfortunately, most online sources seem to assume libvirt and SPICE setups, which is of little use to me.

## The how

[`usbredir`](https://www.spice-space.org/usbredir.html) is a protocol for running USB traffic over TCP. This has two sides; we need to run a server on the machine that has the USB device plugged in, and the client on the machine where we would like to use that USB device. The server part is provided by [`usbredir`](https://gitlab.freedesktop.org/spice/usbredir). Thankfully, qemu has built-in support for the client part **if** it was built with `usb-net-redir` support. To check support, check the output of:

```sh
qemu-system-x86_64 -device ? | grep "name \"usb-redir\""
```

### Running the server

```sh
usbredirect --device <idVendor>:<idProduct> --as 0.0.0.0:42069
```

Choose the port number to your liking and remember it. The server is limited to one single device; if you want to redirect multiple devices, just launch multiple servers on different ports. Also, note that the server will only accept one single connection, and shut down afterwards. Place it in a while loop if you plan on using it over and over.

### Connecting qemu

```sh
qemu-system-<arch> -chardev socket,id=usb-redir-chardev0,port=42069,host=<server ip> -device usb-redir,chardev=usb-redir-chardev0,id=usb-redir0,bus=xhci.0
```

Rinse and repeat for additional devices.
