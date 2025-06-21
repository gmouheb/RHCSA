# Lab 0: Disk Partitioning and Mounting

This lab focuses on creating and managing disk partitions, formatting them with different filesystems, and mounting them persistently.

## Requirements

1. Create a primary partition with a size of 1 GiB. Format it with Ext4, and mount it persistently on /mounts/lab0, using its UUID.
2. Create an extended partition that includes all remaining disk space. In this partition, create a 500MiB XFS partition and mount it persistently on /mounts/xfslab0, using the label myxfslab0.
3. Create a 500MiB swap partition and mount it persistently.

Note: Add 10GiB on your VM so you can do the LAB.
