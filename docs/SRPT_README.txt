SCSI RDMA Protocol (SRP) Target driver for Linux
=================================================

SRP Target driver is designed to work directly on top of OpenFabrics
OFED-1.x software stack (http://www.openfabrics.org) or Infiniband
drivers in Linux kernel tree (kernel.org). It also interfaces with 
Generic SCSI target mid-level driver - SCST (http://scst.sourceforge.net)

By interfacing with SCST driver we are able to work and support a lot IO
modes on real or virtual devices in the backend
1. scst_disk  -- interfacing with scsi sub-system to claim and export real
scsi devices ie. disks, hardware raid volumes, tape library as SRP's luns

2. scst_vdisk -- fileio and blockio modes. This allows you to turn software
raid volumes, LVM volumes, IDE disks, block devices and normal files into
SRP's luns

3. NULLIO mode will allow you to measure the performance without sending IOs
to *real* devices


Prerequisites
-------------
0. Supported distributions: RHEL 5.3/5.4, SLES 10 sp2, SLES 11 

Note: On distribution default kernels you can run scst_vdisk blockio mode
      to have good performance. You can also run scst_disk ie. scsi pass-thru
      mode; however, you have to compile scst with -DSTRICT_SERIALIZING 
      enabled and this does not yield good performance.
      It is required to recompile the kernel to have good performance with
      scst_disk ie. scsi pass-thru mode

1. Download and install SCST driver.

a. download scst-1.0.1.tar.gz from this URL
   http://scst.sourceforge.net/downloads.html

b. untar and install scst-1.0.1
   $ tar zxvf scst-1.0.1.tar.gz
   $ cd scst-1.0.1
  
   For RedHat 5.2/5.3/5.4 and SLES 11:
   $ make && make install

   For SLES 10 sp2
   . Save the following patch to /tmp/scst_sles10_sp2.patch

/************************  Start scst_sless_spX.patch *********************/

diff -Naur scst-1.0.1/src/scst_lib.c scst-1.0.1.wk/src/scst_lib.c
--- scst-1.0.1/src/scst_lib.c	2009-04-27 12:14:21.000000000 -0700
+++ scst-1.0.1.wk/src/scst_lib.c	2009-08-14 13:53:40.000000000 -0700
@@ -1680,7 +1680,7 @@
 	return res;
 }
 
-#if LINUX_VERSION_CODE < KERNEL_VERSION(2, 6, 18)
+#if LINUX_VERSION_CODE < KERNEL_VERSION(2, 6, 16)
 static void scst_req_done(struct scsi_cmnd *scsi_cmd)
 {
 	struct scsi_request *req;
@@ -1743,7 +1743,7 @@
 	TRACE_EXIT();
 	return;
 }
-#else /* LINUX_VERSION_CODE < KERNEL_VERSION(2, 6, 18) */
+#else /* LINUX_VERSION_CODE < KERNEL_VERSION(2, 6, 16) */
 static void scst_send_release(struct scst_device *dev)
 {
 	struct scsi_device *scsi_dev;
@@ -1790,7 +1790,7 @@
 	TRACE_EXIT();
 	return;
 }
-#endif /* LINUX_VERSION_CODE < KERNEL_VERSION(2, 6, 18) */
+#endif /* LINUX_VERSION_CODE < KERNEL_VERSION(2, 6, 16) */
 
 /* scst_mutex supposed to be held */
 static void scst_clear_reservation(struct scst_tgt_dev *tgt_dev)
@@ -2038,7 +2038,7 @@
 	sBUG_ON(cmd->inc_blocking || cmd->needs_unblocking ||
 		cmd->dec_on_dev_needed);
 
-#if LINUX_VERSION_CODE < KERNEL_VERSION(2, 6, 18)
+#if LINUX_VERSION_CODE < KERNEL_VERSION(2, 6, 16)
 #if defined(CONFIG_SCST_EXTRACHECKS)
 	if (cmd->scsi_req) {
 		PRINT_ERROR("%s: %s", __func__, "Cmd with unfreed "
@@ -2212,7 +2212,7 @@
 	return;
 }
 
-#if LINUX_VERSION_CODE < KERNEL_VERSION(2, 6, 18)
+#if LINUX_VERSION_CODE < KERNEL_VERSION(2, 6, 16)
 int scst_alloc_request(struct scst_cmd *cmd)
 {
 	int res = 0;
diff -Naur scst-1.0.1/src/scst_main.c scst-1.0.1.wk/src/scst_main.c
--- scst-1.0.1/src/scst_main.c	2009-04-27 12:13:59.000000000 -0700
+++ scst-1.0.1.wk/src/scst_main.c	2009-08-14 13:56:17.000000000 -0700
@@ -1672,7 +1672,7 @@
 
 	TRACE_ENTRY();
 
-#if LINUX_VERSION_CODE < KERNEL_VERSION(2, 6, 18)
+#if LINUX_VERSION_CODE < KERNEL_VERSION(2, 6, 16)
 	{
 		struct scsi_request *req;
 		BUILD_BUG_ON(SCST_SENSE_BUFFERSIZE !=
diff -Naur scst-1.0.1/src/scst_priv.h scst-1.0.1.wk/src/scst_priv.h
--- scst-1.0.1/src/scst_priv.h	2009-04-27 12:14:12.000000000 -0700
+++ scst-1.0.1.wk/src/scst_priv.h	2009-08-14 16:29:11.000000000 -0700
@@ -27,7 +27,7 @@
 #include <scsi/scsi_device.h>
 #include <scsi/scsi_host.h>
 
-#if LINUX_VERSION_CODE < KERNEL_VERSION(2, 6, 18)
+#if LINUX_VERSION_CODE < KERNEL_VERSION(2, 6, 16)
 #include <scsi/scsi_request.h>
 #endif
 
@@ -322,7 +322,7 @@
 void scst_check_retries(struct scst_tgt *tgt);
 void scst_tgt_retry_timer_fn(unsigned long arg);
 
-#if LINUX_VERSION_CODE < KERNEL_VERSION(2, 6, 18)
+#if LINUX_VERSION_CODE < KERNEL_VERSION(2, 6, 16)
 int scst_alloc_request(struct scst_cmd *cmd);
 void scst_release_request(struct scst_cmd *cmd);
 
diff -Naur scst-1.0.1/src/scst_targ.c scst-1.0.1.wk/src/scst_targ.c
--- scst-1.0.1/src/scst_targ.c	2009-04-27 12:14:27.000000000 -0700
+++ scst-1.0.1.wk/src/scst_targ.c	2009-08-14 13:55:23.000000000 -0700
@@ -1272,7 +1272,7 @@
 	return context;
 }
 
-#if LINUX_VERSION_CODE < KERNEL_VERSION(2, 6, 18)
+#if LINUX_VERSION_CODE < KERNEL_VERSION(2, 6, 16)
 static inline struct scst_cmd *scst_get_cmd(struct scsi_cmnd *scsi_cmd,
 					    struct scsi_request **req)
 {
@@ -1324,7 +1324,7 @@
 	TRACE_EXIT();
 	return;
 }
-#else /* LINUX_VERSION_CODE < KERNEL_VERSION(2, 6, 18) */
+#else /* LINUX_VERSION_CODE < KERNEL_VERSION(2, 6, 16) */
 static void scst_cmd_done(void *data, char *sense, int result, int resid)
 {
 	struct scst_cmd *cmd;
@@ -1346,7 +1346,7 @@
 	TRACE_EXIT();
 	return;
 }
-#endif /* LINUX_VERSION_CODE < KERNEL_VERSION(2, 6, 18) */
+#endif /* LINUX_VERSION_CODE < KERNEL_VERSION(2, 6, 16) */
 
 static void scst_cmd_done_local(struct scst_cmd *cmd, int next_state,
 	enum scst_exec_context pref_context)
@@ -1875,7 +1875,7 @@
 static int scst_do_real_exec(struct scst_cmd *cmd)
 {
 	int res = SCST_EXEC_NOT_COMPLETED;
-#if LINUX_VERSION_CODE >= KERNEL_VERSION(2, 6, 18)
+#if LINUX_VERSION_CODE >= KERNEL_VERSION(2, 6, 16)
 	int rc;
 #endif
 	bool atomic = scst_cmd_atomic(cmd);
@@ -1942,7 +1942,7 @@
 	}
 #endif
 
-#if LINUX_VERSION_CODE < KERNEL_VERSION(2, 6, 18)
+#if LINUX_VERSION_CODE < KERNEL_VERSION(2, 6, 16)
 	if (unlikely(scst_alloc_request(cmd) != 0)) {
 		if (atomic) {
 			res = SCST_EXEC_NEED_THREAD;
@@ -1993,7 +1993,7 @@
 	scst_set_cmd_error(cmd, SCST_LOAD_SENSE(scst_sense_hardw_error));
 	goto out_done;
 
-#if LINUX_VERSION_CODE < KERNEL_VERSION(2, 6, 18)
+#if LINUX_VERSION_CODE < KERNEL_VERSION(2, 6, 16)
 out_busy:
 	scst_set_busy(cmd);
 	/* go through */
/************************  End scst_sless10_sp2.patch ***********************/

   . patch -p1 < /tmp/scst_sles10_sp2.patch
   . make && make install

c. Save the following patch into /tmp/scst.patch.
   If you run SLES 11 you can skip steps (c,d) and go directly to step (2)

/************************  Start scst.patch *********************/
diff -Naur scst/scst.h scst.wk/scst.h
--- scst/scst.h	2009-08-14 17:09:47.000000000 -0700
+++ scst.wk/scst.h	2009-08-14 17:01:28.000000000 -0700
@@ -2581,7 +2581,7 @@
 void scst_aen_done(struct scst_aen *aen);
 
 #if LINUX_VERSION_CODE < KERNEL_VERSION(2, 6, 24)
-
+#ifndef RHEL_RELEASE_CODE
 static inline struct page *sg_page(struct scatterlist *sg)
 {
 	return sg->page;
@@ -2610,6 +2610,7 @@
 	sg->length = len;
 }
 
+#endif
 #endif /* LINUX_VERSION_CODE < KERNEL_VERSION(2, 6, 24) */
 
 static inline void sg_clear(struct scatterlist *sg)
/************************  End scst.patch *********************/

d. patch scst.h file with /tmp/scsi_tgt.patch
   $ cd /usr/local/include/scst;
   $ patch -p1 < /tmp/scst.patch

2. Download/install OFED-1.5 - SRP target is part of OFED

Note: if your system already have OFED stack installed, you need to remove
      all the previous built RPMs and reinstall
      
   $ cd ~/OFED-1.5
   $ rm RPMS/*
   $ ./install.pl -c ofed.conf

a. download OFED packages from this URL
   http://www.openfabrics.org/downloads/OFED/

b. install OFED - remember to choose srpt=y
   $ cd ~/OFED-1.5
   $ ./install.pl


How-to run
-----------
A. On srp target machine

1. Please refer to SCST's README for loading scst driver and its dev_handlers
drivers (scst_disk, scst_vdisk block or file IO mode, nullio, ...)

Note: In any mode you always need to have lun 0 in any group's device list
      Then you can have any lun number following lun 0 (it does not required
      have lun number in order except that the first lun is always 0)

      Setting SRPT_LOAD=yes in /etc/infiniband/openib.conf is not good enough
      It only load ib_srpt module and does not load scst and its dev_handlers
 
Example 1: working with VDISK BLOCKIO mode
           (using md0 device, sda, and cciss/c1d0)
a. modprobe scst
b. modprobe scst_vdisk
c. echo "open vdisk0 /dev/md0 BLOCKIO" > /proc/scsi_tgt/vdisk/vdisk
d. echo "open vdisk1 /dev/sda BLOCKIO" > /proc/scsi_tgt/vdisk/vdisk
e. echo "open vdisk2 /dev/cciss/c1d0 BLOCKIO" > /proc/scsi_tgt/vdisk/vdisk
f. echo "add vdisk0 0" >/proc/scsi_tgt/groups/Default/devices
g. echo "add vdisk1 1" >/proc/scsi_tgt/groups/Default/devices
h. echo "add vdisk2 2" >/proc/scsi_tgt/groups/Default/devices

Example 2: working with real back-end scsi disks in scsi pass-thru mode
a. modprobe scst
b. modprobe scst_disk
c. cat /proc/scsi_tgt/scsi_tgt
ibstor00:~ # cat /proc/scsi_tgt/scsi_tgt 
Device (host:ch:id:lun or name)                             Device handler
0:0:0:0                                                     dev_disk
4:0:0:0                                                     dev_disk
5:0:0:0                                                     dev_disk
6:0:0:0                                                     dev_disk
7:0:0:0                                                     dev_disk

Now you want to exclude the first scsi disk and expose the last 4 scsi disks
as IB/SRP luns for I/O

echo "add 4:0:0:0 0" >/proc/scsi_tgt/groups/Default/devices
echo "add 5:0:0:0 1" >/proc/scsi_tgt/groups/Default/devices
echo "add 6:0:0:0 2" >/proc/scsi_tgt/groups/Default/devices
echo "add 7:0:0:0 3" >/proc/scsi_tgt/groups/Default/devices

Example 3: working with scst_vdisk FILEIO mode
           (using md0 device and file 10G-file)
a. modprobe scst
b. modprobe scst_vdisk
c. echo "open vdisk0 /dev/md0" > /proc/scsi_tgt/vdisk/vdisk
d. echo "open vdisk1 /10G-file" > /proc/scsi_tgt/vdisk/vdisk
e. echo "add vdisk0 0" >/proc/scsi_tgt/groups/Default/devices
f. echo "add vdisk1 1" >/proc/scsi_tgt/groups/Default/devices

2. modprobe ib_srpt

B. On initiator machines you can manualy do the following steps:

1. modprobe ib_srp
2. ipsrpdm -c -d /dev/infiniband/umadX 
   (to discover new SRP target)
   umad0: port 1 of the first HCA
   umad1: port 2 of the first HCA
   umad2: port 1 of the second HCA
3. echo {new target info} > /sys/class/infiniband_srp/srp-mthca0-1/add_target
4. fdisk -l (will show new discovered scsi disks)

Example:
Assume that you use port 1 of first HCA in the system ie. mthca0

[root@lab104 ~]# ibsrpdm -c -d /dev/infiniband/umad0
id_ext=0002c90200226cf4,ioc_guid=0002c90200226cf4,
dgid=fe800000000000000002c90200226cf5,pkey=ffff,service_id=0002c90200226cf4
[root@lab104 ~]# echo id_ext=0002c90200226cf4,ioc_guid=0002c90200226cf4,
dgid=fe800000000000000002c90200226cf5,pkey=ffff,service_id=0002c90200226cf4 >
/sys/class/infiniband_srp/srp-mthca0-1/add_target

OR

+ You can edit /etc/infiniband/openib.conf to load srp driver and srp HA daemon
automatically ie. set SRP_LOAD=yes, and SRPHA_ENABLE=yes
+ To set up and use high availability feature you need dm-multipath driver
and multipath tool
+ Please refer to OFED-1.x SRP's user manual for more in-details instructions
on how-to enable/use HA feature

Here is an example of srp target setup file
--------------------------------------------

#!/bin/sh

modprobe scst scst_threads=1
modprobe scst_vdisk scst_vdisk_ID=100

echo "open vdisk0 /dev/cciss/c1d0 BLOCKIO" > /proc/scsi_tgt/vdisk/vdisk
echo "open vdisk1 /dev/sdb BLOCKIO" > /proc/scsi_tgt/vdisk/vdisk
echo "open vdisk2 /dev/sdc BLOCKIO" > /proc/scsi_tgt/vdisk/vdisk
echo "open vdisk3 /dev/sdd BLOCKIO" > /proc/scsi_tgt/vdisk/vdisk
echo "add vdisk0 0" > /proc/scsi_tgt/groups/Default/devices
echo "add vdisk1 1" > /proc/scsi_tgt/groups/Default/devices
echo "add vdisk2 2" > /proc/scsi_tgt/groups/Default/devices
echo "add vdisk3 3" > /proc/scsi_tgt/groups/Default/devices

modprobe ib_srpt

echo "add "mgmt"" > /proc/scsi_tgt/trace_level
echo "add "mgmt_dbg"" > /proc/scsi_tgt/trace_level
echo "add "out_of_mem"" > /proc/scsi_tgt/trace_level


How-to unload/shutdown
-----------------------

1. Unload ib_srpt
 $ modprobe -r ib_srpt
2. Unload scst and its dev_handlers first
 $ modprobe -r scst_vdisk scst
3. Unload ofed
 $ /etc/rc.d/openibd stop
