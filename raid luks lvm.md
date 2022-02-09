#Raid 5 encrypted + lvm config


- start by installing this following packages to check if they are all installed;
        
        apt install mdadm gdisk raidutils cryptsetup -y
        
- now you'll need to format all the divices you'r using for the array;

        - start by creating new GPT partition tables on each device, to do this run;
        
                gdisk /dev/xvdf
                type o - for gtp part table
                n - add a new partition
                now press enter 3 times for - partition number, first sector and last sector
                then for the filesystem code - type fd00 for Linux Raid filesystem
                
                then type w to save
                
- after all devices are format you can make the array by running;
        
        mdadm --create /dev/md0 --level 5 --raid-devices 3 /dev/xvdf1 /dev/xvdg1 /dev/xvdh1
        
- also check the details;

        mdadm --detail /dev/md0 
                Note: wait until you see the clean status;
                        
                        Update Time : Tue Feb  8 22:59:19 2022
                        State : clean 
                        Active Devices : 3
                        Working Devices : 3
                        Failed Devices : 0
                        Spare Devices : 0
               
- now that the array is ready we can encrypt it, to do that run;

        cryptsetup luksFormat --hash=sha512 --key-size=512 --cipher=aes-xts-plain64 --verify-passphrase /dev/md0 
        
                type YES and set password
                
- then run the following command to open the array and name it;

        cryptsetup luksOpen /dev/md0 md0_crypt
        
- and to check the details run;

        cryptsetup luksDump -v /dev/md0
        
- after this the array it's already encrypted so we can start creating logical volumes in it;

- start by creating a phisical volume, runing;

        pvcreate /dev/mapper/md0_crypt
        
- then create a volume group;

        vgcreate vg0 /dev/mapper/md0_crypt
        
- and after that you can create a logical volume by  running;

        lvcreate -n lv0 -l +100%FREE vg0
        
- you will also need to format your lv0 and set a filesystem;

        mkfs.ext4 /dev/vg0/lv0 - in my case I set ext4 filesystem
        
- now that you have already your lv created in the encrypted array its time to mount it;

- start by creating a directory inside /mnt;

        mkdir /mnt/raid5 - this will be your mount point
        
- then run the following command to mount the lv0 into raid5;

        mount /dev/vg0/lv0 /mnt/raid5/
        
- you can check if it was mounted by runing;

        df -hT
        
- now cat /etc/mtab and copy the last line;

        /dev/mapper/vg0-lv0 /mnt/raid5 ext4 rw,relatime,stripe=256 0 0
        
- then open /etc/fstab file, paste it there and add very carefully the ,nofail parameter;

        /dev/mapper/vg0-lv0 /mnt/raid5 ext4 rw,relatime,nofail,stripe=256 0 0
        
- after that reboot your instance;

- as soon as it boot again open your encrypted array;

        cryptsetup luksOpen /dev/md0 md0_crypt - type password
        
- then run;

        mount -a - to mount your logical volume inside the array
        
- and check if it got mounted;

        df -hT 
                
- if you can see your logical volume mounted on /mnt/raid5 that means its all working good, and your done;


