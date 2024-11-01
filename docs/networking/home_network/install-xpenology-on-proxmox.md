# Install XPEnology on Proxmox

## Obtain the VM image

1. Go to [Arc Loader website](https://github.com/AuxXxilium/arc/releases/) and copy `arc-{version}.img.zip` link.

2. Go to proxmox console and run following commands

```sh
wget <arc_loader_download_address>

unzip </path/to/arc.img.zip>
```

## Create the VM

1. General

- Select your `VM name` and `ID`
- Select `Start at boot`

2. OS

- Select `Do not use any media`

3. System

- Leave default

4. Disks

- Delete the SCSI drive and any other disks

5. CPU

- Set minimum 2 cores

6. Memory

- Set minimum 4096 MB

7. Network

- Leave default unless you have special requirements (static, VLAN, etc)

Confirm and finish. Do \*NOT\*\* start the VM yet.

## Add the image to the VM

In the proxmox console run following commands

```sh
qm importdisk <VM_ID> </path/to/arc.img> <EFI location>
```

For example

```sh
qm importdisk 106 /home/user/arc.img local-lvm
```

## Configure XPEnology VM

Select your XPEnology VM and go to the `Hardware` tab. Select the `Unused Disk` and click the `Edit` button selecting `SATA` as `Bus/Device`.

![imgur.png](https://i.imgur.com/QFTHSm4.png)

Finally, we just need to set the `sata0` drive as the boot drive

![imgur.png](https://i.imgur.com/6wJRwJD.png)

## Configuring Arc

Time to boot the VM and configure Arc. You can use the `Console` tab on the UI to interact with it.

![imgur.png](https://i.imgur.com/axD9qxH.png)

On first boot, it will automatically boot into `Config Mode` which is exactly what we need

![imgur.png](https://i.imgur.com/80exdxX.png)

Let’s start by choosing the Synology model we want to impersonate, in this case I will choose `DS920+`, then choose the latest DSM version and DSM build and press `OK`

![imgur.png](https://i.imgur.com/4Xyhaap.png)

Select `No`.

![imgur.png](https://i.imgur.com/vCj1hkj.png)

Select `No - Install with random Serial/Mac`

Arc allows for a series of `add-ons` for things like removing limitations of DSM, or allow for extra hardware. I leave it default for now.

![imgur.png](https://i.imgur.com/SrL4ZYf.png)

We’re ready to build the loader - select `Yes`

![imgur.png](https://i.imgur.com/ALh9UR5.png)

After patching the DSM download, we get a final confirmation before rebooting

![imgur.png](https://i.imgur.com/i7Z1A5J.png)

Once rebooted, the default boot option will be `DSM mode`

![imgur.png](https://i.imgur.com/M3uD6OT.png)

## Configure HDD

### Adding a Storage Device to XPEnology VM in Proxmox

To add a storage device to your XPEnology VM in proxmox, follow these steps:

**Step 1: List available disks**

Open the proxmox console and run the following command to list all available disks in your system:

```sh
lsblk |awk 'NR==1{print $0" DEVICE-ID(S)"}NR>1{dev=$1;printf $0" ";system("find /dev/disk/by-id -lname \"*"dev"\" -printf \" %p\"");print "";}'|grep -v -E 'part|lvm'
```

This command outputs a list of disks along with their corresponding device IDs. The output will look something like this:

```sh
NAME                         MAJ:MIN RM   SIZE RO TYPE MOUNTPOINTS DEVICE-ID(S)
sda                            8:0    0   1.8T  0 disk   /dev/disk/by-id/wwn-0x50014ee26bd596b7 /dev/disk/by-id/ata-WDC_WD20EFPX-68C4TN0_WD-WX32D94J0KY1
nvme0n1                      259:0    0 238.5G  0 disk   /dev/disk/by-id/nvme-eui.e8238fa6bf530001001b448b4cbb7a08 /dev/disk/by-id/nvme-WD_PC_SN740_SDDPNQD-256G-1006_234222800596 /dev/disk/by-id/nvme-WD_PC_SN740_SDDPNQD-256G-1006_234222800596_1
```

**Step 2: Identify the disk to passthrough**

Locate the hard drive ID that you want to passthrough to your XPEnology VM. For example, if you want to passthrough the following disk `ata-WDC_WD20EFPX-68C4TN0_WD-WX32D94J0KY1`

**Step 3: Passthrough the disk to XPEnology VM**

```sh
qm set <VM_ID> -sata3 /dev/disk/by-id/ata-WDC_WD20EFPX-68C4TN0_WD-WX32D94J0KY1
```

Replace `<VM_ID>` with your VM’s ID.

**Step 4: Restart the VM**

Finally, restart the VM to apply the changes.

### Passthrough HDD Serial Number

Install lshw to verify the HDD serial:

```sh
apt install lshw
```

Execute following command:

```sh
lshw -C disk
```

Sample output:

```sh
*-disk
       description: ATA Disk
       product: WDC WD20EFPX-68C
       vendor: Western Digital
       physical id: 0.0.0
       bus info: scsi@0:0.0.0
       logical name: /dev/sda
       version: 0A81
       serial: WD-WX32D94J0KY1
       size: 1863GiB (2TB)
       configuration: ansiversion=5 logicalsectorsize=512 sectorsize=4096
  *-namespace:0
       description: NVMe disk
       physical id: 0
       logical name: hwmon1
  *-namespace:1
       description: NVMe disk
       physical id: 2
       logical name: /dev/ng0n1
  *-namespace:2
       description: NVMe disk
       physical id: 1
       bus info: nvme@0:1
       logical name: /dev/nvme0n1
       size: 238GiB (256GB)
       capabilities: gpt-1.00 partitioned partitioned:gpt
       configuration: guid=3cc91a1e-becf-49ff-8fd8-7366acbacc0e logicalsectorsize=512 sectorsize=512 wwid=eui.e8238fa6bf530001001b448b4cbb7a08
```

Open `/etc/pve/qemu-server/<VM_ID>.conf`, locate the `sata3` configuration, and add the serial:

```sh
sata3: /dev/disk/by-id/ata-WDC_WD20EFPX-68C4TN0_WD-WX32D94J0KY1,size=1953514584K,serial=WD-WX32D94J0KY1
```

## Note

You can assign static IP address for XPEnology VM in mikrotik using following script

```rsc
/ip dhcp-server lease
add address=192.168.0.6 mac-address=BC:24:11:BC:C0:26 server=dhcp-bridge-lan comment="NAS"
```

## References

- [https://pve.proxmox.com/wiki/Passthrough*Physical_Disk_to_Virtual_Machine*(VM)](<https://pve.proxmox.com/wiki/Passthrough_Physical_Disk_to_Virtual_Machine_(VM)>)
