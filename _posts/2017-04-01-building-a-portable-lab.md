---
layout: post
title: Building A Portable Lab
---

## Introduction

This post will show you how to setup a portable lab that can house all your day to day security tools.

For quite a while I maintained two labs, one for work and one for home. Both had sets of tools that would expand apart from one another and after a while it became too cumbersome.

I had the bright spark idea loading a Windows XP virtual machine onto a USB drive and attempting to run it. It works but as soon as you do anything that required the transfer for data. The virtual machine ground to halt, as expected.

I wondered if results would be better with a Solid State Drive (SSD) and E-Sata or USB 3.0.

A co-worker finally pulled the trigger on the idea and turns out it works well so I decided to build my own.

---

## Required Components

Here are the recommended parts for a basic portable lab.

- Solid State Drive
- Sata to USB 3.0 Adapter/E-Sata
- Protect Drive Case

For my build I used the following components, these cost a total of $156 AUD.

- Samsung 850 Evo 250GB = $120
- StarTech USB 3.0 to 2.5” SATA HDD Adapter Cable = $26
- J. Burrows Portable Hard Drive Case Black = $10

I would recommend getting a 500GB+ capacity hard drive. At the time of writing this I have only 83GB free.

---

## Setup

Once you've pulled apart all the packaging and assembled the portable lab you can get down to the fun stuff.

### Encryption

Having a lab that is portable increase it's risk profile, it's much more prone to loss or theft. So you should consider deploying the same protections to it as you would a workstation.

Generally I would tell you not to keep anything confidential on a portable lab. My lab only has free tools, virtual machines and a few malware samples. So I could go without encryption but that would be against best practice.

#### VeryCrypt

The tool I decided to use was [VeraCrypt](https://veracrypt.codeplex.com/). I attempted to use BitLocker-To-Go but it turned out to have incompatibility issues.

VeraCrypt offers a [Portable Mode](https://veracrypt.codeplex.com/wikipage?title=Portable%20Mode). This allows you to run the executable without installation.

### Partitioning

To keep things simple I decided to create two partitions on the SSD.

- Public NTFS Volume (1GB)
- Private NTFS VeraCrypt volume (231GB)

This allows you to house the portable tools needed to mount the drive. You don't have to worry about having them available on any machines you decide to use the lab on.

Partitioning can all done via [Disk Management](https://support.microsoft.com/en-us/help/17418/windows-7-create-format-hard-disk-partition).

### Encrypting

Once partitioned you can move to encrypting your drive. I installed VeraCrypt using the 'Extract' method as I won't be needing it on my workstations.

Follow the below steps to encrypt your partition.

1. Select `Create Volume`.
2. Select `Encrypt a non-system partition/drive`.
3. Leave radio button checked on `Standard VeryCrypt Volume`.
4. Select `Device` and select your partition.
5. Leave radio button checked on `Create encrypted volume and format it`.
6. Leave `AES` selected under `Encryption Algorithm`, under `Hash Algorithm` select `SHA-256`.
7. Confirm your volume size.
8. Input your password, be sure to store it somewhere safe.
9. Optional, select `Use keyfiles` and select a keyfile on your choosing.
10. Under `Options` change `Filesystem` to `NTFS`.
11. Generate a random pool by moving your mouse and select `Format`.

Encrypting can take quite a while, around 1-4 hours depending on your system.

---

## Traveler Disk Setup

Once your volume is encrypted you can deploy VeraCrypt's [Traveler Disk](https://veracrypt.codeplex.com/wikipage?title=Portable%20Mode) to your public NTFS partition with the following instructions.

1. Select `Tools` and then `Traveler Disk Setup`.
2. Select `Browse` and select your partition.
3. Optional, adjust `AutoRun Configuration` settings to your requirements.
4. Select `Create`.

This will deploy a lighter version of VeraCrypt to your public volume.

---

## Mounting

Once encrypted and your portable files loaded. You can mount the partition with the following instructions.

1. Plug in your portable drive.
2. Execute `VeraCrypt.exe`.
3. Select which drive letter you want to mount your lab on.
4. Under `Volume` select `Select Device` and select your encrypted partition.
5. Select `Mount`.
6. Enter your password.
7. Optional, select `Mount Options` and adjust settings to your requirements.
8. Select `OK`.

Wait for your volume to mount, this can take a few minutes.

---

## Closing

Congratulations! You've now got a portable lab, you can now begin moving over all your tools.

I've found a portable lab to be an indispensable tool for my day to day work. Being able to have a ever changing tool set on hands at all times is a real game changer. Best of all your spend more time doing the interesting stuff and less on maintenance.

I hope you found this post informative. If you have any questions you can contact me via the [About](/about/) section.

---

## Tips

Here are some general tips I've come across after using a portable lab.

1. If you have issues mounting the drive. Check `Mount volume and removal able medium` under `Mount Options`.
2. Always be sure to use VeraCrypt to dismount your drives when finished.
3. You might use tools that get flagged as malicious. Putting them in a password protected zip file fixes this issue.
