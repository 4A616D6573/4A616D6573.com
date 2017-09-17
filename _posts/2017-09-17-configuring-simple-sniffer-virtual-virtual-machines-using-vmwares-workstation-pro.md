---
layout: post
title: Configuring Simple Victim and Sniffer Virtual Machines Using VMware's Workstation Pro
---

## Introduction

This post will show you how to create a victim and sniffer virtual machine pair which you can use to analyse malware or general application network traffic. This post was mainly inspired by [Malware Unicorn's Reverse Engineering 101](https://securedorg.github.io/RE101/) which provides one of the best introductions to malware reverse engineering I've read.

I did have a few issues importing and configuring the Virtual Box images into VMware's Workstation Pro, so I deployed the configuration from scratch using Workstation Pro as the base. I also had trouble finding a central guide on how to configure INetSim and Wireshark, so I figured I'd make a post not only as a reference for myself but to aid other in configuring this setup.

---

## Required Components

Below are the recommended components:

- [VMware Workstation Pro](https://www.vmware.com/au/products/workstation.html)
- Ubuntu 16.4 Virtual Machine
- Windows 7 Virtual Machine (Windows XP or Windows 10 can also be used)
- [INetSim](http://www.inetsim.org/)
- [Wireshark](https://www.wireshark.org/)

---

## Virtual Machine Setup

Firstly deploy your virtual machines and name appropriately, I currently use Workstations [Snapshots](https://www.vmware.com/support/ws4/doc/preserve_snapshot_ws.html) to preserve various virtual machine configurations, this saves me from having multiple deployments of the same OS and also reduces disk space. Make sure you at least have a "Vanilla" snapshot before beginning incase anything goes wrong. I recommend that both virtual machines have at least 2GB of RAM and 2 Processors, this can be changed under `Edit virtual machine settings` by changing `Memory` and `Processors`.

### Virtual Network Configuration

Next you will need to configure a network for your victim and sniffer  to run in.

0. Open the Virtual Network Editor in VMware Workstation by selecting `Edit` and `Virtual Network Editor`.
0. Select `Change Settings` and enter your credentials if required.
0. Select `Add Network`.
0. Choose a network to add, I usually select the last available network `VMnet19`.
0. Once the network has been configured confirm that the network is using `Host-only`.

You can change the subnet to suit your needs but for this post I will leave it as the 'default' configuration which is `192.168.150.0`.

**Note:** Do not assign this network to your victim and sniffer machines until you have configured INetSim and Wireshark.

---

## Ubuntu Configuration

Once you have a "Vanilla" snapshot you can launch Ubuntu, this will be used as the Sniffer and will require INetSim and Wireshark to be installed.

### INetSim

INetSim is a tool that simulates common internet services in a lab environment and is mainly used to analyse network behavior, this provides additional benefits such as allowing execution in an isolated environment while allowing malware samples to make "real" communications to the internet that can lead to retrieval of important indicators.

Setting up INetSim isn't too difficult but some of the information to set it up correctly is spread out over multiple sites.

0. Launch `Terminal`.
0. Enter `sudo -i` and type in your virtual machines password to launch as root and make this process a little quicker.
0. Enter `echo "deb http://www.inetsim.org/debian/ binary/" > /etc/apt/sources.list.d/inetsim.list` to add the Debian Archive repository for INetSim to your sources list.
0. Enter `wget -O - http://www.inetsim.org/inetsim-archive-signing-key.asc | apt-key add -` to add the Signing Key to your trusted keys.
0. Enter `apt update` to issue an update to your cache of available packages.
0. Enter `apt install inetsim -y` to install INetSim.

Once this is done you have installed INetSim, however you will need to do some more configuration to get it operational.

0. Enter `nano /etc/default/inetsim` and set `ENABLED` from `0` to `1`, use <kbd>CTRL</kbd> + <kbd>x</kbd> to exit and type `y or yes` to save your changes. This will set INetSim to launch on boot.
0. Enter `nano /etc/inetsim/inetsim.conf` to open the configuration file.
0. Set `service_bind_address` to `XXX.XXX.XXX.1` of your subnet, in this case `192.168.150.1` and delete the `#`, this will act as the internet gateway for your victim virtual machine.
0. Set `dns_default_ip` to the same address as your gateway, remember to delete the `#` and save the configuration.
0. Start INetSim by entering `service inetsim start`.
0. Check that INetSim is running `service inetsim status`.
0. Ensure the correct services are running by entering `ps -ef | grep inetsim`. You should see a list of services such as `inetsim 4404 4394 0 17:59 ? 00:00:00 inetsim_ftp_21_tcp`.

You have now setup INetSim.

## Wireshark

Wireshark is an amazing tool for capturing network traffic that can then be analysed to determine where malware is communicating, it can also be used to troubleshoot a broad variety of network issues.

Yet again Wireshark is "easy" to install but the instructions are outlined here for the sake of completion.

0. Launch `Terminal`.
0. Enter `sudo -i` and type in your virtual machines password.
0. Enter `sudo add-apt-repository ppa:wireshark-dev/stable -y` to add the repository.
0. Enter `sudo apt-get update` to update your available packages.
0. Enter `sudo apt-get install wireshark -y` to install Wireshark.
0. When prompted with the `configuring wireshark-common` window select `Yes`, this will allow non-super users to capture packets.
0. Exit the root account by entering `exit`, this is so you assigned the correct user to the wireshark group.
0. Enter `sudo adduser $USER wireshark`, this will add the terminal user to the wireshark group.
0. Test that you can launch Wireshark by entering `sudo wireshark`, you will see an error message relating to running the application as a super-user but this shouldn't effect functionality.

---

## Final Configurations

Once you have configured your virtual machines, custom network, installed and configured INetSim and Wireshark you will need to complete a few final steps.

### Virtual Machine Virtual Network

Set your victim and sniffer to the network you configured. This can be done by performing the following on each virtual machine.

0. Select `VM`.
0. Select `Settings`.
0. Select `Network Adapter`
0. Check `Custom: Specific virtual network` under `Network connection` and choose your network, in this case it's `VMnet19`.

### Set Ubuntu Static IP Address

Since your Ubuntu machine is acting as your internet gateway you'll want to give it a static IP.

0. Launch `Network`.
0. Select `Options` for the `Wired` connection.
0. Select `IPv4 Settings`.
0. Under `Method` select `Manual`.
0. Select `Add` and enter your address from before in this case `192.168.150.1`, use <kdb>Tab</kbd> to switch to the `Netmask`, leave this as default <kdb>Tab</kbd> again to switch to `Gateway` then entre the same number as before.
0. Select `Save`.

Disable and re-enable the network interface to update to your changes, confirm you address is correct by using `ifconfig` and checking the IP under `ens33`.

### Set Windows Domain Name System

Once you have configured your Ubuntu machine to have a static IP you need to configure your Windows machines DNS so it knows to use the Ubuntu machine as it's reference to the 'internet'.

0. Open the `Start Menu` and select `Control Panel`.
0. Select `Network and Internet`.
0. Select `Network and Sharing Center`.
0. Select `Change adapter settings`.
0. Right click `Local Area Connection` and select `Properties`.
0. Select `Internet Protocol Version 4 (TCP/IPv4)` and select `Properties`.
0. Check `Use the following DNS Server Addresses` and enter the static IP of your Ubuntu machine in `Preferred DNS server` in this case `192.168.150.1`.
0. Select `OK`

Alternatively this can also be achieved by entering the following in an elevated Command Prompt `netsh interface ip add dns name="Local Area Connection" addr=192.168.150.1 index=1`.

Once this is done you can move onto testing.

### Testing

Finally to test this setup.

0. Launch both virtual machines, usually I launch the Ubuntu machine first and allow it to load before launching the victim.
0. Once Ubuntu has loaded check that INetSim is running correctly with `ps -ef | grep inetsim`.
0. Launch `Wireshark`.
0. Select the interface `ens33` and start capturing by selecting `Capture`, `Start`.
0. In the Windows 7 machine ping the Ubuntu machine, if everything is configured correctly you should see 'ICMP' packets in the Wireshark capture trail.
0. Open Internet Explorer and browse to `www.google.com`, the page should load and return the following in HTML 'This is the default HTML page for INetSim HTTP server fake mode.'
0. Confirm you can download fake files, enter a URL with an .exe included such as `www.evil.com/malware.exe`. You should be promoted with a download window, select `Save` and confirm that malware.exe is download.
0. Confirm you can open the downloaded executable, you should see the message prompt `This is the INetSim default binary`.

You now have a fully functioning basic sniffer setup, just remember to snapshot your finished configurations so you can restore them after use.

---

## Closing

You're now free to detonate malware samples with relative safety while having the added benefit of simulating internet connectivity. I'm thinking the next post will be about what to deploy to a victim or malware analysis machine as well as what you should run and capture while analysing samples.

I hope you found this post informative. If you have any questions, noticed inaccurate information or spelling mistakes in this post you can contact me via the [About](/about/) section.

---

## Tips

0. If you're using this setup from multiple machines, you will need to configure the same VMware Workstation network settings for each machines instance of VMware Workstation.
0. If you receive the message "Could not get lock /var/lib/dpkg/lock" or "E: Could not get lock /var/cache/apt/archives/lock" while trying to install an application you can clear it but deleting the "lock" folder using the following: `rm /var/lib/dpkg/lock -r` or `rm /var/cache/apt/archives/lock -r`.
0. If you want to remove the super-user warning message when opening Wireshark, you can do so by editing Wireshark's init.lua file by entering `sudo nano /usr/share/wireshark/init.lua` and setting `disable_lua` from `false` to `true`.

---

## References

0. [https://securedorg.github.io/RE101/]("https://securedorg.github.io/RE101/)
0. [https://www.vmware.com/au/products/workstation.html](https://www.vmware.com/au/products/workstation.html)
0. [http://www.inetsim.org/packages.html](https://www.vmware.com/au/products/workstation.html)
0. [https://www.wireshark.org/](https://www.wireshark.org/)
0. [https://www.vmware.com/support/ws4/doc/preserve_snapshot_ws.html](https://www.vmware.com/support/ws4/doc/preserve_snapshot_ws.html)
0. [https://techanarchy.net/2013/08/installing-and-configuring-inetsim/](https://techanarchy.net/2013/08/installing-and-configuring-inetsim/)
0. [https://askubuntu.com/questions/700712/how-to-install-wireshark](https://askubuntu.com/questions/700712/how-to-install-wireshark)
0. [https://askubuntu.com/questions/454734/running-wireshark-lua-error-during-loading](https://askubuntu.com/questions/454734/running-wireshark-lua-error-during-loading)
0. [https://superuser.com/questions/204046/how-can-i-set-my-dns-settings-using-the-command-prompt-or-ps](https://superuser.com/questions/204046/how-can-i-set-my-dns-settings-using-the-command-prompt-or-ps)

---
