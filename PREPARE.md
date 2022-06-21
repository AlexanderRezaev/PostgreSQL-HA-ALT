echo 'alias nocomments="sed -e :a -re '"'"'s/<!--.*?-->//g;/<!--/N;//ba'"'"' | grep -v -P '"'"'^\s*(#|;|$)'"'"'"' >> ~/.bashrc
source ~/.bashrc

ls -la /dev/sd*
vgs --units m -o vg_name,pv_name,pv_size,pv_free
lvs --units m -o vg_name,lv_name,origin,lv_size,data_percent,mirror_log,devices
pvs --units m -o pv_name,pv_size,pv_free

# fdisk /dev/sdb
# fdisk /dev/sdc
# fdisk /dev/sdd
pvcreate /dev/sdb1
pvcreate /dev/sdc1
pvcreate /dev/sdd1
vgcreate postgresql_data_vg /dev/sdb1
vgcreate postgresql_wal_vg /dev/sdc1
vgcreate dcs_vg /dev/sdd1
lvcreate -n pg_data -l 100%FREE postgresql_data_vg
lvcreate -n pg_wal -l 100%FREE postgresql_wal_vg
lvcreate -n dcs -l 100%FREE dcs_vg

# mkfs.ext4 /dev/postgresql_data_vg/pg_data
# mkfs.ext4 /dev/postgresql_wal_vg/pg_wal

mkfs.xfs /dev/postgresql_data_vg/pg_data
mkfs.xfs /dev/postgresql_wal_vg/pg_wal
mkfs.ext4 /dev/dcs_vg/dcs

mkdir /pg_data
mkdir /pg_wal
mkdir /dcs

cat /etc/fstab | nocomments
nano /etc/fstab

/dev/postgresql_data_vg/pg_data /pg_data xfs defaults        0 0
/dev/postgresql_wal_vg/pg_wal /pg_wal ext4 rw,noatime,async,barrier=0,data=writeback,commit=60 0 0
/dev/dcs_vg/dcs /dcs ext4 rw,noatime,async,barrier=0,data=writeback,commit=60 0 0

mount -a

df -hT | grep -v "devtmpfs\|tmpfs\|squashfs"
Filesystem                             Type      Size  Used Avail Use% Mounted on
/dev/sda1                              xfs      1014M  215M  800M  22% /boot
/dev/mapper/rl-root                    xfs        14G  2.0G   12G  15% /
/dev/mapper/postgresql_wal_vg-pg_wal   xfs       8.0G   90M  7.9G   2% /pg_wal
/dev/mapper/postgresql_data_vg-pg_data xfs       8.0G   90M  7.9G   2% /pg_data
/dev/mapper/dcs_vg-dcs                 ext4      3.9G   16M  3.7G   1% /dcs

vgs --units m -o vg_name,pv_name,pv_size,pv_free
  VG                 PV         PSize     PFree
  rl                 /dev/sda2  15356.00m    0m
  postgresql_data_vg /dev/sdb1   8188.00m    0m
  postgresql_wal_vg  /dev/sdc1   8188.00m    0m
  dcs_vg             /dev/sdd1   4092.00m    0m

lvs --units m -o vg_name,lv_name,origin,lv_size,data_percent,mirror_log,devices
  VG                 LV      Origin LSize     Data%  Log Devices       
  rl                 root           13716.00m            /dev/sda2(410)
  rl                 swap            1640.00m            /dev/sda2(0)  
  postgresql_data_vg pg_data         8188.00m            /dev/sdb1(0)  
  postgresql_wal_vg  pg_wal          8188.00m            /dev/sdc1(0)  
  dcs_vg             dcs             4092.00m            /dev/sdd1(0)  

df -hT | grep -v "devtmpfs\|tmpfs\|squashfs"

vgs --units m -o vg_name,pv_name,pv_size,pv_free

lvs --units m -o vg_name,lv_name,origin,lv_size,data_percent,mirror_log,devices

