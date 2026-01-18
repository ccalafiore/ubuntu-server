
# Reduce Phisical Volume Size




You need free space at the end of the physical volume, by moving all PEs at the begining.

Show the physical extents (PEs) in the physical volume with:
```
sudo pvs --segments -v /dev/md0p2
```

Move the PEs at the begining of the phisical volume with 



Reduce the size of the physical volume to 100 GiB with:
```
sudo pvresize --setphysicalvolumesize 100G /dev/md0p2
```










```
ubuntu@ubuntu:~$ sudo pvs
  PV           VG  Fmt  Attr PSize   PFree  
  /dev/md127p2 vg0 lvm2 a--  928.33g 862.33g
ubuntu@ubuntu:~$ sudo vgs
  VG  #PV #LV #SN Attr   VSize   VFree  
  vg0   1   2   0 wz--n- 928.33g 862.33g
ubuntu@ubuntu:~$ sudo lvs
  LV   VG  Attr       LSize  Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  lv-0 vg0 -wi-ao---- 50.00g                                                    
  lv-1 vg0 -wi-a----- 16.00g
```


```
ubuntu@ubuntu:~$ sudo pvresize --setphysicalvolumesize 100G /dev/md127p2
/dev/md127p2: Requested size 100.00 GiB is less than real size 928.33 GiB. Proceed?  [y/n]: y
  WARNING: /dev/md127p2: Pretending size is 209715200 not 1946857472 sectors.
  /dev/md127p2: cannot resize to 25599 extents as later ones are allocated.
  0 physical volume(s) resized or updated / 1 physical volume(s) not resized
```


```
ubuntu@ubuntu:~$ sudo pvs -v --segments /dev/md127p2
  PV           VG  Fmt  Attr PSize   PFree   Start SSize  LV   Start Type   PE Ranges               
  /dev/md127p2 vg0 lvm2 a--  928.33g 862.33g     0  58804          0 free                           
  /dev/md127p2 vg0 lvm2 a--  928.33g 862.33g 58804  12800 lv-0     0 linear /dev/md127p2:58804-71603
  /dev/md127p2 vg0 lvm2 a--  928.33g 862.33g 71604   4096 lv-1     0 linear /dev/md127p2:71604-75699
  /dev/md127p2 vg0 lvm2 a--  928.33g 862.33g 75700 161953          0 free                           
```

```
ubuntu@ubuntu:~$ sudo pvmove --alloc anywhere /dev/md127p2:58804-71603 /dev/md127p2:0-12799
  /dev/md127p2: Moved: 0.16%
  /dev/md127p2: Moved: 60.51%
  /dev/md127p2: Moved: 100.00%
```



```
ubuntu@ubuntu:~$ sudo pvs -v --segments /dev/md127p2
  PV           VG  Fmt  Attr PSize   PFree   Start SSize  LV   Start Type   PE Ranges               
  /dev/md127p2 vg0 lvm2 a--  928.33g 862.33g     0  12800 lv-0     0 linear /dev/md127p2:0-12799    
  /dev/md127p2 vg0 lvm2 a--  928.33g 862.33g 12800  58804          0 free                           
  /dev/md127p2 vg0 lvm2 a--  928.33g 862.33g 71604   4096 lv-1     0 linear /dev/md127p2:71604-75699
  /dev/md127p2 vg0 lvm2 a--  928.33g 862.33g 75700 161953          0 free                           
```

```
ubuntu@ubuntu:~$ sudo pvmove --alloc anywhere /dev/md127p2:71604-75699 /dev/md127p2:12800+4096
  /dev/md127p2: Moved: 0.44%
  /dev/md127p2: Moved: 100.00%
```

Check again the PEs with:
```
ubuntu@ubuntu:~$ sudo pvs -v --segments /dev/md127p2
  PV           VG  Fmt  Attr PSize   PFree   Start SSize  LV   Start Type   PE Ranges               
  /dev/md127p2 vg0 lvm2 a--  928.33g 862.33g     0  12800 lv-0     0 linear /dev/md127p2:0-12799    
  /dev/md127p2 vg0 lvm2 a--  928.33g 862.33g 12800   4096 lv-1     0 linear /dev/md127p2:12800-16895
  /dev/md127p2 vg0 lvm2 a--  928.33g 862.33g 16896 220757          0 free
```

as you can see above, all the free space has been successfully left at the end of the physical volume. 

Now, you can re-try to reduce the size of the physical volume with:
```
ubuntu@ubuntu:~$ sudo pvresize --setphysicalvolumesize 100G /dev/md127p2
/dev/md127p2: Requested size 100.00 GiB is less than real size 928.33 GiB. Proceed?  [y/n]: y
  WARNING: /dev/md127p2: Pretending size is 209715200 not 1946857472 sectors.
  Physical volume "/dev/md127p2" changed
  1 physical volume(s) resized or updated / 0 physical volume(s) not resized
```
The above says that the physical volume has been successfully reduced to 100 GiB.



```
ubuntu@ubuntu:~$ sudo pvs
  PV           VG  Fmt  Attr PSize    PFree  
  /dev/md127p2 vg0 lvm2 a--  <100.00g <34.00g

ubuntu@ubuntu:~$ sudo vgs
  VG  #PV #LV #SN Attr   VSize    VFree  
  vg0   1   2   0 wz--n- <100.00g <34.00g

ubuntu@ubuntu:~$ sudo lvs
  LV   VG  Attr       LSize  Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  lv-0 vg0 -wi-a----- 50.00g                                                    
  lv-1 vg0 -wi-a----- 16.00g                                                    

ubuntu@ubuntu:~$ sudo pvs -v --segments /dev/md127p2
  PV           VG  Fmt  Attr PSize    PFree   Start SSize LV   Start Type   PE Ranges               
  /dev/md127p2 vg0 lvm2 a--  <100.00g <34.00g     0 12800 lv-0     0 linear /dev/md127p2:0-12799    
  /dev/md127p2 vg0 lvm2 a--  <100.00g <34.00g 12800  4096 lv-1     0 linear /dev/md127p2:12800-16895
  /dev/md127p2 vg0 lvm2 a--  <100.00g <34.00g 16896  8703          0 free
```



