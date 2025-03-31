# Lenovo Yoga 7 16IAH7

Instructions for [Lenovo Legion 7 2020](https://github.com/thiagotei/linux-realtek-alc287/tree/main/lenovo-legion) work for the Yoga 7 16IAH7 too.

## Device info

```
$ dmidecode
System Information
        Manufacturer: LENOVO
        Product Name: 82UF
        Version: Yoga 7 16IAH7

```

## OS

```
$ cat /etc/os-release
PRETTY_NAME="Ubuntu 24.04.2 LTS"
NAME="Ubuntu"
VERSION_ID="24.04"
VERSION="24.04.2 LTS (Noble Numbat)"
VERSION_CODENAME=noble
ID=ubuntu
ID_LIKE=debian
```


## Kernel version

```
$ uname -srv
Linux 6.11.0-1017-oem #17-Ubuntu SMP PREEMPT_DYNAMIC Fri Feb 28 08:48:41 UTC 2025
```

## Sound device

```
$ aplay --list-devices
**** List of PLAYBACK Hardware Devices ****
card 0: PCH [HDA Intel PCH], device 0: ALC287 Analog [ALC287 Analog]
  Subdevices: 1/1
  Subdevice #0: subdevice #0
```
