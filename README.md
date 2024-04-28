# README

* Research Paper: [DroidScope: Seamlessly Reconstructing the OS and Dalvik Semantic Views for Dynamic Android Malware Analysis](https://www.usenix.org/conference/usenixsecurity12/technical-sessions/presentation/yan)
* Before you start, please check the original repository of [Droidscope](https://github.com/decaf-project/Droidscope/tree/master).
* Check [Google DECAF group](https://groups.google.com/g/decaf-platform-discuss), if you face any issues.
* Check [Google archive of Droidscope](https://code.google.com/archive/p/decaf-platform/wikis/DroidScope.wiki).
* [Watch My DroidScope Implementation](https://github.com/rahul07bagul/Droidscope/blob/master/videos/Droidscope.mp4)
* [Watch My DECAF Implementation](https://github.com/rahul07bagul/Droidscope/blob/master/videos/DECAF.mp4)
* Credit goes to all authors from original repository.

I implemented this research paper as part of my course and faced many issues while re-implementing this project. That's why I am creating this README file for future students.

<img width=1200 src="https://github.com/rahul07bagul/Droidscope/blob/master/images/Final.png" alt="bench">

## Environment for Droidscope:
* Ubuntu 16.04
* Android-5.0.0_r2  
* Goldfish 3.4  
* Docker   

## Environment for Android Build:
* As the implementation used Android 5, we must also use the same version. I tried using other versions of Android to see if they would work, but they did not.
* I attempted Android 6, 10, and 11 using AOSP, but ultimately you have to use Android 5 as the Droidscope implementation only supports Android 5.
* Since Android 5 is a very old version, we need to set up an environment to build it.
* Android build official documentation: [AOSP](https://source.android.com/setup/build/downloading)
* For building Android 5, read: [Android 5 dependencies](https://source.android.com/docs/setup/start/older-versions)

### We need the following for the build:
* Ubuntu 14.04
* OpenJDK-7
* Storage: Minimum 400 GB (to be safe, ensure you have 1 TB on the host as you will need to copy files from Docker to the host, which will use more storage)
* RAM: Minimum 16 GB (use swap memory if needed)
* If you are considering a Docker Ubuntu 14.04 image:
   * For the next steps, you will need to copy some build folders from the Docker container to the host machine, which will take time and more storage.
   * I tried the Docker image and was unsure where I went wrong, but I encountered errors indicating missing GPU support needed to run the Android emulator.
   * Conclusion: You can build Android using Docker, but you cannot run the emulator from Docker. (This was my experience; your results may vary)
* If you are considering using VirtualBox, which I eventually used when Docker failed to run the emulator:
   * Download the Ubuntu 14.04 ISO image from: [Ubuntu 14.04 ISO Image](https://old-releases.ubuntu.com/releases/14.04.0/)
   * Download and install VirtualBox from: [VirtualBox](https://github.com/seed-labs/seed-labs/blob/master/manuals/vm/seedvm-manual.md)
      * Follow the steps in the manual and use your Ubuntu 14.04 image, allocating 12 GB of memory for the VM.

## Use the below Docker file to set up the Ubuntu 14.04 environment:
[Android Build Environment Ubuntu 14.04 Dockerfile](https://github.com/rahul07bagul/Droidscope/blob/master/android-docker/Dockerfile)
 
## Android Build
  * Create an Android directory using `mkdir android`
  * Follow the steps here: https://github.com/enlighten5/android_build
  * You also have to build the Goldfish kernel; those steps are also in the above repository.

## After a successful build, follow the below steps; these steps are required for the Droidscope build step 3:
 * Create an Android directory on the host (use the `docker cp` command on the host; for more, check Docker documentation)
 * Copy `android/external` from Docker/VM to host `/android`
 * Copy `android/prebuilts` from Docker/VM to host `/android`
 * Copy `android/out` from Docker/VM to host `/android`
 * Create an images directory inside `/android` on the host
    * After building Android from the Android Open Source Project (AOSP), the generated image files are typically located in the "out" directory within your AOSP source tree. Here's where you can find them:
        * Copy System Image: The system image file, which contains the Android operating system, is usually located at `out/target/product/<device_name>/system.img`
        * Copy Ram Disk Image: `out/target/product/<device_name>/ramDisk.img`
        * Copy User Data Image: `out/target/product/<device_name>/userData.img`
        * Copy Cache Image: `out/target/product/<device_name>/cache.img`
    * If the emulator command is running in Docker, it creates a `userData-qemu.img` file automatically:
        * If the emulator command is working: Copy User Data QEMU Image: `out/target/product/<device_name>/userData-qemu.img`
        * If the emulator command didn't work but the Android build is successful, then in the Droidscope container after the below steps, create this file manually: `userData-qemu.img` in the images directory (`touch userData-qemu.img`). 

## Steps to run Droidscope in Docker:
* Download GitHub repository for droidscope.
* Now you can follow the same steps as from the main repository, but they used Docker to build Droidscope; however, I used a VM. You can go with Docker if you are an "expert" in Docker.
* Remember, if you use a VM for Droidscope, then you might need to follow some different steps, I mean don't use Docker commands but use commands from the Dockerfile of Droidscope.
* If your host is Windows, then download Xlaunch and use that with Docker commands for GUI; use GPT for commands.
* If you are on Ubuntu, then follow the below commands as is.

### 1. Build the Docker image
`docker build --network=host -t droidscope /path/to/the/dockerfile`
### 2. Search for the created image:
`sudo docker image ls`
and copy that IMAGE ID
### 3. Start the Docker image
* Here you have to use the path of `prebuilts/external/out/images` file which you copied from the Docker container previously to the host.
* `sudo docker run -it -e DISPLAY -v /PATH/TO/EXTERNAL:/home/developer/android_source/external -v /PATH/TO/PREBUILTS:/home/developer/android_source/prebuilts -v /PATH/TO/OUT:/home/developer/android_source/out -v /PATH/TO/IMAGE:/home/developer/images -v /tmp/.X11-unix:/tmp/.X11-unix -v $HOME/.Xauthority:/home/developer/.Xauthority --net=host IMAGE_ID`
### 4. Build Droidscope
* On the host
  * `docker cp /PATH/TO/Droidscope/ container_Id:/home/developer/Droidscope`
  * If you are using a VM, copy the Droidscope directory to the VM.
* On Docker
  * `cp -a /home/developer/Droidscope/droidscope/ /home/developer/android_source/external/`
  * `cd /home/developer/android_source/external/droidscope/`
  * `sudo ./android-configure.sh`
  * `sudo make -j4`

### 5. Start Droidscope:
`./startDroidScope.sh`

### 6. Now at this point, it will work or not at all.
  * Let's see where you are; if you can see the emulator window and your virtual device has started, then you are there.
  * But if you can't see any window, or if you get any error while running Droidscope, you might need to use the `-no-window` option with the command in `startDroidScope.sh`.
  * Then you will see the QEMU console and virtual device on another side.

### 7. The `help` command will not work here; I don't know why.
  * e.g., command `ps` to list the running processes

### 8. Before using DroidUnpack, you have to follow some extra steps as these are not in the main repository's README.
  * In their `unpacker.h` file, they used a path where the logs get created.
  * The path is `/home/developer/Droidscope/droidscope/DECAF_plugins/old_dex_extractor/out/`
  * but in your container, `old_dex_extractor` is not there, so create those directories.
      * `mkdir old_dex_extractor`
      * `cd old_dex_extractor`
      * `mkdir out`
      * `cd out`
      * `vi stats.json`
  * You are good to go now; you can check issues in the main repository if you don't follow the above steps.
        
## Steps to Use DroidUnpack
### 1. Build unpacker
`./configure --decaf-path=/home/developer/android_source/external/droidscope/ --target=android` then `make`

### 2. Here you can use existing apps that are there, e.g., Hello Jni
  * Using the virtual device, start the Hello Jni app.
  * Use the `ps` command to get the process name.

### 3. Install app  (optional if you want to install another app)
You may need to run `install_uninstall.sh` to install the app needed.  
Or run the following commands before installing the app:  
`adb shell setprop dalvik.vm.dex2oat-filter "interpret-only"`  
`adb shell setprop dalvik.vm.image-dex2oat-filter "interpret-only"`  

To use the script: `./install_uninstall.sh /path/to/your/app.apk 1`

### 4. Load DroidUnpack in Droidscope
`load_plugin /home/developer/android_source/external/droidscope/DECAF_plugin/DroidUnpack/libunpacker.so`

### 5. Run cmd
* `do_hookapitests com.android.hellojni`
* You can find logs in `/home/developer/Droidscope/droidscope/DECAF_plugins/old_dex_extractor/out/`

### 6. I didn't get any logs after running the above commands, I don't know what I missed but if anyone gets logs let me know. There is one file in the droidscope folder that shows what logs look like.

<img width=1200 src="https://github.com/rahul07bagul/Droidscope/blob/master/images/droidscope%20(1).png" alt="bench">

## Learning:
* You might face issues with missing libraries, use chatgpt with errors.
* I don't know which commands I actually used to solve those errors but you will get answers on chatgpt or StackOverflow, else let me know. I have a file with the history of my commands.
* I used decaf-discussion group to find solutions as I was totally a beginner.
* I also contacted the origin repository author and he helped with some steps.
* At the end if you are not getting the screen that I added in the first screenshot, contact me and I will give you the original Android build files that they used in their project but first follow all steps.
