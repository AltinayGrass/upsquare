# upsquare
5.15 preemth_rt kernel config for upsquare 

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
   sudo apt install make gcc libncurses-dev libssl-dev flex libelf-dev bison
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
   ```
7. Build the kernel (note: this can take some time).
   ```
   sudo make clean
   sudo make -j4 
   ```
8. Install the kernel modules.
   `sudo make -j4 modules_install`
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
