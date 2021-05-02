# Enabling Full Disk Encryption

## Partitioning

Create a ext4 on LVM on LUKS partition and copy over the manjaro root. This can be done using Jumpdrive, or when booted in a system from SD card. Keep in mind:

- If using Jumpdrive and formatting LUKS from a PC that it may choose computationally expensive PBKDF settings based on what that PC can do. If you want the unlock to be quick, either format from the phone itself or manually set lighter `--pbkdf-...` options for luksFormat.
- You can either copy an existing manjaro root filesystem directly (providing its small enough), or create a clean filesystem and copy the files over (e.g. using a tar pipe). I prefer the latter.
- If creating a fresh filesystem, make sure it has a decent number of inodes. See man mkfs.ext4, -N option.

## fbkeyboard

Build https://github.com/bakonyiferenc/fbkeyboard - that seems to be the repo used by postmarket OS.

Patch the source to reference /usr/share/fonts/TTF/DejaVuSans.ttf instead of /usr/share/fonts/ttf-dejavu/DejaVuSans.ttf

## Reconfiguring the system

Mount the ext4-on-lvm-on-luks filesystem on /mnt. Also make sure the appropriate boot partition is mounted as /mnt/boot.

Ensure fbkeyboard is installed to /mnt/usr/bin/fbkeyboard

chroot into it (preferably using arch-chroot), but you can alternatively manually set up the dev, proc, sysfs mounts and use vanilla chroot. It's best to do this on the phone itself, but you could also try using qemu-static - just make sure the qemu static binary is in /mnt/usr/bin/ and that the binfmt is registered.

### From within the chroot

Alter /etc/fstab with the updated root (and boot) devices.

Alter /etc/mkinitcpio.conf:
  - Copy the provided fbkeyboard hooks to /etc/initcpio/hooks and /etc/initcpio/install as appropriate.
  - Add "encrypt" and "lvm2" hooks. Add our custom "fbkeyboard" hook BEFORE "encrypt". Hooks should read something like `...`.
  - `mkinitcpio -P`. The errors about missing dm_... modules can be ignored - we're not using those specific dm features.

Alter boot.txt:
  - add a uuid_crypt partition variable
  - reorder console=tty0 after console=${console} - this is needed to get working console output on the screen. DO NOT remove the console=${console} or earlycon=... options, as doing so somehow breaks certain power management features such as waking from suspend or rebooting.
  - remove the bootsplash and quiet options
  - add cryptdevice=${uuid_crypt}:PineCrypt (or whatever name you use)
  - alter root option to say root=/dev/PineVG/Root or whatever name you use.
  - run `./mkscr`
