            Open Fabrics Enterprise Distribution (OFED)
                        Version 1.5
                         README
                       
                       December 2009

This is the OpenFabrics Enterprise Distribution (OFED) version 1.5
software package supporting InfiniBand and iWARP fabrics. It is composed
of several software modules intended for use on a computer cluster
constructed as an InfiniBand subnet or an iWARP network.

*** Note:  If you plan to upgrade OFED on your cluster, please upgrade all
           its nodes to this new version.

This document includes the following sections:

1. HW and SW Requirements
2. OFED Package Contents
3. Installing OFED Software
4. Starting and Verifying the IB Fabric
5. MPI (Message Passing Interface)
6. Related Documentation

OpenFabrics Home Page:  http://www.openfabrics.org

The OFED rev 1.5 software download available in
http://www.openfabrics.org/builds/ofed-1.5/release/

Please email bugs and error reports to your InfiniBand vendor, or use bugzilla
https://bugs.openfabrics.org/



1. HW and SW Requirements:
==========================
1) Server platform with InfiniBand HCA or iWARP RNIC (see OFED Distribution
   Release Notes for details)

2) Linux operating system (see OFED Distribution Release Notes for details)

3) Administrator privileges on your machine(s)

4) Disk Space:  - For Build & Installation: 300MB
                - For Installation only:    200MB

5) For the OFED Distribution to compile on your machine, some software
   packages of your operating system (OS) distribution are required. These
   are listed here.

OS Distribution         Required Packages
---------------         ----------------------------------
General:
o  Common to all        gcc, glib, glib-devel, glibc, glibc-devel,
                        glibc-devel-32bit (to build 32-bit libraries on x86_86
                        and ppc64), zlib-devel, libstdc++-devel
o  RedHat, Fedora       kernel-devel, rpm-build, redhat-rpm-config
o  SLES                 kernel-source, rpm

Note:   To build 32-bit libraries on x86_64 and ppc64 platforms, the 32-bit
        glibc-devel should be installed.

Specific Component Requirements:
o  Mvapich              a Fortran Compiler (such as gcc-g77)
o  Mvapich2             libsysfs-devel
o  Open MPI             libsysfs-devel
o  ibutils              tcl-8.4, tcl-devel-8.4, tk, libstdc++-devel
o  tvflash              pciutils-devel
o  mstflint             libstdc++-devel (32-bit on ppc64), gcc-c++
o  rnfs-utils           krb5-devel, krb5-libs, libevent-devel,
                        nfs-utils-lib-devel, openldap-devel,
                        e2fsprogs-devel (on RedHat)
                        krb5-devel, libevent-devel, nfsidmap-devel,
                        libopenssl-devel, libblkid-devel (on SLES11)
                        krb5-devel, libevent, nfsidmap, krb5, openldap2-devel,
                        cyrus-sasl-devel, e2fsprogs-devel (on SLES10)

Note:   The installer will warn you if you attempt to compile any of the
        above packages and do not have the prerequisites installed.

*** Important Note for open-iscsi users:
    Installing iSER as part of OFED installation will also install open-iscsi.
    Before installing OFED, please uninstall any open-iscsi version that may
    be installed on your machine. Installing OFED with iSER support while
    another open-iscsi version is already installed will cause the installation
    process to fail.

2. OFED Package Contents
========================

The OFED Distribution package generates RPMs for installing the following:

  o   OpenFabrics core and ULPs
        - HCA drivers (mthca, mlx4, mlx4_en, qib, ehca)
        - iWARP driver (cxgb3, nes)
        - core
        - Upper Layer Protocols: IPoIB, SDP, SRP Initiator and target, iSER
          Initiator and target, RDS, uDAPL, qlgc_vnic and NFS-RDMA.
  o   OpenFabrics utilities
        - OpenSM: InfiniBand Subnet Manager
        - Diagnostic tools
        - Performance tests
  o   MPI
        - OSU MVAPICH stack supporting the InfiniBand and iWARP interface
        - Open MPI stack supporting the InfiniBand and iWARP interface
        - OSU MVAPICH2 stack supporting the InfiniBand and iWARP interface
        - MPI benchmark tests (OSU BW/LAT, Intel MPI Benchmark, Presta)
  o   Extra packages
        - open-iscsi: open-iscsi initiator with iSER support
        - ib-bonding: Bonding driver for IPoIB interface
  o   Sources of all software modules (under conditions mentioned in the
      modules' LICENSE files)
  o   Documentation


3. Installing OFED Software
============================

The default installation directory is: /usr

Install Quick Guide:
1) Download and extract: tar xzvf OFED-1.5.tgz file.
2) Change into directory: cd OFED-1.5
3) Run as root: ./install.pl
4) Follow the directions to install required components. For details, please see
   OFED_Installation_Guide.txt under OFED-1.5/docs.


