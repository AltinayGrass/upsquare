# UP Squared
5.15 preempt_rt kernel config for UP Squared from Aaeon 

1. Great working directory
   ```
   mkdir ~/kernel
   cd ~/kernel
   ```
2. Download the kernel source files and real-time patch files for your specific Linux kernel version from kernel.org.
   ```
   wget https://mirrors.edge.kernel.org/pub/linux/kernel/v5.x/linux-5.15.148.tar.gz
   wget https://mirrors.edge.kernel.org/pub/linux/kernel/projects/rt/5.15/patch-5.15.148-rt74.patch.xz
   ```
3. Unpack the source files.
   ```
   tar -xzf linux-5.15.148.tar.gz
   xz -d patch-5.15.148-rt74.patch.xz
   cd linux-5.15.148
   patch -p1 <../patch-5.15.148-rt74.patch
   ```
4. Configure the kernel build options and install package dependencies.
   ```
   cp /boot/config-5.15.0-97-generic .config
   sudo apt update
   sudo apt install make gcc libncurses-dev libssl-dev flex libelf-dev bison zstd build-essential libnuma-dev
   make menuconfig
   ```
5. Activate “Fully Preemptible Kernel (Real-Time)” option from “General setup” / “Preemption Model” then SAVE and EXIT.
   
   <img width="662" alt="linux-kernel-config-fully-preemptible" src="https://github.com/AltinayGrass/upsquare/assets/97592357/eff53f19-4393-4e0b-850e-14c5c029fe18">

6. Sure about below parameters.
   ```
   CONFIG_HIGH_RES_TIMERS=y
   CONFIG_CPU_ISOLATION=y
   CONFIG_SYSTEM_TRUSTED_KEYS=""
   CONFIG_SYSTEM_REVOCATION_KEYS=""
   CONFIG_MODULE_SIG=n
   CONFIG_MODULE_SIG_ALL=n
   CONFIG_MODULE_SIG_FORCE=n
   CONFIG_MODULE_SIG_KEY=""
   #CONFIG_X86_X32 is not set
   CONFIG_R8169=m
   ```
7. Build the kernel (note: this can take some time).
   ```
   sudo make clean
   sudo make -j4 
   ```
8. Install the kernel modules.
      ```
      sudo make -j4 modules_install
      ```
9. Install the kernel.
      ```
      sudo make install
      ```
10. Add kernel command line parameters.
      ```
      sudo nano /etc/default/grub
      ```
      ```
      GRUB_CMDLINE_LINUX_DEFAULT="quiet splash clocksource=tsc tsc=reliable nmi_watchdog=0 idle=pull hpet=disable irqaffinity=0-2 isolcpus=3 i915.enable_dc=0 i915.disable_power_well=0 processor.max_cstate=0 intel_idle.max_cstate=0 intel_pstate=disable"
      ```
      ```
      sudo update-grub
      ```

    ![plot](https://github.com/AltinayGrass/upsquare/assets/97592357/ad0a71a7-bf85-413b-8327-753bfe6441b3)

11. Setup sources for the EtherCAT Master.
      ```
      git clone https://gitlab.com/etherlab.org/ethercat.git
      cd ethercat
      git checkout stable-1.5
      sudo rm /usr/bin/ethercat
      sudo rm /etc/init.d/ethercat
      ./bootstrap  # to create the configure script
      ```
12. Configure, build and install libs and kernel modules.
      ```
      ./configure --prefix=/usr/local/etherlab  --disable-8139too --disable-eoe --enable-generic --enable-r8169 --enable-hrtimer
      make all modules
      sudo make modules_install install
      sudo depmod
      ```
13. Configure system.
      ```
      sudo ln -s /usr/local/etherlab/bin/ethercat /usr/bin/
      sudo ln -s /usr/local/etherlab/etc/init.d/ethercat /etc/init.d/ethercat
      sudo mkdir -p /etc/sysconfig
      sudo cp /usr/local/etherlab/etc/sysconfig/ethercat /etc/sysconfig/ethercat
      ```
14. Create a new udev rule.
      ```
      sudo gedit /etc/udev/rules.d/99-EtherCAT.rules
      ```
      containing:
      ```
      KERNEL=="EtherCAT[0-9]*", MODE="0666"
      ```
15. Configure the network adapter for EtherCAT.
      ```
      sudo gedit /etc/sysconfig/ethercat
      ```
      containing:
      ```
      MASTER0_DEVICE="xx:xx:xx:xx:xx:d6"
      DEVICE_MODULES="r8169"
      ```
16. Now you can start the EtherCAT master.
      ```
      sudo /etc/init.d/ethercat start
      ```
      it should print 
      
      `Starting EtherCAT master 1.5.2  done`
   
      Then you can run servo positioning with ethercat servo drive
   
      `taskset -c 3 sudo chrt --rr 99 ./servo_pos 3000`
