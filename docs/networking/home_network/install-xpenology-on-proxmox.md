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
