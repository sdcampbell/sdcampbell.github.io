---
published: false
---
# Android-Testing-Cheatsheet - Updated 4/20/20
Configuring tools and mobile device for Android app testing.

## Download APK's

https://apkpure.com/

## Extract APK's from device

```
adb shell pm list packages | grep [name]
adb shell pm path <package name>
adb pull /path/to/app/base.apk /path/to/desired/location
```

## Extract APK

```
apktool d application.apk
```

## Convert APK to JAR

```
d2j-dex2jar /path/to/app.apk
```

## Restore factory image (unroot) Pixel

On Kali:

```
sudo apt install -y adb fastboot
fastboot erase userdata
fastboot update image-*
```

## Root Android Device (Pixel)

Note: There may be different methods required to root specific Android devices and versions. The methods in the following YouTube link are specific to a Google Pixel! You should be searching for how to root your device model and follow those instructions.

https://www.youtube.com/watch?v=zNjsb_V8NBk (Check for links in video description!)

## Install CA certificate on device

This section assumes that the device is already rooted.

If Burp Proxy: Options tabs, click 'Import/export CA certificate'. Click 'Certificate in DER format' and then click the 'Next' button. Save the file.

If ZAP: go to Tools, Options, Dynamic SSL Certificates and click the save button.

Convert the certificate:

```
openssl x509 --inform DER -in cacert.der -out cacert.pem
openssl x509 --inform PEM -subject_hash_old -in cacert.pem | head -1
mv cacert.pem <hash>.0
```

Copy cert to device: `adb push 9a5ba575.0 /sdcard/`

Remount file system r/w on physical device:

```
adb shell
su
mount -o rw,remount rootfs /
```

Remount file system r/w on Genymotion device:

```
adb remount
```

Move cert:

```
adb shell
mv /sdcard/9a5ba575.0 /system/etc/security/cacerts/
chmod 644 /system/etc/security/cacerts/9a5ba575.0
reboot
```

# Configuring Workstation

## Install adb and fastboot
```
sudo apt install -y adb fastboot
```
## Install Drozer on a workstation (Debian/Ubuntu/Kali)

1. Install prerequisites: https://github.com/fsecurelabs/drozer/#prerequisites
2. Install deb: https://github.com/FSecureLABS/drozer/releases/download/2.4.4/drozer_2.4.4.deb
3. Install Oracle Java 1.6
   1. Download from: https://www.oracle.com/java/technologies/javase-java-archive-javase6-downloads.html#license-lightbox
   2. Extract bin file and move to /usr/lib/jvm.
   3. Set JAVA_HOME in .bashrc: `JAVA_HOME="/usr/lib/jvm/java-6-oracle"`
4. Drozer agent
   1. Download agent: https://github.com/mwrlabs/drozer/releases/download/2.3.4/drozer-agent-2.3.4.apk
   2. Install Drozer on mobile device: `adb install drozer-agent-2.3.4.apk`
   3. Port forward: `adb forward tcp:31415 tcp:31415`
   4. Connect to a physical device: `drozer console connect --server <server IP (mobile device)>`
   5. List modules:Find out more information on the app: `run app.package.info -a (application)`
   6. Drozer guide: https://labs.f-secure.com/assets/BlogFiles/mwri-drozer-user-guide-2015-03-23.pdf

## Other mobile analysis tools

https://www.stevencampbell.info/Mobile-Pentesting-Tools/

## Install and run tcpdump

First you need to obtain a tcpdump binary compiled for ARM architecture. You can find it at this link: https://www.androidtcpdump.com/android-tcpdump/downloads

Install tcpdump on the rooted device:

```
adb shell
su
mount -o rw,remount rootfs / 
adb push ./tcpdump /system/xbin/tcpdump
```

Run tcpdump and select an interface: `tcpdump -D`

Start the capture, saving to sdcard: `tcpdump -vv -i any -s 0 -w /sdcard/dump.pcap`

Retrieve the capture from the device: `adb pull /sdcard/dump.cap .`

# Finding Vulnerabilities

## Start here