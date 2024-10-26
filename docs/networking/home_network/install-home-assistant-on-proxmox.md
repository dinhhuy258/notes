# Install Home Assistant on Proxmox

## Obtain the VM image

1. Go to [Home Assistant website]( https://www.home-assistant.io/installation/alternative) and copy `KVM/Proxmox` link.

![imgur.png](https://i.imgur.com/t9shvXA.png)

2. Go to proxmox console and run following commands

```sh
wget <ha_download_address>

unxz </path/to/file.qcow2.xz>
```

## Create the VM

1. General

- Select your `VM name` and `ID`
- Select `Start at boot`

2. OS

- Select `Do not use any media`

3. System

- Change `Machine` to `q35`
- Change `BIOS` to `OVMF (UEFI)`
- Select the `EFI storage` (typically `local-lvm`)
- Uncheck `Pre-Enroll keys`

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
qm importdisk <VM_ID> </path/to/file.qcow2> <EFI location>
```

For example

```sh
qm importdisk 105 /home/user/haos_ova-12.0.qcow2 local-lvm
```

## Configure HA VM

1. Select your HA VM

2. Go to the `Hardware` tab

- Select the `Unused Disk` and click the `Edit` button
- Check the `Discard` box if you're using an SSD then click `Add`
- Select the `Options` tab
- Select `Boot Order` and hit `Edit`
- Check the newly created drive (likely `scsi0`) and uncheck everything else

## Finish Up

1. Start the VM
2. Check the shell of the VM. If it booted up correctly, you should be greeted with the link to access the Web UI.

![imgur.png](https://i.imgur.com/IPirqcA.png)


