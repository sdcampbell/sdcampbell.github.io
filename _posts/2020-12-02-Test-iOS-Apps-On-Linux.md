---
published: true
---
In the past I'd use a Mac when I needed to test iOS apps due to getting an error in Linux about needing to mount an Apple Developer Disk image when trying to use Frida and Objection on Ubuntu to test an iOS device. Here I've outlined the steps to download the Apple Developer Disk image, mount the image on Ubuntu, and run some tools including taking a screenshot of the device, when testing on a jailbroken device.

Install the necessary tools: `sudo apt-get install libimobiledevice usbmuxd`

[Download](https://github.com/xushuduo/Xcode-iOS-Developer-Disk-Image/releases) the appropriate developer disk image.

Extract the files, then mount the disk image: `ideviceimagemounter -t Developer DeveloperDiskImage.dmg DeveloperDiskImage.dmg.signature`

Check to ensure the disk image was mounted: `ideviceimagemounter -l`

Take a device screenshot: `idevicescreenshot [OPTIONS] [FILE]`

Happy Hacking!
