1,下载ubuntu内核源码(https://kernel.ubuntu.com/~kernel-ppa/mainline/v4.4.115/)和补丁

>git clone git://git.launchpad.net/~ubuntu-kernel-test/ubuntu/+source/linux/+git/mainline-crack -b v4.4.115 v4.4.115

0001-base-packaging.patch
0002-UBUNTU-SAUCE-add-vmlinux.strip-to-BOOT_TARGETS1-on-p.patch
0003-UBUNTU-SAUCE-tools-hv-lsvmbus-add-manual-page.patch
0004-UBUNTU-SAUCE-no-up-disable-pie-when-gcc-has-it-enabl.patch
0005-debian-changelog.patch
0006-configs-based-on-Ubuntu-4.4.0-113.136.patch

和
linux-image-4.4.115-0404115-generic_4.4.115-0404115.201802031230_amd64.deb

和rtai-5.1(https://www.rtai.org/)并解压

>tar Jxf rtai-5.1.tar.bz2

2,获得.config文件

>dpkg -x linux-image-4.4.115-0404115-generic_4.4.115-0404115.201802031230_amd64.deb ./configs
>cp ./configs/boot/config-4.4.115-0404115-generic ./v4.4.115/.config

3,打补丁:

>cd v4.4.115
>patch -p1 < ../0001-base-packaging.patch
>patch -p1 < ../0002-UBUNTU-SAUCE-add-vmlinux.strip-to-BOOT_TARGETS1-on-p.patch
>patch -p1 < ../0003-UBUNTU-SAUCE-tools-hv-lsvmbus-add-manual-page.patch
>patch -p1 < ../0004-UBUNTU-SAUCE-no-up-disable-pie-when-gcc-has-it-enabl.patch
>patch -p1 < ../0005-debian-changelog.patch
>patch -p1 < ../0006-configs-based-on-Ubuntu-4.4.0-113.136.patch
>patch -p1 < ../rtai-5.1/base/arch/x86/patches/hal-linux-4.4.115-x86-10.patch

应该都能打成功

4,配置内核:

>make menuconfig


按以下修改内核配置参数(见:https://github.com/relacs/makertai)

Basic kernel configuration

For making the Linux kernel work with RTAI you should check the following settings in the kernel configuration dialog.

This list is updated for RTAI 5.1. For other RTAI versions read /usr/local/src/rtai/README.CONF_RMRKS !

    "General setup":
        Disable "Enable sytem-call auditing support" (AUDITSYSCALL)
        Important: set "Stack Protector buffer overflow detection" (at the bottom of the menu) to "Regular" (CC_STACKPROTECTOR_REGULAR) - or even "None" (CC_STACKPROTECTOR_NONE) if the latency test crashes.

    "Power management and ACPI options":
        In "ACPI (Advanced Configuration and Power Interface) Support":
            Disable "Processor" (ACPI_PROCESSOR)
        Disable "CPU Frequency scaling" (CPU_FREQ)
        In "CPU Idle":
            Disable "CPU idle PM support" (CPU_IDLE)

    "Device Drivers":
        In "Staging drivers":
            Deselect "Data acquisition support (comedi)" (COMEDI)

    "Kernel hacking":
        In "Compile-time checks and compiler options":
            Disable "Compile the kernel with debug info" (DEBUG_INFO) Disabling debugging information makes the kernel much smaller. So unless you know that you need it disable it.
        Disable "Tracers" (FTRACE)

Leave the configuration dialog by pressing "Exit" until you are asked "Save kernel config?". Select "Yes".

Then the new kernel is being compiled - be patient.


5,编译:

>make -j8 deb-pkg

6,安装:

>sudo dpkg -i linux-headers-4.4.115-rtai-5.1+_4.4.115-rtai-5.1+-1_amd64.deb
>sudo dpkg -i linux-image-4.4.115-rtai-5.1+_4.4.115-rtai-5.1+-1_amd64.deb

一般都能成功启动新内核。

7,编译rtai-5.1库和驱动

>cd  ../rtai-5.1
>make menuconfig
-----------------------
General --->
 (/opt/rtai-5.1_v4.4.115) Installation directory                                         
 (/usr/src/linux-headers-4.4.115-rtai-5.1+) Linux source tree
--------------------------------------------------------------
>make
>sudo make install
>cd /opt/rtai-5.1_v4.4.115/modules
>sudo insmod rtai_hal.ko
>sudo insmod rtai_sched.ko  //如果到这儿没死机就说明rtai安装成功了
>sudo insmod rtai_fifos.ko
>sudo insmod rtai_sem.ko
>sudo insmod rtai_shm.ko
>sudo insmod rtai_rtdm.ko



