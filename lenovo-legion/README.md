# Lenovo Legion 7

Credit to Cameron Berkenpas (cam@neo-zeon.de) and you can find the original post [here](https://bugzilla.kernel.org/show_bug.cgi?id=208555#c294).

The sound speakers for Linux in Lenovo Legion 7 15IMH05 and 15IMHG05 can be fixed by adding a patch for `snd-hda`.
Here's how:

1. Download the [legion-alc287-0.0.5.patch](legion-alc287-0.0.5.patch).
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

### Alternative 2

You can apply the hda verbs and test the sound on speakers. Here's how:

- Download the [verbs-working.txt](verbs-working.txt) ([source](https://bugzilla.kernel.org/show_bug.cgi?id=208555#c206)).
- Install the `alsa-tools`. For Ubuntu, `sudo apt install alsa-tools`.
- `git clone https://github.com/ryanprescott/realtek-verb-tools`
- `sudo python3 realtek-verb-tools/applyverbs.py verbs-working.txt`
- Put some audio to play

## Finding the verbs

(add here the instructions about qemu)
