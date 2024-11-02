# SSH into Proxmox

## Enable SSH Password Authentication in Proxmox

1. Set a root password

```sh
sudo passwd root
```

You will be prompted to enter and confirm a new password for the root user.

2. Enable SSH password authentication

Edit the SSH configuration file `/etc/ssh/sshd_config` to allow root login. Change the line `PermitRootLogin without-password` to `PermitRootLogin yes`

3. Restart the SSH service

```sh
systemctl restart sshd
```

## SSH into Proxmox LXC Container

```sh
lxc-attach --name <VM_ID>
```

Replace `<VM_ID>` with the actual ID of the container you wish to access. You will need to enter the root password you set earlier if prompted.