Notes:
1.  The install script removes previously installed IB packages and
    re-installs from scratch. You will be prompted to acknowledge the deletion
    of the old packages. However, configuration files (.conf) will be
    preserved and saved with a ".rpmsave" extension.

2.  After the installer completes, information about the OFED
    installation such as the prefix, the kernel version, and
    installation parameters can be found by running
    /etc/infiniband/info.

3.  Information on the driver version and source git trees can be found
    using the ofed_info utility


4. Starting and Verifying the IB Fabric
=======================================

1)  If you rebooted your machine after the installation process completed,
    IB interfaces should be up. If you did not reboot your machine, please
    enter the following command: /etc/init.d/openibd start

2)  Check that the IB driver is running on all nodes: ibv_devinfo should print
    "hca_id: <linux device name>" on the first line.
  
3)  Make sure that a Subnet Manager is running by invoking the sminfo utility.
    If an SM is not running, sminfo prints:
    sminfo: iberror: query failed
    If an SM is running, sminfo prints the LID and other SM node information.
    Example:
    sminfo: sm lid 0x1 sm guid 0x2c9010b7c2ae1, activity count 20 priority 1

    To check if OpenSM is running on the management node, enter: /etc/init.d/opensmd status
    To start OpenSM, enter: /etc/init.d/opensmd start

    Note: OpenSM parameters can be set via the file /etc/opensm/opensm.conf

4)  Verify the status of ports by using ibv_devinfo: all connected ports should
    report a "PORT_ACTIVE" state.

5)  Check the network connectivity status: run ibchecknet to see if the subnet
    is "clean" and ready for ULP/application use. The following tools display
    more information in addition to IB info: ibnetdiscover, ibhosts, and
    ibswitches.

6)  Alternatively, instead of running steps 3 to 5 you can use the ibdiagnet
    utility to perform a set of tests on your network. Upon finding an error,
    ibdiagnet will print a message starting with a "-E-". For a more complete
    report of the network features you should run ibdiagnet -r. If you have a
    topology file describing your network you can feed this file to ibdiagnet
    (using the option: -t <file>) and all reports will use the names they
    appear in the file (instead of LIDs, GUIDs and directed routes).

7)  To run an application over SDP set the following variables:
    env LD_PRELOAD='stack_prefix'/lib/libsdp.so
    LIBSDP_CONFIG_FILE=/etc/libsdp.conf <application name>
    (or LD_PRELOAD='stack_prefix'/lib64/libsdp.so on 64 bit machines)
    The default 'stack_prefix' is /usr


5. MPI (Message Passing Interface)
==================================

In Step 2 of the main menu of install.pl, options 2, 3 and 4 can
install one or more MPI stacks.  Multiple MPI stacks can be installed
simultaneously -- they will not conflict with each other.

Three MPI stacks are included in this release of OFED:
- MVAPICH
- Open MPI
- MVAPICH2

OFED also includes 4 basic tests that can be run against each MPI
stack: bandwidth (bw), latency (lt), Intel MPI Benchmark and Presta. The tests
are located under: <prefix>/mpi/<compiler>/<mpi stack>/tests/.

Please see MPI_README.txt for more details on each MPI package and how to run
the tests.


6. Related Documentation
========================
1) Release Notes for OFED Distribution components are to be found under
   OFED-1.5/docs and, after the package installation, under
   /usr/share/doc/ofed-docs-1.5 for RedHat
   /usr/share/doc/packages/ofed-docs-1.5 for SuSE.
2) For a detailed installation guide, see OFED_Installation_Guide.txt.
3) For more information, please visit the OFED web-page http://www.openfabrics.org


For more information contact your vendor.

