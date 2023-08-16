# Encrypted ISO disc

## How to create this disk


### Create base Slack ISO image
1) download stock slax
2) apt install gpg, cryptsetup, syslinux-utils (provides hybrid iso) and other tools
3) use savechanges
4) create new ISO
5) make it hybrid


### Create ISO image

1) Slax ISO (modded or stock)
2) If it is bootable iso, make it hybid iso with MBR (if it is not already)

3) Create GPG key, use strong password
  - echo "some random key, can be used also from /dev/urandom" | gpg -q -c > key.txt

4) create SquashFS, which will be encrypted
  - mksquashfs private/ private.sqfs -all-root

5) Encrypt by using LUKS2
  - truncate -s +8M private.sqfs
  - gpg -q -d testenc/key.txt | cryptsetup reencrypt --key-file=- --encrypt --type luks2 --resilience none --disable-locks --reduce-device-size=8M private.sqfs
  - truncate -s -4M private.sqfs

6) resize key to 2048 bytes
  - truncate -s 2048 key.txt

6) Combine key and SquashFS with ISO (SquashFS is already padded to 4096 bytes)
  - cat key.txt private.sqfs >> output.iso

7) use fdisk -w never output.iso
  - use "c" to enable old-DOS partition (support less than 1MB size)
  - create new partition with +3 sectors (total 4 sectors of 512, for key)
  - create new partition rest of image (SquashFS)


OPTIONAL: Check ISO in some virtual environment

7) Burn ISO


### Mount private part
1) Prepare Linux environment with Cryptsetup with support LUKS2 and GPG
2) Prepare loop device with partition detect (if hybrid ISO was used)
  - losetup -P /dev/loop10 /dev/sr0

3) Open LUKS2 device with GPG key by using password
  - cat /dev/loop10p2 | gpg -q -d | cryptsetup --key-file=- open /dev/loop10p3 private

4) Mount encrypted mapper device
  - mount /dev/mapper/private /mnt/private
