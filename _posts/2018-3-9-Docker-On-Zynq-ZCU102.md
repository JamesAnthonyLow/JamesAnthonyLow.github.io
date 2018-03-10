This guide is meant to serve as a walk-through regarding the use of the [Xilinx Petalinux tools](http://www.wiki.xilinx.com/PetaLinux?responseToken=03e1bb67ae0bfe14ce7c225634ac9f676) to configure and build [Xilinx Linux](https://github.com/Xilinx/linux-xlnx) for the [Zynq ZCU102](https://www.xilinx.com/products/boards-and-kits/ek-u1-zcu102-g.html).   The reader will also learn how to use [Docker](https://www.docker.com/) both on the PC and Zynq for the purpose of performing [containerized development](https://www.infoworld.com/article/3204171/linux/what-is-docker-linux-containers-explained.html).

### Docker: What and Why?

- [A Not Very Short Introduction To Docker](https://blog.jayway.com/2015/03/21/a-not-very-short-introduction-to-docker/)
- Email [JamesALowenthal@gmail.com](mailto:jamesalowenthal@gmail.com) if you have any further questions

Docker can most easily be explained as "```chroot``` on steroids".  If you have never used ```chroot```, it is a Linux utility that allows you to run an interactive shell with the root directory of your choice.  So if you had a hard-disk with a completely different Linux OS on it than the one on your host you could "enter" the file-system of that OS using ```chroot``` (given you are running the same architecture and kernel).  Docker utilizes this concept to produce "file-system images" called "Docker images" that can be packaged with various dependencies for a specific application.  The beauty of this is that you can configure your Docker image with whatever it needs to run an application and easily reproduce it in a variety of environments.  This approach and Docker in general was and is intended primarily for web developers  to reduce variability between development and production machines, however it can also be incredibly useful as a method to streamline the process of getting dependencies onto an embedded machine.  This is important in the case of the Zynq because it allows us to use Xilinx's Linux custom distribution to get the most functionality out of the Zynq's specialized architecture while gaining access to the richer tools and package managers of distributions such as Ubuntu or Arch.  Furthermore, we can use Docker to improve our Xilinx workflow on our host machines by producing Docker images for the various Xilinx SDKs.  The lengthy download and build times and squirrelly dependencies can make these SDKs a nightmare to install **especially on air-gapped machines**, however we can circumvent this with Docker.  We can adopt a **build once, use often** approach by having one developer go through the install process from within a Docker container, save and then distribute the image of that container.  

## Table of Contents

 1. [Installing Petalinux 2017.1 on Host Machine](#installing-petalinux-2017.1-on-host-machine)
 2. [Build Linux for the Zynq ZCU102 using the Xilinx BSP](#build-linux-for-the-zynq-zcu102-using-the-xilinx-bsp)
 3. [Installing Docker on an Air-Gapped 64-bit Linux Machine](#installing-docker-on-an-air-gapped-64-bit-linux-machine)
 
## Installing Petalinux 2017.1 On Host Machine

#### Method 1: Install Natively Without Docker (Not Recommended)

##### Requirements:

 - [Petalinux v 2017.1 Installer (7.54 GB)](https://www.xilinx.com/member/forms/download/xef.html?filename=petalinux-v2017.1-final-installer.run&akdm=1) 
 
 ##### Instructions:
 
 See [Petalinux documentation](https://www.xilinx.com/support/documentation/sw_manuals/xilinx2017_1/ug1144-petalinux-tools-reference-guide.pdf)
 
 #### Method 2: Install Using Docker (Preferred)
 
 ##### Requirements:
 
 - [jamesanthonylow/petalinux:latest (if you can access Docker Hub ~40 GB)]() OR [petalinux.tar (if you can't access Docker Hub ~40 GB)]() (available by request only)
 - [run.sh (4.0 KB)](https://raw.githubusercontent.com/JamesAnthonyLow/docker-xilinx-petalinux-desktop/master/run.sh)
 - [Host PC running Docker](https://www.docker.com/community-edition) to install Docker on an air-gapped Linux machine please see the guide [below](#installing-docker-on-an-air-gapped-linux-machine)

**Note:** the petalinux images are so large because the comparative sizes of the Xilinx SDK and petalinux tools are quite large to begin with so unfortunately this cannot really be avoided.

You can find more information regarding the building of the **petalinux** image and its operation in 

##### Instructions:

 1. Load **petalinux:latest** image:
		- **With Docker Hub:**
			- pull image from docker hub --> ```docker pull jamesanthonylow/petalinux:2017.1```
			- tag image as **petalinux:latest** --> ```docker tag jamesanthonylow/petalinux:2017.1 petalinux:latest```
		- **Without Docker Hub:**	
			- obtain ```petalinux.tar``` from a secure source.
			- load the ```petalinux.tar``` as a docker image --> ```docker load -i petalinux.tar``` 
		
 2. Check that the image was loaded properly:
 ```bash
 $ docker images
 REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
 petalinux   		 latest        	 00f0a1152c7b        19 hours ago        42.3GB
 ```
3. Create Petalinux work directory with workspace subdirectory \<My petalinux work\>/workspace:
```bash
$ mkdir -p <My petalinux work>/workspace
```
4. Place the **run.sh** script into  \<My petalinux work\>.

This script will run the **petalinux:latest** image interactively (```-it```)  and sync the ```<My petalinux work>/workspace``` directory with the ```<My petalinux work>/workspace``` directory (```v```), meaning that all of your work done inside the **petalinux** container will be saved on your host machine in the ```/workspace``` directory.  This makes obtaining your build outputs much easier, **and any files you need Petalinux to access such as your BSP should go in this shared directory**.

5. ```cd``` into your ```<My petalinux work>``` directory and run run.sh:
```bash
$ cd <My petalinux work> && ./run.sh
```
Repeat step 5 anytime you want to do some Petalinux work, for example to build Linux for the Zynq ZCU10d in the next section!

## Build Linux for the Zynq ZCU102 using the Xilinx BSP

##### Requirements

- [xilinx-zcu102-v2017.1-final.bsp (299.45 MB)](https://www.xilinx.com/member/forms/download/xef.html?filename=xilinx-zcu102-v2017.1-final.bsp&akdm=1)
- [SD Card (>=32 GB)](https://www.amazon.com/SanDisk-Ultra-Class-Memory-SDSDUNC-032G-GN6IN/dp/B0143RT8OY)
- [USB SD Card Reader](https://www.amazon.com/UGREEN-Reader-Memory-Windows-Simultaneously/dp/B01EFPX9XA/ref=sr_1_6?s=electronics&ie=UTF8&qid=1517588403&sr=1-6&keywords=sd+card+reader)

**Important**: You will also need *.bit wrapper and design wrapper *.hdf  files both of which can be produced with Vivado but since the production of these files is outside the scope of this guide you can instead obtain them through two other projects provided by Xilinx.  The *.bit file can be obtained from the TRD: [zcu102-base-trd-2017-1.zip (474 MB)](https://www.xilinx.com/member/forms/download/design-license.html?cid=6af8b7ef-685e-4338-83eb-c611bffb5839&filename=rdf0421-zcu102-base-trd-2017-1.zip).  Just download and unzip the directory and then copy the file at ```rdf0421-zcu102-base-trd-2017-1/apu/sdsoc_pfm/zcu102_base_trd/sw/prebuilt/zcu102_Base_trd_wrapper.bit``` to use as your FPGA design.  The *hdf can be found as part of the [Ubuntu_Desktop_Release_2017_4.zip](https://www.xilinx.com/member/forms/download/xef.html?filename=Ubuntu_Desktop_Release_2017_4.zip&akdm=1) resources provided as part of Xilinx's [Zynq UltraScale + MPSoC Ubuntu part 2 tutorial](http://www.wiki.xilinx.com/Zynq+UltraScale%EF%BC%8B+MPSoC+Ubuntu+part+2+-+Building+and+Running+the+Ubuntu+Desktop+From+Sources) (which I know sounds promising but I do not recommend at this time for the ZCU102).  Just download and unzip the directory, you will find the relevant *.hdf in the directory
1. Create a new Petalinux project using **xilinx-zcu102-v2017.1-final.bsp** (board support package)
```bash
$ petalinux-create -t project –s xilinx-zcu102-v2017.1-final.bsp
```
This will create the directory ```xilinx-zcu102-2017.1```

2. Enter the new project directory and configure the bootloader:
```bash
$ cd xilinx-zcu102-2017.1 && petalinux-config --get-hw-description=<path of dir containing *.hdf file>
```  
An [ncurses](https://linux.die.net/man/3/ncurses) menu will pop up.  You need to select **SD card** as the **Root filesystem type** by selecting it in the menu ```Image Packaging Configuration ---> Root filesystem type (SD card)```.  After making the selection you can **save** and then **exit** the bootloader configuration menu.

3. Now is time for the hard part, configuring the kernel:
```bash
$ petalinux-config -c kernel
```
This will give you access to the kernel menu.  It is important that you enable all the features required by Docker.  This is all laid out nicely in the following graphic copied from the [Gentoo Linux Wiki](https://wiki.gentoo.org/wiki/Docker).  I highly recommend the [Gentoo Wiki](https://wiki.gentoo.org) as a resource for kernel configuration.  Any time you need use a utility that requires kernel support ([iptables](https://wiki.gentoo.org/wiki/Iptables) for example), just search for it in the Gentoo Wiki and copy the kernel configuration from there.

**Note:** It is important to keep in mind that some menu options will not show up until other dependencies are enabled. For example ```"ipvs" match support``` will not show up until you enable ```IP virtual server support```. If you have trouble finding anything you can bring up a search menu by typing ```/```.
```
General setup  --->
 [*] POSIX Message Queues
    -*- Control Group support  --->
        [*]   Memory controller 
        [*]     Swap controller
        [*]       Swap controller enabled by default
        [*]   IO controller
        [*]     IO controller debugging
        [*]   CPU controller  --->
              [*]   Group scheduling for SCHED_OTHER
              [*]     CPU bandwidth provisioning for FAIR_GROUP_SCHED
              [*]   Group scheduling for SCHED_RR/FIFO
        [*]   PIDs controller
        [*]   Freezer controller
        [*]   HugeTLB controller
        [*]   Cpuset controller
        [*]     Include legacy /proc/<pid>/cpuset file
        [*]   Device controller
        [*]   Simple CPU accounting controller
        [*]   Perf controller
        [*]   Example controller 
    -*- Namespaces support
        [*]   UTS namespace
        -*-   IPC namespace
        [*]   User namespace
        [*]   PID Namespaces
        -*-   Network namespace
[*]	GCC plugins
[*] Enable loadable module support --->
	[*]	Module unloading
	[*]	Module versioning support
	
[*] Enable the block layer  --->
    [*]   Block layer bio throttling support
    -*- IO Schedulers  --->
        [*]   CFQ IO scheduler
            [*]   CFQ Group Scheduling support 
Kernel Features	--->  
	[*]	Enable seccomp to safely computer untrusted bytecode
	[*] Networking support  --->
	      Networking options  --->
		     <*> IP: ESP transformation
	        [*] Network packet filtering framework (Netfilter)  --->
	            [*] Advanced netfilter configuration
	            [*]  Bridged IP/ARP packets filtering
                Core Netfilter Configuration  --->
                  <*> Netfilter connection tracking support
                  <*> DCCP protocol connection tracking support
                  <*> SCTP protocol connection tracking support
                  <*> UDP-Lite protocol connection tracking support
                  <*> Amanda backup protocol support
                  <*> FTP protocol support
                  <*> H.323 protocol support
                  <*> IRC protocol support
                  <*> PPtP protocol support
                  <*> SIP protocol support
                  <*> TFTP protocol support 
                  <*> Netfilter nf_tables support
                  <*>   Netfilter nf_tables mixed IPv4/IPv6 tables support                             
                  <*>   Netfilter nf_tables netdev tables support                                   
                  <*>   Netfilter nf_tables IPv6 exthdr module                                         
                  <*>   Netfilter nf_tables meta module                                               
	              <*>   Netfilter nf_tables number generator module                                 
			      <*>   Netfilter nf_tables conntrack module                                          
                  <*>   Netfilter nf_tables rbtree set module                                        
                  <*>   Netfilter nf_tables hash set module                                           
                  <*>   Netfilter nf_tables counter module                                             
                  <*>   Netfilter nf_tables log module                                                                      		
                  <*>   Netfilter nf_tables limit module
                  <*>   Netfilter nf_tables masquerade support                                        
                  <*>   Netfilter nf_tables redirect support                                           
                  <*>   Netfilter nf_tables nat module                                                                      
                  <*>   Netfilter nf_tables quota module                                               
                  <*>   Netfilter nf_tables reject support                                              
                  <*>   Netfilter x_tables over nf_tables module                                               
                  <*>   Netfilter nf_tables hash module                                      
                  <*>   Netfilter packet duplication support                                                             
                  <*>   Netfilter nf_tables netdev packet duplication support                          	                    
                  <*>   Netfilter nf_tables netdev packet forwarding support 
                  *** Xtables matches ***
                  <*>   "addrtype" address type match support
                  <*>   "conntrack" connection tracking match support
                  <M>   "ipvs" match support
            <M> IP virtual server support  --->
                  *** IPVS transport protocol load balancing support ***
                  [*]   TCP load balancing support
                  [*]   UDP load balancing support
                  *** IPVS scheduler ***
                  <M>   round-robin scheduling
                  [*]   Netfilter connection tracking
                IP: Netfilter Configuration  --->
                  <*> IPv4 connection tracking support (required for NAT)
                  <*> IP tables support (required for filtering/masq/NAT)
                  <*>   Packet filtering
                  <*>   IPv4 NAT
                  <*>     IPv4 masquerade support
                  <*>   iptables NAT support  
                  <*>     MASQUERADE target support
                  <*>     NETMAP target support
                  <*>     REDIRECT target support
			  IPv6: Netfilter Configuration -->
				   <*>	Ipv6 connection tracking support
				   <M>	ip6tables NAT support
				   <M>		MASQUERADE target support
				   <M>		NPT (Network Prefix translation) target support
        <*> 802.1d Ethernet Bridging
		<*>	802.1Q/802.1ad VLAN Support
        [*] QoS and/or fair queueing  ---> 
            <*>   Control Group Classifier
        [*] L3 Master device support
        [*] Network priority cgroup
        -*- Network classid cgroup
Device Drivers  --->
    [*] Multiple devices driver support (RAID and LVM)  --->
        <*>   Device mapper support
        <*>     Thin provisioning target
    [*] Network device support  --->
        [*]   Network core driver support
        <M>     Dummy net driver support
        <M>     MAC-VLAN support
        <M>     IP-VLAN support
        <M>     Virtual eXtensible Local Area Network (VXLAN)
        <*>     Virtual ethernet pair device
    Character devices  --->
        -*- Enable TTY
        -*-   Unix98 PTY support
File systems  --->
	[*]	Ext3 POSIX Access Control Lists
	[*]	Ext3 Security Labels
	<*> Btrfs filesystem support
	[*]		Btrfs POSIX Access Control Lists
	[*]	Inotify support for userspace
	<*>	FUSE (Filesystem in Userspace) support
    <*> Overlay filesystem support 
    Pseudo filesystems  --->
        [*] HugeTLB file system support
    [*]	Miscellaneous filesystems --->
	    <*>	Apple Extended HFS file system support
	    [*]		HFS+ POSIX Access Control Lists
	[*]	Network File Systems
		<*>	NFS client support
Kernel hacking	--->
	[*]	Sample kernel code 	--->
		<M>		Build seccomp sample code -- loadable modules only
Security options  --->
    -*- Enable access key retention support
    [*]   Enable register of persistent per-UID keyrings
    <M>   ENCRYPTED KEYS
    [*]   Diffie-Hellman operations on retained keys
-*-	Cryptographic API	--->
	<*>	Diffie-Hellman algorithm
	<*>	Deflate compression algorithm
	
```
4. Configure the Root file system:
```bash
$ petalinux-config -c rootfs
```

Once again it is important you include all of the [dependencies required by Docker](https://docs.docker.com/install/linux/docker-ce/binaries/#prerequisites) including **update-rc.d** in order to enable running the Docker daemon at boot:

```
Filesystem Packages  --->
    base  --->
        procps  --->
            [*] procps
        update-rc.d  --->
            [*] update-rc.d
        xz  --->
            [*] xz
            [*] liblzma
    console  --->
        utils  --->
            git  --->
                [*] git
	libs --->
		network	-->
			openssl	-->
				[*]	openssl
				[*]	libcrypto
				[*]	libssl
    misc  --->
        iptables  -->
            [*] iptables
```

Here are some other packages I recommend you include:

```
Filesystem Packages  --->
    admin  --->
        sudo  --->
            [*] sudo
    base  --->
        shell  --->
            [*] bash
        tar  --->
            [*] tar
    console  --->
        network  --->
            curl  --->
                [*] curl
            nfs-utils  --->
                [*] nfs-utils
                [*] nfs-utils-client
            wget  --->
                [*] wget
        utils  --->
            findutils  -->
                [*] findutils
            git  --->
                [*] git
            grep --->
                [*] grep
            vim  --->
                [*] vim
                [*] vim-vimrc
                [*] vim-tools
                [*] vim-common
                [*] vim-syntax
     devel  --->
         python  --->
             python  --->
                 [*] python
                 [*] libpython2
                 [*] python-tkinter
             python-smartpm  --->
                 [*] python-smartpm
                 [*] python-smartpm-interface-images
                 [*] smartpm                  
     misc  --->
         packagegroup-core-buildessential    --->
             [*] packagegroup-core-buildessential
        
```

**Save** and **exit**.


5. Finally it is time to build! This step can be somewhat time consuming especially the first time before Petalinux has had the chance to chance any of the outputs:
```bash
$ petalinux-build
```
6. We can format the **SD Card** while we wait for Petalinux to finish.  I am not sure what then actual minimum size of the SD Card can be but the more space the better, for this walk-through I used a 64 GB PNY SD Card.  You must format the SD Card as follows:
```
Partition Size   Type   LABEL
1         256M   FAT32  BOOT
2         REST   EXT4   Rootfs
```
The easiest way to do this on Ubuntu Linux is with the [gparted GUI](https://gparted.org/):
```bash
$ sudo gparted <Disk name>
# disk name can be found on the command line with lsblk
```
This will bring up a graphical menu representing the current contents of the disk.  Unmount and delete **all** current partitions by **right clicking** them and selecting the corresponding menu options. You should now have only **unallocated** space.  **Right click** the unallocated space and select **new**.  Here is what you should enter for your first **BOOT** partition:
```
Free space preceding (MiB): [<Leave alone>]   Create as:       [Primary Partition]
New size (MiB):             [256          ]   Partition name : [<Leave alone>    ]
Free space following (MiB): [<Leave alone>]   File system:     [fat32            ]
Align to:                   [<Leave alone>]   Label:           [BOOT             ]
```
Select **Add** when you are finished entering the correct parameters.  Now create the **second** partition by **right clicking** the unallocated space and clicking **new**:
```
Free space preceding (MiB): [<Leave alone>]   Create as:       [Primary Partition]
New size (MiB):             [<Leave alone>]   Partition name : [<Leave alone>    ]
Free space following (MiB): [<Leave alone>]   File system:     [ext4             ]
Align to:                   [<Leave alone>]   Label:           [rootfs           ]
```
Select **Add** to finish the second partition.  To finalize the changes click the **Edit** drop-down menu from the menu bar up top and select **Apply all operations**, select **Apply** and allow **gparted** to take care of the rest.

7. Once ```petalinux-build``` you need to is package the outputs for use:
```bash
$ petalinux-package --boot --u-boot --fsbl images/linux/zynqmp_fsbl.elf  --fpga <path to *.bit>/zcu102_base_trd_wrapper.bit --force
```
8. Copy the outputs to the respective partitions ```BOOT``` and ```rootfs``` on the sd card:
```bash
$ cp images/linux/BOOT.BIN <sd card>/BOOT
$ cp images/linux/image.ub <sd card>/BOOT
$ cp images/linux/rootfs.cpio.gz <sd card>/rootfs
```
9. Unzip ```rootfs.cpio.gz``` and expand the contents of the ```rootfs.cpio``` file in ```rootfs``
```bash
$ cd <sd card>/rootfs
$ gzip -d rootfs.cpio.gz
$ pax -rvf rootfs.cpio
``` 
10. Unmount your sd card and insert it into your Zynq ZCU102 and boot it up!
**NOTE:** don't forget your boot switches --> **1 up, 2 down, 3 down, 4 down** on Zynq ZCU102 Rev 1.1

Proceed to the next step to install Docker on the Zynq ZCU102.

## Installing Docker on an Air-Gapped 64-bit Linux Machine

##### Requirements

 1. [docker-\<version\>-ce.tgz for your architecture (ex. x86_64, aarch64 ~30MB)](https://download.docker.com/linux/static/stable/)
 2. [docker_daemon service script (4.0 KB)](https://gist.githubusercontent.com/JamesAnthonyLow/aa3a672915fa9b7ae516b7e5d16e53b3/raw/018294c309c30340728ef96efe05e77986bdd88c/docker_daemon)
 3. [check-config.sh (12.0 KB)](https://raw.githubusercontent.com/moby/moby/master/contrib/check-config.sh)
 4. [cgroupfs-mount (4.0 KB) *If needed](https://raw.githubusercontent.com/tianon/cgroupfs-mount/master/cgroupfs-mount)
 5. [update-rc.d (already installed)](http://manpages.ubuntu.com/manpages/xenial/en/man8/update-rc.d.8.html)
 
 
 ##### Instructions (all steps can be performed as root without the ```sudo``` command)
 
 
 1. First check to make sure that your kernel satisfies the requirements for Docker by running check-config.sh:
```bash
$ chmod a+x ./check-config.sh && ./check-config.sh
```
The output should pass for all features in the **Generally Necessary** section, however **cgroup hierarchy** may fail but **this is OK we will fix it**.  If you are configuring your own kernel you can use the Docker guide from the [Gentoo Linux wiki](https://wiki.gentoo.org/wiki/Docker)

2. (**If cgroup hierarchy failed initially**) run **cgroupfs-mount** and make sure it causes **cgroup hierarchy** to pass, otherwise you may have bigger issues with kernel compatibility:
```bash
$ chmod a+x cgroupfs-mount && ./cgroupfs-mount && ./check-config.sh
```
3. Untar **docker-\<version\>-ce.tgz**:
```bash
$ tar xvzf docker-<version>-ce.tgz
```
4. Move Docker binaries somewhere into ```/usr/bin``` which should already be in your ```PATH```:
```bash
$ mv docker/* /usr/bin
```
5. Test Docker daemon:
```bash
$ dockerd&
$ docker info # should report a successful connection
```
6. (**If cgroup hierarchy failed initially**) move the **cgroupfs-mount** script into ```/etc/init.d``` and enable using **update-rc.d**:
```bash
$ chmod a+x cgroupfs-mount && mv cgroupfs-mount /etc/init.d
$ update-rc.d cgroupfs-mount defaults 99 # ensure it runs before 100
```
7. Move the **docker_daemon service script** into ```/etc/init.d``` and enable using **update-rc.d**:
```bash
$ chmod a+x docker_daemon && mv docker_daemon /etc/init.d
$ update-rc.d docker_daemon defaults 100 # ensure it runs after 99
```
8. Reboot and check to see that the Docker daemon started properly on its own:
```bash
$ reboot
# After reboot
$ docker info
```
9. (Optional) enable Docker usage without sudo for a user:
```bash
$ usermod -aG docker <user> # Add user to docker group
# If logged in, log out and log back in again
# Now you should be able to use docker commands without sudo
```  
At this point I recommend going through a Docker tutorial if you are not yet familiar with Docker's use and benefits.  The [Get Started Guide](https://docs.docker.com/get-started/) from the official documentation is pretty great for a start.
