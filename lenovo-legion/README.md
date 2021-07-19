# Lenovo Legion 7 2020

Credit to Cameron Berkenpas (cam@neo-zeon.de) and [sycxyc](https://bugzilla.kernel.org/show_bug.cgi?id=208555#c277). You can find the original post [here](https://bugzilla.kernel.org/show_bug.cgi?id=208555#c294).

The sound speakers for Linux in Lenovo Legion 7 15IMH05 and 15IMHG05 can be fixed by adding a patch for `snd-hda`.
Here's how:

1. Download the [legion-alc287-0.0.5.patch](https://raw.githubusercontent.com/thiagotei/linux-realtek-alc287/main/lenovo-legion/legion-alc287-0.0.5.patch).
2. Copy it to `/lib/firmware/legion-alc287.patch`
      * `sudo cp legion-alc287-0.0.5.patch /lib/firmware/legion-alc287.patch`
3. Create/open `/etc/modprobe.d/lenovo-fix.conf` in your favorite text editor.
      * `sudo nano -w /etc/modprobe.d/lenovo-fix.conf`
4. Set the contents of `/etc/modprobe.d/lenovo-fix.conf` to be following then save and exit:
```
# Patch file to enable output on speakers.
options snd-hda-intel patch=legion-alc287.patch
```
5. Reboot
6. Test your sound

If necessary, you can remove `/etc/modprobe.d/lenovo-fix.conf` and reboot to disable the patch.
Some more info on `snd-hda` patch files can be found [here](https://www.kernel.org/doc/html/latest/sound/hd-audio/notes.html) for those wanting to experiment.

Tested on Ubuntu 20.04.2 Kernel 5.8.0-55.

### Alternative 1

You can apply the hda verbs and test the sound on the speakers. This is a stopgap solution that is much more useful for
debugging. For example, your speakers will stop working after a while as they will switch off to save power and the
snd-hda-intel driver will not know how to re-initialize the speakers. At this point, you will need to re-apply the verbs.

Here's how to apply the verbs to enable sound on the speakers::

- Download [verbs-legion.txt](verbs-legion.txt) (with much credit going to [sycxyc](https://bugzilla.kernel.org/show_bug.cgi?id=208555#c277) for narrowing the verbs down).
- Install the `alsa-tools`. For Ubuntu 20.04, `sudo apt install alsa-tools`.
- `git clone https://github.com/ryanprescott/realtek-verb-tools`
- `sudo python3 realtek-verb-tools/applyverbs.py verbs-legion.txt`
- Put some audio to play.

Putting the laptop to sleep and resuming results in non-working speakers. You will need to re-apply these verbs.
It's also possible that if your laptop stays on but is idle that the speakers will go to sleep and will need to be re-initialized in that case too.

### Alternative 2

Patch the kernel source code. 
(Add instructions and kernel patch examples.)

## Finding the verbs

(add here the instructions about qemu)
