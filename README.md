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
