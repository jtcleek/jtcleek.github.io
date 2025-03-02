---
title: "I use Arch (btw)"
date: "2025-02-27"
locale: "en_US"
categories: ["arch", "linux", "howto",]
---

{{< admonition type="warning" >}}
This post serves as documentation of my install on a laptop; It is not meant as an exhaustive guide on how to install Arch. My steps may be helpful to you, they may not. Use at your own risk.
{{< /admonition >}}

# Preface

## Secureboot

Enter BIOS and clear secureboot keys.

# Install

Download ISO from https://archlinux.org/download/ and write to USB drive. Boot laptop from USB and set root password. From a seperate system, ssh to the target device.

```shell
ssh root@target.device
```

## Base system

Confirm Secureboot setup mode
```shell
bootctl status | grep "Secure Boot"
```

Clear previous boot entries (if reusing disk)
```shell
efibootmgr --unicode
efibootmgr --bootnum 0001 --delete-bootnum --unicode
```

Prepare disk
```shell
export disk="/dev/nvme0n1"
sgdisk --zap-all --clear $disk
nvme id-ns -H $disk | grep "Relative Performance"
nvme format --lbaf=1 $disk --force
```

Wipe disk (if reusing)
```shell
cryptsetup open --type plain -d /dev/urandom $disk target
dd if=/dev/zero of=/dev/mapper/target bs=4096MiB \
  status=progress iflag=fullblock oflag=direct
cryptsetup close target
```

Create partitions
```shell
sgdisk -n 0:0:+1GiB -t 0:ef00 -c 0:esp $disk
sgdisk -n 0:0:0 -t 0:8309 -c 0:luks $disk
```

LUKS encrypt
```shell
cryptsetup luksFormat ${disk}p2 \
  --cipher aes-xts-plain64 \
  --key-size 512 \
  --hash sha512 \
  --pbkdf argon2id \
  --pbkdf-memory 1048576 \
  --pbkdf-parallel 4 \
  --iter-time 5000 \
  --sector-size 4096
cryptsetup open ${disk}p2 cryptroot
```