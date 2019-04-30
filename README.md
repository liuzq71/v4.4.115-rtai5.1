1,下载ubuntu内核源码(https://kernel.ubuntu.com/~kernel-ppa/mainline/v4.4.115/)和补丁 <br>

$git clone git://git.launchpad.net/~ubuntu-kernel-test/ubuntu/+source/linux/+git/mainline-crack -b v4.4.115 v4.4.115 <br>

0001-base-packaging.patch<br>
0002-UBUNTU-SAUCE-add-vmlinux.strip-to-BOOT_TARGETS1-on-p.patch <br>
0003-UBUNTU-SAUCE-tools-hv-lsvmbus-add-manual-page.patch <br>
0004-UBUNTU-SAUCE-no-up-disable-pie-when-gcc-has-it-enabl.patch <br>
0005-debian-changelog.patch <br>
0006-configs-based-on-Ubuntu-4.4.0-113.136.patch <br>

和 <br>
linux-image-4.4.115-0404115-generic_4.4.115-0404115.201802031230_amd64.deb <br>

和rtai-5.1(https://www.rtai.org/)并解压 <br>

$tar Jxf rtai-5.1.tar.bz2 <br>

2,获得.config文件 <br>

$dpkg -x linux-image-4.4.115-0404115-generic_4.4.115-0404115.201802031230_amd64.deb ./configs <br>
$cp ./configs/boot/config-4.4.115-0404115-generic ./v4.4.115/.config <br>

3,打补丁: <br>

$cd v4.4.115<br>
$patch -p1 < ../0001-base-packaging.patch<br>
$patch -p1 < ../0002-UBUNTU-SAUCE-add-vmlinux.strip-to-BOOT_TARGETS1-on-p.patch<br>
$patch -p1 < ../0003-UBUNTU-SAUCE-tools-hv-lsvmbus-add-manual-page.patch<br>
$patch -p1 < ../0004-UBUNTU-SAUCE-no-up-disable-pie-when-gcc-has-it-enabl.patch<br>
$patch -p1 < ../0005-debian-changelog.patch<br>
$patch -p1 < ../0006-configs-based-on-Ubuntu-4.4.0-113.136.patch<br>
$patch -p1 < ../rtai-5.1/base/arch/x86/patches/hal-linux-4.4.115-x86-10.patch<br>

应该都能打成功<br>

4,配置内核:<br>

$make menuconfig<br>


按以下修改内核配置参数(见:https://github.com/relacs/makertai)<br>

Basic kernel configuration<br>

For making the Linux kernel work with RTAI you should check the following settings in the kernel configuration dialog.<br>

This list is updated for RTAI 5.1. For other RTAI versions read /usr/local/src/rtai/README.CONF_RMRKS !<br>

    "General setup":<br>
        Disable "Enable sytem-call auditing support" (AUDITSYSCALL)<br>
        Important: set "Stack Protector buffer overflow detection" (at the bottom of the menu) to "Regular" (CC_STACKPROTECTOR_REGULAR) - or even "None" (CC_STACKPROTECTOR_NONE) if the latency test crashes.<br>

    "Power management and ACPI options":<br>
        In "ACPI (Advanced Configuration and Power Interface) Support":<br>
            Disable "Processor" (ACPI_PROCESSOR)<br>
        Disable "CPU Frequency scaling" (CPU_FREQ)<br>
        In "CPU Idle":<br>
            Disable "CPU idle PM support" (CPU_IDLE)<br>

    "Device Drivers":<br>
        In "Staging drivers":<br>
            Deselect "Data acquisition support (comedi)" (COMEDI)<br>

    "Kernel hacking":<br>
        In "Compile-time checks and compiler options":<br>
            Disable "Compile the kernel with debug info" (DEBUG_INFO) Disabling debugging information makes the kernel much smaller. So unless you know that you need it disable it.<br>
        Disable "Tracers" (FTRACE)<br>

Leave the configuration dialog by pressing "Exit" until you are asked "Save kernel config?". Select "Yes".<br>

Then the new kernel is being compiled - be patient.<br>


5,编译:<br>

$make -j8 deb-pkg<br>

6,安装:<br>

$sudo dpkg -i linux-headers-4.4.115-rtai-5.1+_4.4.115-rtai-5.1+-1_amd64.deb <br>
$sudo dpkg -i linux-image-4.4.115-rtai-5.1+_4.4.115-rtai-5.1+-1_amd64.deb <br>
$sudo reboot<br>

一般都能成功启动新内核。<br>

7,编译rtai-5.1库和驱动<br>

$cd  ../rtai-5.1<br>
$make menuconfig<br>
<br>
General ---><br>
  (/opt/rtai-5.1_v4.4.115) Installation directory <br>                                        
  (/usr/src/linux-headers-4.4.115-rtai-5.1+) Linux source tree <br>
<br>
$make<br>
$sudo make install<br>
$cd /opt/rtai-5.1_v4.4.115/modules<br>
$sudo insmod rtai_hal.ko<br>
$sudo insmod rtai_sched.ko  //如果到这儿没死机就说明rtai安装成功了<br>
$sudo insmod rtai_fifos.ko<br>
$sudo insmod rtai_sem.ko<br>
$sudo insmod rtai_shm.ko<br>
$sudo insmod rtai_rtdm.ko<br>



