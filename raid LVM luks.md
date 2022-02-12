#Raid 6 lvm encrypted config


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

        mdadm --create /dev/md0 --level 6 --raid-devices 4 /dev/xvdf1 /dev/xvdg1 /dev/xvdh1 /dev/xvdi1
          
- also check the details;

        mdadm --detail /dev/md0
                Note: wait until you see the clean status;
                        
                        Update Time : Sat Feb 12 19:16:18 2022
                        State : clean 
                        Active Devices : 4
                        Working Devices : 4
                        Failed Devices : 0
                        Spare Devices : 0
                        
- now that the array is ready we can start creating logical volumes in it;

- start by creating a phisical volume, runing;

        pvcreate /dev/md0
        
- then create a volume group;

        vgcreate vg0 /dev/md0
        
- and after that you can create a logical volume by running;

        lvcreate -n lv0 -l +100%FREE vg0
        
- now that you have already your lv created in the array you can encrypt it, to do that run;

        cryptsetup luksFormat --hash=sha512 --key-size=512 --cipher=aes-xts-plain64 --verify-passphrase /dev/vg0/lv0
        
                type YES and set password
                
- then run the following command to open the lv and name it;

        cryptsetup luksOpen /dev/vg0/lv0 lv0_crypt
        
- and to check the details run;

        cryptsetup luksDump -v /dev/vg0/lv0
        
- now that your lv is encrypted you'll need to format it and set a file system;

        mkfs.ext4 /dev/mapper/lv0_crypt
        
- now that you have already your lv encrypted its time to mount it;

- start by creating a directory inside /mnt;

        mkdir /mnt/raid6 - this will be your mount point       
        
- then run the following command to mount the lv0_crypt into raid6;

        mount /dev/mapper/lv0_crypt /mnt/raid6/
        
- you can check if it was mounted by runing;

        -df -hT
        
- now cat /etc/mtab and copy the last line;

        /dev/mapper/lv0_crypt /mnt/raid6 ext4 rw,relatime,stripe=256 0 0
        
- then open /etc/fstab file, paste it there and add very carefully the ,nofail parameter;

        /dev/mapper/lv0_crypt /mnt/raid6 ext4 rw,nofail,relatime,stripe=256 0 0
        
- after that reboot your instance;

- as soon as it boot again open your encrypted lv;

        cryptsetup luksOpen /dev/vg0/lv0 lv0_crypt
        
- then run;

        mount -a - to mount your encrypted logical volume
        
- and check if it got mounted;

        df -hT
        
if you can see your logical volume mounted on /mnt/raid6 that means its all working good, and your done;








          
          
