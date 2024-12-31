# home-lab
Home Laboratory

# 45 Drives Home Lab 15

A project to build a home-lab server based on the 45 Drive HL15 chassis.

## Bill of Matrials

1. [45 Drives HL15](https://store.45homelab.com/configure/hl15)
   1. Black Faceplate
   2. Noctua Fans
2. [Supermicro H13SSL-NT](https://www.supermicro.com/en/products/motherboard/h13ssl-nt)
3. [AMD 9015](https://www.amd.com/en/products/processors/server/epyc/9005-series/amd-epyc-9015.html)
4. [Corsair WS DDR5 ECC RDIMM 64GB (4 x 16GB) DDR5 DRAM 6400MT/s CL32 Memory Kit × 1](https://www.corsair.com/ca/en/p/memory/cma64gx5m4b6400c32/ws-ddr5-ecc-rdimm-64gb-4-x-16gb-ddr5-dram-6400mt-s-cl32-memory-kit-cma64gx5m4b6400c32)
6. [WD Black SN850X](https://www.westerndigital.com/en-in/products/internal-drives/wd-black-sn850x-nvme-ssd?sku=WDS800T2XHE)
7. [MCIO x8 74 Pin ( NVMe) to 2 X Mini SAS SFF-8643 Cable - 50 CM](https://www.microsatacables.com/mcio-x8-74pin-nvme-to-2x-sff-8643-nvme-50-cm)
8. [JetKVM](https://www.kickstarter.com/projects/jetkvm/jetkvm)
9. [TrueNAS](https://www.truenas.com)
10. [Plex](https://www.plex.tv/en-ca/personal-media-server)


## Rationale

I have been using a Dell Precision T3500 Since 2009. First this was a workstation I used while working at Kodak,
then later using it at home, where I hadded a Sans Digital 8 bay RAID system for archival, file and media services.

*As an aside, I have been using RAID systems&mdash;hardware and software&mdash;for over ten years now, and have been
burned many times. These days I am convinced that
[Software Defined Storage](https://www.supermicro.com/en/glossary/software-defined-storage)
(SDS) is the way to go, in particular with something like TrueNAS and ZFS.*

This system also hosts my two printers, an HP LaserJest 1018 and a Kodak Hero 9.1. I cannot get these to run on
modern MacOS and Windows systems, but there is some home TrueNAS may have support for the HP printer. The Kodak printer is quite the beast,
and is increasingly unreliable, so that will be a write-off.

After all this time, it is still running Windows 10, where I cannot upgrade to Windows 11. Also, when rebooting,
I have to manually intervene because the system battery is low, which has become troublesome, and worrying.

After watching a lot of YouTube videos, and doing other research, I was sold on the 45 Drives products because so
many people testified to the quality. Initially I thought I would go with the HL15, but considered the
[45drives.com](https://45drives.com) AV15 with AMD motherboard. However, this turned out to be too pricy for my tastes,
especially for cusomization. Also, the Storinator people really dropped the ball on presales support.

Going back to [45homelab.com](https://45homelab.com) I decided to just build it myself, and the presales support has
been much better.

### Supermicro H13SSL-NT

While I looked at a lot of differnt system boards, ultimately I decided I wanted at server board, and it had to be an
ATX form factor for the HL15. I also decided to go with the latest AMD Epyc technology, starting with a low end 9015.
If I decided I want more power, I can always upgrade the CPU.

The H13SSL-NT has 12 DIMM slots, but I will only use 4 DIMMs to start with, and upgrade later when I have the money.

The H13SSL-NT has two M.2 slots, and I decided to go with the WD Black SN850X with 8 TB so the wear leveling has the
most chance to preserve reliable storage, while also being high performance. I may find some Optane memory for the
second M.2 for ZFS write log if I feel I need better write performance.

The H13SSL-NT uses MCIO connectors for SATA, but unfortunately, 45 Drive do not stock cables for this.

While the H13SSL-NT supports remote management, I decided to get a JetKVM just in case... In particular, for first
configuration, you need to get in to the console to configure the remote management first. Also, if for any reason
the remote management is not working, the KVM is the most basic way to manage the server board.

### Software

My plan is to use TrueNAS and ZFS because my primary use will be as a media server, using Plex running under Docker.
Also, I will be serving both Apple File Service and Windows File Service (samba).

As a seasoned computer scientist and software engineer, who started programming in 1970, I like to do what I call
'recreational programming.' Now that I am retired, I would like to continue to play with technology, but not in a
mission critical way. TrueNAS has recently given up on Kubernetes, relying on Docker instead, and this makes me happy,
as there is little I find 'recreational' about Kubernetes.

### ZFS

One of the nicest new features of TrueNAS Electric Eel is that you can now add drives to a ZFS vdev, one drive at
a time, and the system will resilver it over time. This means I can start with fewer than 15 drives, and add more
when I need more storage.

## Details

If you have any questions or comments, please use GitHub Issues, and I will try to update this documentation.

### RAID

The main problem with RAID solutions is that you get tied to proprietry implementation.

Certainly this was the case with my Sans Digital hardware RAID controller. Recently I tried adding brand new
Western Digital Red 8 TB drives, but the RAID controller failed the drives shortly after accepting them, for no
obvious reason. Western Digital was happy to RMA the drive, but even the replacement failed.

When I contacted Sans Digital, after a while, looking at the RAID logs, he concluded that the WD Red drives are
not suitable for their controller because they are NAS drives, not Enterprise drives. While I had been using the
Red drives for many years, the newer 8 TB drives had more latency, so the Sans Digital controller just failed them
without explanation. ***Sans Digital have no empathy or compassion for their customers, and clearly their software
developer are morons.***

I leaned the hard way that RAID-5 is not good enough, where after relacing a failed drive, if you get another failed
drive while rebuilding the array, the whole array is now a brick.

Even with RAID-6 I have been burned, where I had multiple drives fail, and the array became a brick during rebuilding.

Now I am planning on using RAID-Z with three parity drives because I am that paranoid.

On my Dell T3500 I run RAID-0 from the system board so I can boot from it. Initially I used 15000 RPM disks, without
problems, but then replaced these with SSDs, and still have had no problems. Still, I live in fear, even though this
is backed up daily. This is something I expect to fail, but I am okay with losing data that is younger than 24 hours
old.

### HL15

The largest RAID system I have ever run is eight disks, so I am hoping to evenutally run with fifteen or more drives.
While I could probably squeeze one more disk in the HL15, because the H13SSL-NT supports that, it is probably more
trouble than it's worth.

### H13SSL-NT

The H13SSL-N has two 1 Gigabit ethernet ports while the H13SSL-NT has two 10 Gigabit ports, and that is the only difference.

With 12 RDIMM slots, this board can handle up to 4.5 Terrabytes of RAM, but I will probably not go above 192 Gigabytes as
larger DIMMs are slower and have more letency.

[Supermicro Intelligent Management](https://www.supermicro.com/en/solutions/management-software/bmc-resources) offers
state-of-the-art management via [IPMI](https://en.wikipedia.org/wiki/Intelligent_Platform_Management_Interface) 2.0.
Before you can use it, you have to configure it in the UEFI, such as the IP address, then after that, via the 1 GB
ethernet port, you can access it from any web browser.

This is handy because I want to run my NAS headless. However, the JetKVM will be my backup, where I can access the VGA
and USB ports on the H13SSL-NT if there are any problems with IPMI. I need a VGA to mini-HDMI cable, and a USB-A to USB-C
cable.
