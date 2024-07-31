---
layout: default
title:  Encrypt second drive when the first is encrypted with TPM
date:   2024-07-31 16:00 -03:00
author: Murcio Filho
tags:
  - Linux
  - Encryption
parent: Linux
has_children: false
---

# Encrypt second drive when the first is encrypted with TPM

After installing a [[Linux]] distro using TPM as an option, the secondary partitions isn't encrypted because the usual installers, like the Ubuntu installer, only encrypt the main partition, needing a workaround to protect the other drives.

To do that, you need to generate a key and insert it in the root folder located in the main partition protected with TPM:

```bash
# Generating the key
dd if=/dev/random bs=64 count=1 | xxd -p -c999 | tr -d '\n' > luks_key

# Moving the key to the root folder
sudo cp luks_key /root
```

So, the key generated will be located in the ``/root`` folder

After that, you need to format the secondary drive with LUKS and, after that, add the key generated located in the TPM drive:

```bash
# Formatting the other drive with LUKS, but you can use the Gnome Drive Formatter if preferred.
sudo cryptsetup luksFormat ${YOUR_DRIVE}

# Adding the luks_key after formatting
sudo cryptsetup luksAddKey ${YOUR_DRIVE} /root/luks_key --pbkdf-force-iterations=4 --pbkdf-parallel=1
```

After formatting, mount the partition and check the encrypted second drive name in a file manager. In my case, the Gnome File Manager reports that my partition is named as ``/dev/dm-2``.

Furthermore, to create an automatic unlock, you need to set up the ``/etc/crypttab`` and ``/etc/fstab`` files, note that I'm using the partition naming (``/dev/dm-2`` and ``/dev/sda``) found on my machine, so change the naming according to the reported in your system:

``/etc/crypttab``

```
dm-2    /dev/sda    /root/luks_key    luks,discard,noatime
```

``/etc/fstab``

```
/dev/dm-2    /media/${USER_NAME}/DATA    ext4    noatime,discard    0    0
```

After that, the system should unlock the encrypted LUKS secondary partition while booting, using the TPM encryption to store the LUKS keys.

## References

1. [The ultimate guide to Full Disk Encryption with TPM and Secure Boot (with hibernation support!)](https://blastrock.github.io/fde-tpm-sb.html)
2. [dm-crypt/Device encryption](https://wiki.archlinux.org/title/Dm-crypt/Device_encryption)


[//begin]: # "Autogenerated link references for markdown compatibility"
[Linux]: ../Linux "Linux"
[//end]: # "Autogenerated link references"