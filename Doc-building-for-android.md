These are preliminary build instructions for Android. Note that ARM and Android integration are still very early and incomplete. Some [features](https://github.com/mozilla/rust/issues?labels=A-android&page=1&state=open) do not work yet, and as such this information is primarily for hackers that want to work on the ARM port.

1. setup android ndk standalone tool chain with platform=14 option

    Android NDK can be downloaded from http://developer.android.com/tools/sdk/ndk/index.html
    
    example command to setup standalone tool chain:
    
        ~/android-ndk-r8d/build/tools/make-standalone-toolchain.sh --platform=android-14 --install-dir=/opt/ndk_standalone --ndk-dir=~/android-ndk-r8d


    In case of 64bit linux system, android ndk needs 32bit linux libraries.

        e.g) /lib32/libc.so.6, /usr/lib32/libz.so.1, /usr/lib32/libstdc++.so.6.0.17

    You can simple install at Ubuntu System by 

        apt-get install libc6-i386 lib32z1 lib32stdc++6

2. Download rustc from git repository

        git clone  http://github.com/mozilla/rust.git
    
3. Configure with ndk toolchain path

        mkdir build; cd build

        ../configure --target-triples=arm-linux-androideabi --android-cross-path=[path of standalone toolchain dir]

4. Build

        make
  
        make install (Optional) 

    It will copy rustc binaries and libraries into /usr/local (or as defined with --prefix)
    
5. How to cross compile
    
        rustc --target=arm-linux-androideabi --android-cross-path=[path of standalone toolchain dir] hello.rs
 
6. How to run on Android

    Remount your device for get the read/write permission on /system partition

        adb remount

    Push arm libs and Android GNU STL shared lib

        adb push NDK_TOOLCHAIN_PATH/arm-linux-androideabi/lib/libgnustl_shared.so
        adb push /usr/local/lib/rustc/arm-unknown-android/libextra-a7c050cfd46b2c9a-0.7.so /system/lib
        adb push /usr/local/lib/rustc/arm-unknown-android/librustrt.so /system/lib
        adb push /usr/local/lib/rustc/arm-unknown-android/libstd-6c65cf4b443341b1-0.7.so /system/lib
        e.g) adb push YOUR_NDK_TOOLCHAIN_PATH/build/x86_64-unknown-linux-gnu/stage2/lib/rustc/arm-linux-androideabi/liblibrustrt.so /system/lib

    Push your binary

        adb push hello /system/bin
        chmod 777 hello

    Run using adb shell

        adb shell /system/bin/hello