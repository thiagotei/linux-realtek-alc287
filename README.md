# Find your laptop's Intel HDA verb initialization sequence

Some laptops can have their speaker output enabled through an Intel HDA verb initialization sequence. You can use PCI passthrough to pass your sound device through to a Windows VM to capture the verbs sent to the sound hardware and from there determine what sequence is needed to enable audio output through your speakers in Linux. This sequence could also be used for other operating systems as well.

However, not all laptops use an Intel HDA verb sequence to enable sound. We're starting to see laptops that use amplifier chips that must be initialized over the i2c bus. Such amp chips will also likely require firmware. This document cannot help with those cases (I'm looking at you Lenovo Legion 7i 16ITHg6 and Legion 7 16ACHg6).

## Finding the verbs

# Setting up PCI passthrough.

You'll need to pass your sound card through to the Windows 10 virtual machine that you'll eventually be creating. In order to do this, you'll need to setup IOMMU. I found this Arch wiki article to be indispensable even though I personally use Kubuntu: [PCI passthrough via OVMF](https://wiki.archlinux.org/title/PCI_passthrough_via_OVMF)

# JCS Qemu

GitHub user JCS has their own "fork" of qemu with some very convenient changes to its logging output. While it would be nice to see vanilla qemu implement equivalent functionality (perhaps part of the tracing functionality), the JCS fork gets the job done.

# Now that PCI passthrough is setup, you need to get jcs-qemu built and installed:
- Clone the repo: `git clonehttps://github.com/jcs/qemu.git jcs-qemu`
- `cd jcs-qemu`
- We need the dependencies for qemu. For Debian/Ubuntu, you can run: `sudo apt install build-essential && apt build-dep qemu-system-x86`
- Now run the configure script: `./configure --prefix=/opt/qemu/jcs --enable-kvm --enable-trace-backends=log --target-list=x86_64-softmmu`
- Now we need to build jcs-qemu: `make -$(nproc)
- Next we install jcs-qemu: `make install`

# Now you need to get your Windows 10 virtual machine setup:
- Create the disk image, adjusting disk sizes where appropriate, however, Windows 10 probably needs a fair amount of space. This will create a [thin-provisioned](https://en.wikipedia.org/wiki/Thin_provisioning) disk image for your VM: `qemu-img create -f qcow2 win10.qcow2 32G`
- Download a legitimate Windows 10 installation ISO image. These can be downloaded directly from Microsoft.
- Next we need a convenience script to start qemu for us as there are a lot of options. Tune the number of CPU's, and the amount of memory as needed. Also set _iso to the path to your Windows 10 ISO image. Enter the following into a text file we'll call `start.bash` (you can call it whatever you like, just remember to replace your name for `start.bash` in subsequent instructions:
```
#!/usr/bin/env bash                                                                                                                                                                                                                         

_qemu="/opt/qemu/jcs/bin/qemu-system-x86_64"
_cpu_count="4"
_mem="4G"
_sound_card="00:1f.3"
_iso="Win10_21H1_English_x64.iso"

# Set this "d" for booting from the ISO to install Windows.
# Set this to "c" to boot from the hard disk iamge after installing Windows.
_boot_dev="d"

