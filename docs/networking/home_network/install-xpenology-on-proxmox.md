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

Confirm and finish. Do *NOT** start the VM yet.

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

Select `No` to install addons.

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

## Installing DSM
