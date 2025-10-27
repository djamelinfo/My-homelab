# Proxmox LXC CIFS Share Mount Configuration

## 1. In the LXC Container

### Step 1: Create Group `lxc_shares`
Run the following command to create the group `lxc_shares` with GID=10000 in the LXC. This will match the GID=110000 on the PVE host.
```bash
groupadd -g 10000 lxc_shares
```

### Step 2: Add User(s) to the Group
Add the user(s) who need access to the CIFS share to the group `lxc_shares`. For example, if you're configuring Jellyfin or Plex, replace `USERNAME` with the actual username of the application:
```bash
usermod -aG lxc_shares USERNAME
```

### Step 3: Shutdown the LXC
Shutdown the LXC to apply changes:
```bash
shutdown now
```

## 2. On the Proxmox Host (PVE)

### Step 1: Create a Mount Point
Create a mount point on the PVE host for the NAS CIFS share:
```bash
mkdir -p /mnt/lxc_shares/nas_rwx
```

### Step 2: Add NAS CIFS Share to `/etc/fstab`
Add the following entry to `/etc/fstab`. Adjust `//NAS/nas/` to match your CIFS hostname or IP and share name. Also, modify `user=smb_username,pass=smb_password` to match your credentials:
```bash
{ echo '' ; echo '# Mount CIFS share on demand with rwx permissions for use in LXCs (manually added)' ; echo '//NAS/nas/ /mnt/lxc_shares/nas_rwx cifs _netdev,x-systemd.automount,noatime,uid=100000,gid=110000,dir_mode=0770,file_mode=0770,user=smb_username,pass=smb_password 0 0' ; } | tee -a /etc/fstab
```

### Step 3: Mount the Share
Mount the share:
```bash
mount /mnt/lxc_shares/nas_rwx
```

### Step 4: Add Bind Mount in LXC Config
Add a bind mount for the share to the LXC configuration file. Replace `LXC_ID` with the LXC container's ID.

#### Read+Write+Execute (rwx) Permissions:
```bash
{ echo 'mp0: /mnt/lxc_shares/nas_rwx/,mp=/mnt/nas' ; } | tee -a /etc/pve/lxc/LXC_ID.conf
```

#### Read-Only (ro) Permissions:
```bash
{ echo 'mp0: /mnt/lxc_shares/nas_rwx/,mp=/mnt/nas,ro=1' ; } | tee -a /etc/pve/lxc/LXC_ID.conf
```

### Step 5: Start the LXC
Start the LXC container:
```bash
pct start LXC_ID
```
