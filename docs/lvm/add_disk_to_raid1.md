

# Add a New Disk to an Existing RAID 1 

## Identify the device names
Identify the device names and paths of the new drive (e.g., `nvme2n1` and
`/dev/nvme2n1`) and the existing RAID 1 (e.g., `md0` and `/dev/md0`) using
`lsblk`. You can run the command without options for minimal device info.

You can use the option `-f` or `--fs` like below to output some filesystem
device info.
```
lsblk -f
```
You can use the option `-o <list>` or `--output <list>` to select the output
device info, by replacing `<list>` with the selected info names separated by
commas. For example,
```
lsblk -o model,name,path,type,partn,fstype,parttypename,pttype,size,fssize,fsuse%,fsused,fsavail,mountpoints
```

You can also identify the name of the existing RAIDs with:
```
cat /proc/mdstat
```

Expect that the name of a phisical drive starts with either `sd` (sata disk) or
`nvme` (Non-Volatile Memory Express). It starts with `sd` if it is phisically
connected to the motherboard via a SATA port. However, it starts with `nvme` if
it is connected via a PCIe port.

In my case, the name and path of the new device are `nvme2n1` and
`/dev/nvme2n1`, in turn. On the other hand, the name and the path of the
existing RAID 1 are `md0` and `/dev/md0`.


## Create in the New Disk the same Partitions as the old Disks of the RAID  

We need to create in the new disk (e.g., `/dev/nvme2n1`) the same two partitions
of the old disks (e.g., `/dev/nvme0n1` and `/dev/nvme1n1`) that are already the
members of the existing RAID. They should have the minimum sizes and the same
types of the partitions in the old disks.

### Get the Partition Sizes and Types of the Old Disks
Get the sizes (in sectors) and types of the two partitions in the old disks
with:
```
sudo parted <old_disk_path> unit s p
```
or
```
sudo gdisk -l <old_disk_path>
```
or
```
sudo fdisk -l <old_disk_path>
```
by replacing the <old_disk_path> with the device path of one of the old disks
(e.g., `/dev/nvme0n1` or `/dev/nvme1n1`).

In the example below, the output has a table with two rows, one for each
partition. You can find the partition sizes and types in the columns "Size" and
"Name", in turn. In this case, we will need to create two partitions in the new
disk. The first partition needs to be with partition type "EFI system partition"
and size of 2201600 sectors. The second partition needs to have a type "Linux
filesystem" and a size of minimum 486191104 sectors, i.e. the minimum of
486191104 and 490000000).

```
cc@cdw:~$ sudo parted /dev/nvme0n1 unit s p

Number  Start     End         Size        File system  Name                  Flags
 1      2048s     2203647s    2201600s    fat32        EFI system partition  boot, esp
 2      2203648s  488394751s  486191104s               Linux filesystem

cc@cdw:~$ sudo parted /dev/nvme1n1 unit s p

Number  Start     End         Size        File system  Name                  Flags
 1      2048s     2203647s    2201600s                 EFI system partition  boot, esp
 2      2203648s  492203647s  490000000s               Linux filesystem
```

### Create the Partitions in the New Disk

Once you know the sizes and types of the partitions to be created in the new
disk, you can create them interactively with `gdisk`.

- Start the iteractive partition creation with `sudo gdisk <new_disk_path>` by
replacing `<disk_path>` with the device path of the new disk. In my case, the
device path of the new disk is `/dev/nvme2n1`.

- If needed, delete unwanted existing partitions. Type `d`, press the key
"Enter", type the partition ID (or partition number) and press "Enter".

- Create the new EFI partition by typing `n` and press "Enter".

- Type the partition number and press "Enter". Use the Default value by leaving
it blank and press "Enter".

- Input the desired "First sector" and "Enter. To use the default value, leave
it blank and press "Enter".

- Input `+<size>` for the "Last sector", by replacing `<size>` with the desired
partition size. In our example, the size was 2201600 sectors, so the last sector
input would be `+2201600`.

- Input the partion type "Hex code" and press "Enter". The hex code for "EFI
system partition" is `ef00`. If you do not know the hex code for "EFI system
partition", you can type `l`, press "Enter", input "EFI system partition" and
"Enter" to get the hex code. Now, input the hex code and "Enter".

- Create the new "Linux filesystem" partition by typing `n` and press "Enter".

- Type the partition number and press "Enter". Use the Default value by leaving
it blank and press "Enter".

- Input the desired "First sector" and "Enter. To use the default value, leave
it blank and press "Enter".

- Input `+<size>` for the "Last sector", by replacing `<size>` with the desired
partition size. In our example, the size was 486191104 sectors, so input would
be `+486191104`.

- Input the partion type "Hex code" and press "Enter". The hex code for "Linux
filesystem" partition is `8300`. If you do not know the hex code for "Linux
filesystem", you can type `l`, press "Enter", input "Linux filesystem" and
"Enter" to get the hex code. Now, input the hex code and "Enter".



## Add the "Linux filesystem" partition of the new disk to the RAID

Add the second partion of the new disk to the existing RAID with:
```
sudo mdadm --add /dev/md0 /dev/nvme2n1p2
```

Check the status of the RAID with:
```
cat /proc/mdstat
```
or
```
sudo mdadm --detail /dev/md0
```


# Finish the EFI partition

Format the EFI partition of the new disk with:
```
sudo mkfs.vfat -F32 /dev/nvme2n1p1
```

Mount the EFI partition of the new disk with:
```
sudo mkdir -p /mnt/efi_new
sudo mount /dev/nvme2n1p1 /mnt/efi_new
```

If not mounted, mount the EFI partition of one of the old disks with:
```
sudo mount /dev/nvme0n1p1 /boot/efi
```

Copy all contents in /boot/efi to /mnt/efi_new
```
sudo rsync -av --progress --stats /boot/efi/ /mnt/efi_new/
```

Maybe, do the below after completion of the RAID build. This seems to do more
bad thn good. So, maybe skip it for now.
```
sudo grub-install /dev/nvme2n1p1
```

Unmount new and old efi partitions?

```
sudo umount /boot/efi
sudo umount /mnt/efi_new
```
or
```
sudo umount /dev/nvme0n1p1
sudo umount /dev/nvme2n1p1
```

Remove or comment the line of `/etc/fstab` mounting any EFI partition as root with `nano`.
```
sudo nano /etc/fstab
```


## Resize the RAID and partitions

Once RAID sync has finished, resize the RAID with:
```
sudo mdadm --grow /dev/md0 --size=max
```
Resize the unformated RAID partition with `fdisk`:
```
sudo fdisk /dev/md0p2
```

You may need to delete a swap partition. Follow the steps:
- remove the swap line in /etc/fstab as root.
- run `sudo swapoff /dev/mapper/vg0-lv--1`
- delete the swap partition with `fdisk` or `gdisk`.


Resize logical volume with in the RAID or RAID partition with:
```
sudo lvextend -l +100%FREE /dev/mapper/vg0-lv--0
```
Maybe, this to resize the ext4 file system:
```
sudo resize2fs /dev/mapper/vg0-lv--0
```

You may recreate the swap partition with:
```
todo
```




Maybe, also this:
```
sudo update-initramfs -u
```