---
published: true
title: Error Flashing CrazyRadio PA USB Dongle on VMWare
---
One of the prerequisites for using [jackit](https://github.com/insecurityofthings/jackit) with a CrazyRadio PA USB dongle is first flashing it using [mousejack](https://github.com/BastilleResearch/mousejack). This has always been problematic when using VMware. I was getting the error "The connection for the USB device 'Nordic ASA nRF24LU1P-F32 BOOT LDR' was unsuccessful." which looked like a VMware dialog, not a dialog generated by the virtual machine (Kali Linux).

To fix this, ensure that your VMware vm settings for USB compatibility are set on USB version 2. Then if you're still getting the error, edit "mousejack/nrf-research-firmware/prog/usb-flasher.py" line 41 to increase the timeout. I set mine to 10000. It seems that in addtion to not being compatible with USB version 3, you can run into delays because of VMware switching the hardware between host and vm.

Hope this helps someone.
