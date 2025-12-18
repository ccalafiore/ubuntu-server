

# Add a New Disk to an Existing RAID 1 

## Identify the device names
Identify the device names and paths of the new drive (e.g., `sdc` and
`/dev/sdc`) and the existing RAID 1 (e.g., `md0` and `/dev/md0`) using `lsblk`. You
can run the command without options for minimal device info.

You can use the option `-f` or `--fs` like below to output some filesystem
device info.
```
lsblf -f
```
You can use the option `-o <list>` or `--output <list>` to select the output
device info, by replacing `<list>` with the selected info names separated by
commas. For example,
```
lsblk -o name,path,label,type,fstype,fsver,size,fssize,fsused,fsuse%,fsavail,mountpoints
```

You can also identify the name of the existing RAIDs with:

```
cat /proc/mdstat
```

Expect that the name of a phisical drive starts with either `sd` (sata disk) or
`nvme` (Non-Volatile Memory Express). It starts with `sd` if it is phisically
connected to the motherboard via a SATA port. However, it starts with `nvme` if
it is connected via a PCIe port.

In my case, the name and path of the new device are `sde` and `/dev/sde`, in turn. On
the other hand, the name and the path of the existing RAID 1 are `md0` and `/dev/md0`.


