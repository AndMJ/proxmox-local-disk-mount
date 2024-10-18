
# Mount Disk Shared Between LXCs 
(markdown made with the help of ChatGPT)

## Instructions

### 1. Create Disk Partition
Run the following command:
```bash
cfdisk /dev/sdb
```
Follow the prompts to create a partition (e.g., `/dev/sdb1`).

### 2. Format the Partition with ext4
Format the partition:
```bash
mkfs.ext4 /dev/sdb1
```

### 3. Create Mounting Point on Host
Create a directory for mounting, e.g., `/mnt/pve/myfolder`:
```bash
mkdir /mnt/pve/media
```
You can also create additional folders as needed, like `/Downloads`, `/Movies`, etc.

### 4. Mount the Disk
Mount the partition:
```bash
mount /dev/sdb1 /mnt/pve/media
```
To make this mount permanent across reboots, add it to `/etc/fstab`:
```bash
echo '/dev/sdb1 /mnt/mydata ext4 defaults 0 2' >> /etc/fstab
```

### 5. Verify Read/Write Permissions
Check if the disk is mounted with "rw" (read-write) and not "ro" (read-only):
```bash
lsblk -o NAME,RO,MOUNTPOINT
```
If you see a `1` in the "RO" (read-only) column next to your disk (e.g., `/dev/sdb1`), the disk is read-only.

- If `1`, remount with RW.
- If `0`, continue.

### 6. Change Mounted Directory Owner
Change the owner of the mounted directory:
```bash
chown -R nobody:nogroup /mnt/pve/media
```

### 7. Configure LXC Containers

#### Unprivileged LXC Configuration
Edit each unprivileged LXC configuration using:
```bash
nano /etc/pve/lxc/xxx.conf
```
Add the following lines:
```conf
mp0: /mnt/pve/media,mp=/mnt/media
lxc.idmap: u 0 100000 65536
lxc.idmap: g 0 100000 65536
```

#### Privileged LXC Configuration (e.g., Emby, Jellyfin, Plex)
For privileged LXC containers, add:
```conf
mp0: /mnt/pve/media,mp=/mnt/media
```

### 8. Adjust Folder Permissions for LXC Users
On the host, change the mounted folder's owner for each LXC user (e.g., emby/jellyfin/plex):
1. Check the user ID:
   ```bash
   id <user>
   ```
   Example: `id emby`

2. If `ACL` is not installed, install it:
   ```bash
   apt-get install acl
   ```

3. Set ACL permissions:
   ```bash
   setfacl -R -m u:root:rwx /mnt/pve/media
   setfacl -R -m u:999:rwx /mnt/pve/media  # emby
   setfacl -R -m u:nobody:rwx /mnt/pve/media
   setfacl -d -m u:root:rwx /mnt/pve/media
   setfacl -d -m u:999:rwx /mnt/pve/media  # emby
   setfacl -d -m u:nobody:rwx /mnt/pve/media
   ```

### 9. Set Default ACL for Child Folders
Set permissions for each child folder, e.g., `/Downloads`, `/Movies`, `...`:
```bash
# For Downloads folder
setfacl -d -m u:root:rwx /mnt/pve/media/Downloads
setfacl -d -m u:999:rwx /mnt/pve/media/Downloads  # emby
setfacl -d -m u:nobody:rwx /mnt/pve/media/Downloads

# For Movies folder
setfacl -d -m u:root:rwx /mnt/pve/media/Movies
setfacl -d -m u:999:rwx /mnt/pve/media/Movies  # emby
setfacl -d -m u:nobody:rwx /mnt/pve/media/Movies

# For Shows folder
...
```

Repeat the above steps for other folders as needed.
