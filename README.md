# basic command line set up for chia.

### basic disk management
#### xfs
```
sudo apt-get install xfsprogs
```
```
sudo modprobe -v xfs
```
```
sudo mkfs.xfs /dev/sdq
```

#### xfs label
```
sudo xfs_admin -L "F06" /dev/sd
```

#### ext4
```
sudo mkfs.ext4 /dev/sdq
```

#### ext label
```
sudo e2label /dev/sdq F06
```

#### for unmounting disk /dev/sdy
```
sudo umount /dev/sdy
```
#### for mounting disk /dev/sdy
#### mountpoint is a directory under root (/), mount the filesystem on device /dev/sdy in there.
```
sudo mount /dev/sdy /mountpoint
```

#### if you want the disk to be mounted on login at a specific path (with access times log turned off).
```
sudo blkid
```
note down the UUID for the device and the file system format.
add following line to `/etc/fstab`
an example for disk labelled `F00` having an `ext4` filesystem for a user john:
```
UUID=7b7d38e1-f1d4-4020-9f0d-ed55e277f052    /media/john/F00    ext4    defaults,noatime,nodiratime    0    0
```

then
```
mkdir -p /media/${USER}/F00
```
next time system is rebooted, the auto mount would happen at `/media/john/F00`

#### dont forget to provide write permissions once booted (one time setup after each disk is installed)
777 or 666
```
sudo chmod -R 777 /media/${USER}
```

### Activate py environment
```
cd chia
. ./activate
```

### logs
#### enable log level to info
```
chia configure -log-level INFO
```

#### multitail
```
sudo apt install -y multitail
multitail -n 1000 f00.log f01.log f02.log -s 2
```

#### harvester grep
```
tail -100f  ~/.chia/mainnet/log/debug.log | grep -E 'chia.harvester.harvester'
```

### create plots
#### F for fast drives, S for slow drives
```
chia plots create -k 32 -n 5 -r 4 -t /media/${USER}/F00 -2 /media/${USER}/S00 -d /media/${USER}/S00 >> ~/logs/r01/f00.log 2>&1 &
```

### if your harddrives sleep off, turn sleep off by:
sudo sdparm --clear=STANDBY /dev/sdj -S

### See nvme statistics - including life
```
sudo nvme smart-log /dev/nvme5n1
```

### See RAM stats
```
sudo lshw -short -C memory
```

### for harvestor first do chia init followed by chia init -c copied-ca-folder-from-full-node


### references
https://github.com/Chia-Network/chia-blockchain/wiki/Farming-on-many-machines

### glances - for system stats
```
apt install -y glances
glances
```

### For partition and formatting each partition
[Partion a drive and format](https://techguides.yt/guides/how-to-partition-format-and-auto-mount-disk-on-ubuntu-20-04/)

### log stats for average time taken
```
total=`grep -E 'Total time' *.log | cut -d ' ' -f4 | sort | paste -sd+ | bc`
count=`grep -E 'Total time' *.log | wc -l`
average=`echo "scale=2;${total}/(3600 * ${count})" | bc`
cat << EOF
########################################################
total ${count} plots on this run
average time taken to complete a plot: ${average} hours
########################################################
EOF
```

### increase swap space by adding an additional swap file
```
sudo fallocate -l 16G /swapfile16g
ls -lh /swapfile16g
sudo chmod 600 /swapfile16g
sudo mkswap /swapfile16g
sudo swapon /swapfile16g
echo '/swapfile16g none swap sw,pri=10 0 0' | sudo tee -a /etc/fstab
sudo findmnt --verify --verbose
sudo swapon --show
sudo free -h
```
