These are preliminary build instructions for Android. Note that ARM and Android integration are still very early and incomplete. This information is for hackers that want to work on the ARM port.

1. setup android ndk standalone tool chain with platform=14 option

    Android NDK can be downloaded from http://developer.android.com/tools/sdk/ndk/index.html
    
    example command to setup standalone tool chain:
    
        ~/android-ndk-r8d/build/tools$ ./make-standalone-toolchain.sh --platform=android-14 --install-dir=/opt/ndk_standalone --ndk-dir=~/android-ndk-r8d


    In case of 64bit linux system, android ndk needs 32bit linux libraries.

        e.g) /lib32/libc.so.6, /usr/lib32/libz.so.1, /usr/lib32/libstdc++.so.6.0.17

    You can simple install at Ubuntu System by 

        apt-get install libc-i386 lib32z1 libstdc++6

2. Download rustc from git repository

        git clone  http://github.com/mozilla/rust.git
    
3. Configure with ndk toolchain path

        mkdir build; cd build

        ../configure --target-triples=arm-unknown-android --android-cross-path=[path of standalone toolchain dir]

4. Build

        make
  
        make install  

    It will copy rustc binaries and libraries into /usr/local (or as defined with --prefix)
    
5. How to cross compile
    
        rustc --target=arm-unknown-android --android-cross-path=[path of standalone toolchain dir] hello.rs
 
6. How to run on Android

    Remount your device for get the read/write permission on /system partition

        adb remount

    Push arm libs and Android GNU STL shared lib

        adb push /usr/local/lib/rustc/arm-unknown-android/libcore-c3ca5d77d81b46c1-0.6.so /system/lib
        adb push /usr/local/lib/rustc/arm-unknown-android/librustrt.so /system/lib
        adb push /usr/local/lib/rustc/arm-unknown-android/libstd-4782a756585a81-0.6.so /system/lib
        e.g) adb push YOUR_NDK_TOOLCHAIN_PATH/arm-linux-androideabi/lib/libgnustl_shared.so /system/lib

    Push your binary

        e.g) adb push hello /system/bin

    Run using adb shell

        adb shell
        cd /system/bin
        ./hello