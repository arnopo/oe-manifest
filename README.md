# oe-manifest for OpenAMP demo
This project is the repo manifest of OpenSTLinux release adapted for the OpenAMP demo

## STM32MP15-Ecosystem-v3.0.0 release TAG: openstlinux-5.10-dunfell-mp1-21-03-31
See the wiki user guide for more information: http://wiki.st.com/stm32mpu/index.php/OpenSTLinux_distribution

## Updates vs ST distribution:
*  add poky layer
*  new reference for following layers

    * [meta-zephyr](https://github.com/arnopo/meta-zephyr)
    * [meta-st-stm32mp-addons](https://github.com/arnopo/meta-st-stm32mp-addons)

## Hardware supported
*  stm32mp157C-DK2 board
*  stm32mp157F-DK2 Board

# How to build demos:
## prerequisite:
The installation relies on the repo command. In case the Repo tool (a Google-built repository management tool that runs on top of git) is not yet installed and configured on the host PC, refer to the [PC prerequisites article](https://wiki.st.com/stm32mpu/wiki/PC_prerequisites).

## Installing
* Create the OpenSTLinux distribution installation directory:

    ```
    $ mkdir -p stm32mp157-openamp
    $ cd stm32mp157-openamp
    ```
* Initialize repo in the current directory and synchronize.
    ```
    $ repo init -u https://github.com/arnopo/oe-manifest.git
    $ repo sync
    ```

## Building the OpenSTLinux distribution

###  Initializing the OpenEmbedded build environment
The OpenEmbedded environment setup script must be run once in each new working terminal in which you use the BitBake or devtool tools (see later) from the installation directory:
   ```
   $ MACHINE=stm32mp1-openamp DISTRO=openstlinux-weston source layers/meta-st/scripts/envsetup.sh
   ```
###  Building the OpenSTLinux distribution

   ```
   $ bitbake st-image-core
   ```

Note that: 

- to build around 30 GB is needed
- building the distribution can take several hours depending on performance of the PC.

### Flashing the image

Two possibilities:

* Install and use the STM32CubeProgrammer tool
Faster and more adapted for the stm32mp157F-DK2 board support
See the wiki user guide for more information: https://wiki.st.com/stm32mpu/wiki/STM32MP1_Distribution_Package#Flashing_the_built_image

* Populate the sdcard using "dd" command line

The wic image is built for the stm32mp157C-DK2 but is compatible with the stm32mp157F-DK2 board ( with Cortex-A7 frequency limited to 650 MHZ)
```
cd tmp-glibc/deploy/images/stm32mp1-openamp
sudo dd of=/dev/sdb iflag=fullblock oflag=direct conv=fsync status=progress bs=20M if=st-image-core-openstlinux-weston-stm32mp1-openamp.wic
```
* Populate the sdcard using "pv" command line

Similar to "dd" command but with a progress bar.
```
cd tmp-glibc/deploy/images/stm32mp1-openamp
sudo pv -r -p <st-image-core-openstlinux-weston-stm32mp1-openamp.wic >/dev/sdb
```
## Building the Zephyr distribution

### Initializing the OpenEmbedded build environment
* The OpenEmbedded environment setup script must be run once in each new working terminal in which you use the BitBake or devtool tools (see later) from the installation directory:
   ```
   $ MACHINE="stm32mp157c-dk2" DISTRO="zephyr" source layers/poky/oe-init-build-env build-zephyr
   ```

   Add layers ( To do one time before the first building):
   ```
   $ MACHINE="stm32mp157c-dk2" DISTRO="zephyr" source layers/poky/oe-init-build-env build-zephyr
   $ bitbake-layers add-layer ../layers/meta-openembedded/meta-oe/
   $ bitbake-layers add-layer ../layers/meta-openembedded/meta-python/
   $ bitbake-layers add-layer ../layers/meta-zephyr/
   ```
### Building the OpenSTLinux distribution

   ```
   $ MACHINE="stm32mp157c-dk2" DISTRO="zephyr" bitbake zephyr-openamp-rsc-table
   ```

Note that: 

- to build around 30 GB is needed
- building the distribution can take 1 or 2 hours depending on performance of the PC.

### Install the Zephyr binary on the sdcard
The Zephyr sample binary is available in the sub-folder ./tmp-newlib/deploy/images/stm32mp157c-dk2/
It needs to be installed on the "rootfs" partition of the sdcard

   ```
   $ sudo cp tmp-newlib/deploy/images/stm32mp157c-dk2/zephyr-openamp-rsc-table.elf <mount point>/rootfs/lib/firmware/
   ```
Properly unoumt the sdcard partitions.
   
## Demos

Connect the board to the PC to enable the board console and Boot the board(https://wiki.st.com/stm32mpu/wiki/STM32MP15_Discovery_kits_-_Starter_Package#Booting_the_board)

### Zephyr Demo


#### Linux rpmsg sample client driver demo

The [zephyr-openamp-rsc-table](https://github.com/zephyrproject-rtos/zephyr/tree/main/samples/subsys/ipc/openamp_rsc_table) demonstrates the RPMsg interprocessor communication with the Linux by initiating communication with the Linux rpmsg_client_sample sample. To run the sample embedded in the rootfs partion of the sdcard in /lib/firmware/ directory:

   ```
   root@stm32mp1-openamp:~# echo zephyr-openamp-rsc-table.elf >/sys/class/remoteproc/remoteproc0/firmware 
   root@stm32mp1-openamp:~# echo start >/sys/class/remoteproc/remoteproc0/state 
   ```
On success the following message will appear on the corresponding Zephyr console:

   ```
   rpmsg_client_sample virtio0.rpmsg-client-sample.-1.1024: new channel: 0x400 -> 0x400!
   rpmsg_client_sample virtio0.rpmsg-client-sample.-1.1024: incoming msg 1 (src: 0x400)
   rpmsg_client_sample virtio0.rpmsg-client-sample.-1.1024: incoming msg 2 (src: 0x400)
   rpmsg_client_sample virtio0.rpmsg-client-sample.-1.1024: incoming msg 3 (src: 0x400)
   ...
   rpmsg_client_sample virtio0.rpmsg-client-sample.-1.1024: incoming msg 100 (src: 0x400)
   rpmsg_client_sample virtio0.rpmsg-client-sample.-1.1024: goodbye!
   virtio_rpmsg_bus virtio0: destroying channel rpmsg-client-sample addr 0x400
   rpmsg_client_sample virtio0.rpmsg-client-sample.-1.1024: rpmsg sample client driver is removed
   ```

### STM32Cube Demos

#### Linux rpmsg sample client driver demo
Same demo than the Zephyr openamp_rsc_table but using a stm32Cube firmware.
The demo is described here: https://github.com/STMicroelectronics/STM32CubeMP1/tree/master/Projects/STM32MP157C-DK2/Applications/OpenAMP/OpenAMP_raw/readme.txt
   
To start the demo:
   ```
   root@stm32mp1-openamp:~# cd /usr/local/Cube-M4-examples/STM32MP157C-DK2/Applications/OpenAMP/OpenAMP_raw/
   root@stm32mp1-openamp:~# ./fw_cortex_m4.sh start
   ```

#### RPMSG TTY demo
The demo is described here: https://github.com/STMicroelectronics/STM32CubeMP1/blob/master/Projects/STM32MP157C-DK2/Applications/OpenAMP/OpenAMP_TTY_echo/readme.txt

To start the demo:
   ```
   root@stm32mp1-openamp:~# cd /usr/local/Cube-M4-examples/STM32MP157C-DK2/Applications/OpenAMP/OpenAMP_TTY_echo/
   root@stm32mp1-openamp:~# ./fw_cortex_m4.sh start
   ```
