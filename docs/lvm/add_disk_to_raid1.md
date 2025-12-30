

# Add a New Disk to an Existing RAID 1 

## Identify the device names
Identify the device names and paths of the new drive (e.g., `nvme2n1` and
`/dev/nvme2n1`) and the existing RAID 1 (e.g., `md0` and `/dev/md0`) using `lsblk`.
You can run the command without options for minimal device info.

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

In my case, the name and path of the new device are `nvme2n1` and `/dev/nvme2n1`, in turn. On
the other hand, the name and the path of the existing RAID 1 are `md0` and `/dev/md0`.


## Create in the New Disk the same Partitions as the old Disks of the RAID  

We need to create in the new disk (e.g., /dev/nvme2n1) the same two partitions of
the old disks (e.g., /dev/nvme0n1 and /dev/nvme1n1) that are already the members of the existing
RAID. They should have the minimum sizes and the same types of the partitions in the old
disks.

### Get the Partition Sizes and Types of the Old Disks
Get the sizes (in sectors) and types of the two partitions in the old disks with:
```
sudo parted <old_disk_path> unit s p
```
by replacing the <old_disk_path> with the device path of one of the old disks
(e.g., /dev/nvme0n1 or /dev/nvme1n1).

In the example below, the output has a table with two rows, one for each
partition. You can find the partition sizes and types in the columns "Size" and
"Name", in turn. In this case, we will need to create two partitions in the new
disk. The first partition needs to be with partition type "EFI system partition"
and size "2201600" sectors. The second partition needs to have a type "Linux
filesystem" and a size minimum "486191104" sectors, i.e. the
minimum of "486191104" and "490000000").

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
disk, you can create them interactively with `gdisk`:

- Start the iteractive partition creation with `sudo gdisk <new_disk_path>` by
replacing <disk_path> with the device path of the new disk. In my case, the
device path of the new disk is /dev/nvme2n1.

You can type `d` and press the keybourd "Enter", to delete any unwanted existing partitions.

Then, you can type `n` and "Enter" to create the EFI partion.

Input the desired "First sector" and "Last sector". You can use the default first sector by leaving blank and press "Enter".

you can choose the `+2201600` and ente

the partion "Hex code" 