sudo $_qemu -enable-kvm -hda win10.qcow2 -cdrom $_iso -m $_mem -smp $_cpu_count -vga std -boot $_boot_dev \
-device vfio-pci,host=$_sound_card,multifunction=on,x-no-mmap=true \
-trace events=events.txt
```
- For `start.bash`, _sound_card should be set to the PCI ID of your sound card device. You probably already know this from setting up PCI passthrough, but you can look for it in all of your PCIE devices: `lspci`
- Add in the events.txt file with following contents (TODO: Do we really need this or does JCS Qemu not require this for its output?):
```
-vfio_region_read
vfio_region_write
```
- Set the executable flag on the script: `chmod +x start.bash`
- Start your VM up and install Windows 10 to it: `./start.bash`
- When prompted for the activation key, you can skip to activate later.

# Getting sound working under Windows

This is the most difficult part and this section will need to be fleshed out further as more people provide their
experiences to geting Windows to recognize the sound card as the correct device. So far 3 different Lenovo laptop models
needed 3 different approaches (including one where Windows recognized the device automatically). Below I document what
was needed for the 2020 Lenovo Legion 15IMH05:
- Start your Windows VM up: `./start.bash`.
- In your Windows 10 VM, browse to the location for your model of laptop's drivers. For the 2020 Lenovo Legion 15IMH05, this is [here](https://pcsupport.lenovo.com/us/en/products/laptops-and-netbooks/legion-series/legion-7-15imhg05/81yu/81yucto1ww/mp1t35d3/downloads/driver-list/)
- Go to the audio section and download your laptop's sound drivers.
- Double click on the drivers and choose to extract the drivers rather than to install (installing won't hurt anything, but Windows won't apply the drivers itself as it thinks the passed through sound card is some other device).
- Open up the Windows device manager.
- Open up the device properties of the sound device under "Sound, video and game controllers"
- Select update driver under the "driver" tab.
- Choose "Browse my computer for drivers"
- Select "Let me pick from a list of available drivers on my computer"
- Choose "have disk"
- Browse to where you extract the drivers. For me this was `C:\Drivers\Audio\201211706.15485923\Realtek\Codec_8975.1\HDXLVSST.inf`
- Press Ok
- You should see "Realtek High Definition Audio(SST)" as an option. It was the only option for me)
- Next.
- Yes.
- Restart the virtual machine when prompted.
- When Windows comes back up, try playing some sound. If it works, you'll hear sound through your laptop's speakers.
- If sound isn't working, go back to the device manager and try to mess around with updating your drivers. So far there isn't a precise process. If you don't see a sound device under "system devices" as well, you may need to get that sorted first. On the 15IMH05, I have a device under listed "System Devices" as "Intel(R) Smart Sound Technology (Intel(R) SST) Audio Controller". I also see this device booted into the laptop's Windows' partition.
- Again, this isn't yet a precise process and there seems to be some varience between laptop models. Likely you'll have to stick with it to get to a point where it's working. It's also worth trying the other drivers from the folder you extracted to.
- Once you get it working, shut down your Windows 10 virtual machine, and then backup win10.qcow2. Once or twice I ran into a situation where the sound driver under Windows stopped working and refused to work again no matter what I tried as windows would no longer recognize the sound card as the correct device. Being able to revert to a working image saved a lot of time.

# Getting the verbs from your virtual machine.

Now that you have a Windows 10 VM with working sound, it's time to grab the verbs:
- Start your VM up, this time saving the output: `./start.bash | tee output.txt`
- Play some sound by going to YouTube or whatever you prefer.
- Shutdown the virtual machine.

# Disable PCI passthrough.

In order to start working with your sound card under Linux, you need to make your sound card available for use under Linux again. You need to disable PCI passthrough for your sound card (and any devices in its IOMMU group), or else your host Linux OS won't be able to use/see your sound card. Once you've done this, you'll need to restart your laptop to apply the changes.

# Extract the verbs from the JCS Qemu output.

We have the data, now we need to extract and place it into a usable form.
- You'll need the RealTek Verb Tools from [here.](https://github.com/ryanprescott/realtek-verb-tools.git)
- Using the `cleanverbs.py` script from the RealTek Verb Tools repo, extract the verbs from your qemu output: `python3 cleanverbs.py output.txt >verbs.txt`
- We need `hda-verb` from `alsa-tools`. You can obtain this for Debian/Ubuntu based distributions like so: `sudo apt install alsa-tools`
- While playing music, use the `applyverbs.py` script from the RealTek Verb Tools repo: `sudo python3 applyverbs.py verbs.txt`
- You should probably hear at least short blip of the music you're playing. This means you have a working set of verbs, however, you'll need to work towards just the subset of verbs that you need.
- We know the set of verbs likely starts with some initialization sequence. You can use the `head` command to grab some the first arbitrary number of lines. As a random example, we'll use 500: `head -500 verbs.txt >  verbs-testing.txt`.
- Apply verbs-testing.txt to see if output works on both speakers: `sudo python3 applyverbs.py verbs.txt`
- If sound doesn't work, try different values (increasing or decreasing the value passed to head).
- If sound sound does work, try to find a minimum starting point where you have full sound working on both speakers.
- Once you have some "minimum" set of verbs that gets your system's sound to a place where it's fully functional, back up this set of working verbs and you can use `applyverbs.py` in debug mode to apply the verbs slowly so you can begin to debug and determine which verbs do what. From you'll like to be able to go from hundreds of verbs in your "minimum" set into the double digits.

# What to do with your working verbs.

Once you have a reasonable number of working verbs, you're not done. Eventually your speakers will power off (this can take a while or be fairly quick depending on your model of laptop, etc) to save power. You likely won't have working speakers after resuming or after unplugging headphones. That's because your sound card's driver doesn't know anything about the verbs it needs to initialize the speakers on your system. You can keep re-applying your verbs as needed, but this is an ugly and less than ideal work around.

A more proper and complete solution is to take this sequence and create a kernel patch.

## Patching the kernel

Once you have a narrowed down verb sequence, you can try creating and submitting a patch if you have some knowledge of
the C programming language and are comfortable with compiling your own kernel.  Here are 2 examples of working patches:
- [ Lenovo Ideapad S740 ](https://github.com/torvalds/linux/commit/26928ca1f06aab4361eb5adbe7ef3b5c82f13cf2)
- [2020 Legion, etc](https://lore.kernel.org/all/20210913212627.339362-1-cam@neo-zeon.de/)

Patches to fix audio output in this manner should be submitted to the ALSA-DEVEL mailing list, and you must follow the
guidelines found [here](https://www.kernel.org/doc/html/v5.14/process/submitting-patches.html).


