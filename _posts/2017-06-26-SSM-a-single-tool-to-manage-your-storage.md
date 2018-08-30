---
layout: post

title: "SSM: a single tool to manage your storage"

date: 2017-06-26 15:24:20 +0300

description:  

img: disk.jpg # Add image post (optional)

tags: [linux]
---

### SSM ( System-Storage-Manager ) 

*System Storage Manager* (SSM) provides a command line interface to manage storage in various technologies. We will show a kind of use case that how to manage a brand new disk and mount it on the certain point.<!-- more --> 

### Background

We consider such a situation that we already have had a machine running with Linux OS, and now we finding out the space of `/opt` is almost running out in some reasons. We decide add a new disk for this node ( if you are using virtual machine  you just add a new disk without any other operations, but you have to take consideration of raid in physical machine case ) endeavor to migrate data to this new space and keep the mount point ( `/opt` ) stays the same for long. 

### Script

Here we give a comparatively complete steps in this script demo though the command is pretty simple. You still have to notice that there are several special conditions are limited in our case: 1) Available yum source; 2) The device name of disk new added is `/dev/sdb` and the mount point is `/opt`; 3) The logic volume name is `lv_opt`, the volume group name is `datavg`, those are specified in command `ssm create -n lv_opt --fstype xfs -p datavg /dev/sdb /opt`

```shell
#!/bin/sh
#if [ $# -ne 1 ];then
#  action "USAGE: $0 disk-use"
#  exit 1
#fi
. /etc/init.d/functions

current_user=`whoami`
if [[ "$current_user" == "root" ]];then
  action "ensure user is root" /bin/true
else
  action "run script must is root" /bin/false
  exit 1
fi

rpm -qa |grep "system-storage-manager"
if [ $? -eq 0 ];then
  action "system-storage-manager have been install " /bin/true
else

  yum list |grep system-storage-manager > /dev/null
  if [ $? -ne 0 ];then
    action "yum list haven't system-storage-manager" /bin/false
    exit 1
  fi
  echo "start install system-storage-manager..."

  yum -y install system-storage-manager > /dev/null
  rpm -qa |grep "system-storage-manager" > /dev/null
  if [ $? -ne 0 ];then
    action "system-storage-manager install fail " /bin/false
    exit 1
  fi
  action "system-storage-manager install Success" /bin/true
fi

opt_line=`ls -l /opt | wc -l`
echo  "/opt file lines is $opt_line"
mkdir -p /tmp/opt
mv /opt/* /tmp/opt
tmp_line=`ls -l /tmp/opt | wc -l`
echo  "/tmp/opt file lines is $tmp_line"

if [[ "$opt_line" == "$tmp_line" ]];then
  action "/opt filelist mv Success " /bin/true
else
  action "/opt filelist mv fail " /bin/false
  exit 1
fi

fdisk -l |grep /dev/sdb > /dev/null
if [ $? -ne 0 ];then
  action "fdisk /dev/sdb not exist " /bin/false
  exit 1
fi
action "fdisk /dev/sdb  ready OK " /bin/true

#ssm create -s 100%FREE -n lv_opt --fstype xfs -p datavg /dev/sdb /opt > /dev/null
ssm create -n lv_opt --fstype xfs -p datavg /dev/sdb /opt > /dev/null
pvs |grep /dev/sdb
if [ $? -ne 0 ];then
  action "PV create fail " /bin/false
  exit 1
fi
action "PV create Success " /bin/true
vgs |grep datavg
if [ $? -ne 0 ];then
  action "VG create fail " /bin/false
  exit 1
fi
action "VG create Success " /bin/true
lvs |grep lv_opt
if [ $? -ne 0 ];then
  action "LV create fail " /bin/false
  exit 1
fi
action "LV create Success " /bin/true
df -Th |grep "opt" > /dev/null
if [ $? -ne 0 ];then
  action "fdisk /dev/sdb not exist " /bin/false
  exit 1
fi
action "/opt mount Success " /bin/true

echo  "/dev/mapper/datavg-lv_opt /opt                    xfs    defaults        0 0" >> /etc/fstab
action "write mount INFO to /etc/fstab " /bin/true
tail -1 /etc/fstab

echo  "start auto mount check"
umount /opt
echo  "start umount /opt "
df -Th |grep "opt" > /dev/null
if [ $? -eq 0 ];then
  action "/opt umount fail " /bin/false
  exit 1
fi
action "/opt umount Success " /bin/true
echo "Start Auto mount "
mount -a
df -Th |grep "opt" > /dev/null
if [ $? -ne 0 ];then
  action "fdisk /dev/sdb auot mount fail " /bin/false
  exit 1
fi
action "/opt mount Success " /bin/true

echo  "/tmp/opt/ file now move /opt "
mv /tmp/opt/* /opt/.
new_opt_line=`ls -l /opt/. |wc -l`
if [[ "$new_opt_line" == "$opt_line" ]];then
  action "/tmp/opt filelist mv Success " /bin/true
  echo  "current line is : $new_opt_line"
else
  action   "/tmp/opt filelist mv fail " /bin/false
  exit 1
fi

ls -l /opt
echo  "all of operation End "
```

### Check

Use `df -h` check the result.
