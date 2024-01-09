
LVM (Logical Volume Management) is the recommended way to manage disk or storage on Linux systems, especially for servers. One of the main advantages of LVM partition is that we can extend its size online without any downtime. 

For the demo purpose, I have attached a 32GB USB pen drive to my Kali system, I will create an LVM partition on this disk from the command line.

Requirements:
Raw disk/USB flash drive attached to Linux system
Local User with administrative rights (superuser)
Pre-Installed  lvm2 package

Step 1) Identify newly attached raw disk/USB drive

Login to your system, open the terminal and run following fdisk command,

    $ sudo fdisk -l | grep -i /dev/sd
  ![image](https://github.com/BettShawn/Linux-Disk-LVM-Partitioning/assets/51289343/9bdc2b36-6565-4157-b860-b76a001c4c20)

From output above, it is confirmed that new attached disk is ‘/dev/sdb’

Step 2) Create PV (Physical Volume)

Run following **pvcreate** command to create pv on disk /dev/sdb,

    $ sudo pvcreate /dev/sdb1
To verify pv status run,

    $ sudo pvs /dev/sdb1
Or

    $ sudo pvdisplay /dev/sdb1
  ![image](https://github.com/BettShawn/Linux-Disk-LVM-Partitioning/assets/51289343/e8d2dbc0-1f02-4a71-8029-553e84b2e5c2)

Step 3) Create VG (Volume Group)

To create a volume group, we will use the **vgcreate** command. Creating VG means adding pv to the volume group.

    $ sudo vgcreate volgrp-deploy /dev/sdb1
  ![image](https://github.com/BettShawn/Linux-Disk-LVM-Partitioning/assets/51289343/4100100f-1b2b-401a-9f3e-bdfc3c9ea7e7)

Run following commands to verify the status of vg (volgrp01)

    $ sudo vgs volgrp-deploy
  Or
  
    $ sudo vgdisplay volgrp-deploy
  ![image](https://github.com/BettShawn/Linux-Disk-LVM-Partitioning/assets/51289343/84f4940a-1e70-48d1-b78f-b36213af4a3f)

The above output confirms that the volume group (volgrp-deploy) of size 15 GiB is created successfully and size of one physical extend (PE) is 4 MB. PE size can be changed while creating vg.

Step 4) Create LV (Logical Volume)

Lvcreate command is used to create LV from the VG. 

In our case, following command will be used to create lv of size 10 GB

    $ sudo lvcreate --size 10G --name lvol-opt  volgrp-deploy
  ![image](https://github.com/BettShawn/Linux-Disk-LVM-Partitioning/assets/51289343/2f1b78f4-9f7d-4d8f-b94e-396bc65546a0)

Validate the status of lv, run

    $ sudo lvs /dev/volgrp-deploy/lvol-opt
  Or
  
    $ sudo lvdisplay /dev/volgrp-deploy/lvol-opt
  ![image](https://github.com/BettShawn/Linux-Disk-LVM-Partitioning/assets/51289343/d08276aa-c8da-46e2-8807-7a8ba8727b74)

Output above shows that LV (lvol-opt) has been created successfully of size 10 GiB.

Step 5) Format LVM Partition

Use mkfs command to format the lvm partition. In our case lvm partition is /dev/volgrp-deploy/lvol-opt

Note:  We can format the partition either ext4 or xfs, so choose the file system type according to your setup and requirement.

Run following command to format LVM partition as ext4 file system.

    $ sudo mkfs.ext4 /dev/volgrp-deploy/lvol-opt
  ![image](https://github.com/BettShawn/Linux-Disk-LVM-Partitioning/assets/51289343/0e0c634e-e0bc-4d9c-a1a6-621b46c1a2c2)

Now run mount command to mount it on /mnt/usbdrive folder,

    $ sudo mount /dev/volgrp-deploy/lvol-opt /mnt/usbdrive
    $ df -Th /mnt/usbdrive
  ![image](https://github.com/BettShawn/Linux-Disk-LVM-Partitioning/assets/51289343/95e8ea56-6458-4719-bff1-430b8ca19d5e)

Try to create some dummy file, run following commands,

    $ echo "testing lvm partition" | sudo tee  /mnt/usbdrive/dummy.txt
    $ cat dummy.txt
    testing lvm partition
    $ sudo rm -f  /mnt/usbdrive/dummy.txt
  ![image](https://github.com/BettShawn/Linux-Disk-LVM-Partitioning/assets/51289343/c97fff96-4ac7-4250-b196-53d500cf3b7f)

Perfect, above commands output confirm that we can access lvm partition.

To mount above lvm partition permanently, add its entries in fstab file using following echo command,

    $ echo '/dev/volgrp-deploy/lvol-opt  /mnt/usbdrive  ext4  defaults 0 0' | sudo  tee -a /etc/fstab
    $ sudo mount -a
    $ cat /etc/fstab
    









