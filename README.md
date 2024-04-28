# README
Before you start here, please check the original repository of this project.
https://github.com/decaf-project/Droidscope/tree/master

I implemented this research paper as part of my course, as I faced many issues while doing the re-implementation of this project and that's why I am creating this readme file for future students.

## Host environment for Droidscope:
* Ubuntu 16.04
* Android-5.0.0_r2  
* Goldfish 3.4  
* Docker   

## Host environment for Android Build:
* As they used android 5 for implementation, we also have to used same version. I tried to use other version of android to see that it works or not but it didn't.
* I tried Android 6/10/11 using AOSP, but at the end you have to use Android 5 only as Droidscope implementation only supports android 5.
* As android 5 is very old version and to build same we need to setup environment for that.
* Official documentation: https://source.android.com/setup/build/downloading
* As we are building android 5 read: https://source.android.com/docs/setup/start/older-versions

### We need below for build.
* Ubuntu 14.04
* Openjdk-7
* Storage: Minimum 400 GB (safe side on host you must have 1 TB as you have to copy files from docker to host which will use more storage)
* RAM: Minimum 16 GB (Use swap memory if needed)
* If you are considering docker Ubuntu 14.04 image:
   * For next steps you need some folders of build to build droidscope for that you will need to copy some folders from docker container to host machine, which will take time as well as more storage.
   * I tried docker image and as not sure where I missed but I was getting error that I was missing GPU support to run android emulator.
   * Conclusion is you can build android using docker but you can't run emulator from docker. (This was my experience, you can try)
* If you are considering virtual box, which I used finally when docker was not running emulator.
   * Download Ubuntu 14.04 iso image: https://old-releases.ubuntu.com/releases/14.04.0/
   * Download VirtualBox and install: https://github.com/seed-labs/seed-labs/blob/master/manuals/vm/seedvm-manual.md
      * Follow the steps from the manual and use your Ubuntu 14.04 image, give 12 GB memory for VM.

## Use below docker file to setup Ubuntu 14.04 environment:
 
## Android Build
  * Create android directory using `mkdir android`
  * Follow steps from here: https://github.com/enlighten5/android_build
  * You also have to build goldfish kernal, those steps are also in above repository.

## After successful build, follow below steps, these steps are required for droidscope build step 3
 * Create android directory on host (use docker cp command on host, for more check docker documentation)
 * Copy android/external from docker/VM to host /android
 * Copy android/prebuilts from docker/VM to host /android
 * Copy android/out from docker/VM to host /android
 * Create images directory inside /android on host
    * After building Android from the Android Open Source Project (AOSP), the generated image files are typically located in the "out" directory within your AOSP source tree. Here's where you can find them:
        * Copy System Image: The system image file, which contains the Android operating system, is usually located at: out/target/product/<device_name>/system.img
        * Copy Ram Disk Image: out/target/product/<device_name>/ramDisk.img
        * Copy User Data Image: out/target/product/<device_name>/userData.img
        * Copy Cache Image: out/target/product/<device_name>/cache.img
    * If emulator command is running in docker then it creates userData-qemu.img file automatically:
        * If emulator command working: Copy User Data QEMU Image: out/target/product/<device_name>/userData-qemu.img
        * If emulator command didn't work but android build is successful, then in Droidscope container after below steps create this file manually userData-qemu.img in images directory (`touch userData-qemu.img`). 
  
## Steps to run Droidscope in docker: 
* Now you can follow same steps as from main repository but they used Docker to build Droidscope, but I used VM you can go with docker if you are "expert" in docker.
* If your host is windows then download Xlaunch and use that with docker commands for GUI, use gpt for commands.
* If you are on ubuntu then follow the below commands as it is.
### 1. Build the docker image
`docker build --network=host -t droidscope /path/to/the/dockerfile`
### 2. Search the created image:
`sudo docker image ls`
and copy that IMAGE ID
### 3. Start the docker image
* Here you have to use the path of prebuilts/external/out/images file which you copied from docker container previously to host.
`sudo docker run -it -e DISPLAY -v /PATH/TO/EXTERNAL:/home/developer/android_source/external -v /PATH/TO/PREBUILTS:/home/developer/android_source/prebuilts -v /PATH/TO/OUT:/home/developer/android_source/out -v /PATH/TO/IMAGE:/home/developer/images -v /tmp/.X11-unix:/tmp/.X11-unix -v $HOME/.Xauthority:/home/developer/.Xauthority --net=host IMAGE_ID`
### 4. Build Droidscope  
* On host
  * `docker cp /PATH/TO/Droidscope/ container_Id:/home/developer/Droidscope`
* On docker
  * `cp -a /home/developer/Droidscope/droidscope/ /home/developer/android_source/external/`  
  * `cd /home/developer/android_source/external/droidscope/`  
  * `sudo ./android-configure.sh`  
  * `sudo make -j4`  

### 5. Start Droidscope in docker container:
`./startDroidScope.sh`
### 6. help command will not work here, don't know why.  
  * eg. command `ps` to list the running process

## Steps to use DroidUnpack
### 1. Build unpacker
`./condigure --decaf-path=/<PATH_TO_DROIDSCOPE>/ --target=android` then `make`  
### 2. Install app  
You may need to run install_uninstall.sh to install the app needed.  
Or run the following commads before install the app  
`adb shell setprop dalvik.vm.dex2oat-filter "interpret-only"`  
`adb shell setprop dalvik.vm.image-dex2oat-filter "interpret-only"`   
### 3. Load DroidUnpack in Droidscope
`load_plugin DECAF_plugin/DroidUnpack/libunpacker.so`  
### 4. Run cmd
`do_hookapitests procname`
