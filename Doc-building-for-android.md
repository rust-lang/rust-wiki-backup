Rust has been ported to Android OS running on ARM architecture. Android on other architectures is not yet supported.

In the issue tracker, [Android-related issues](https://github.com/mozilla/rust/issues?labels=A-android) are labelled `A-android`, and [ARM-related issues](https://github.com/mozilla/rust/issues?labels=A-ARM) are labelled `A-ARM`.

Instructions to build a cross compiler to Android on ARM, to cross compile Rust sources, and to run resulting binaries follow.

1. Setup android ndk standalone tool chain with platform=14 option

    Download android NDK version _r9b_ or later (you might not be able to build jemalloc with previous version) from http://developer.android.com/tools/sdk/ndk/index.html

```
wget -c http://dl.google.com/android/ndk/android-ndk-r9b-linux-x86_64.tar.bz2
```
    
    example command to setup standalone tool chain:
    
        ~/android-ndk-r9b/build/tools/make-standalone-toolchain.sh --platform=android-14 --install-dir=/opt/ndk_standalone --ndk-dir=~/android-ndk-r9b


    In case of 64bit linux system, android ndk needs 32bit linux libraries.

        e.g) /lib32/libc.so.6, /usr/lib32/libz.so.1, /usr/lib32/libstdc++.so.6.0.17

    You can simple install at Ubuntu System by 

        apt-get install libc6-i386 lib32z1 lib32stdc++6

2. Download rustc from git repository

        git clone  http://github.com/mozilla/rust.git
    
3. Configure with ndk toolchain path

        mkdir build; cd build

        ../configure --target=arm-linux-androideabi --android-cross-path=[path of standalone toolchain dir]

4. Build

        make
  
        make install (Optional) 

    It will copy rustc binaries and libraries into /usr/local (or as defined with --prefix)
    
5. How to cross compile
    
        rustc --target=arm-linux-androideabi -C android-cross-path=[path of standalone toolchain dir] hello.rs
 
6. How to run on Android

    Remount your device for get the read/write permission on /system partition

        adb remount

    Push your binary

        adb push hello /system/bin
        chmod 777 hello

    Run using adb shell

        adb shell /system/bin/hello

Use rustc with --crate-type=staticlib etc to emit rust code which can be linked with C/C++ sources compiled by the android standalone toolchain's g++. From here is it possible to create an APK as with NDK examples. Use #[no_mangle] extern "C" rust functions to export functions which can be called by android frameworks

Sample modification to android-ndk native-activity sample makefile (jni/Android.mk)

    LOCAL_PATH := $(call my-dir)
    include $(CLEAR_VARS)

    LOCAL_MODULE    := rust-prebuilt
    LOCAL_SRC_FILES := librust_android.a

    include $(PREBUILT_STATIC_LIBRARY)
    include $(CLEAR_VARS)

    LOCAL_MODULE    := native-activity
    LOCAL_SRC_FILES := main.c
    LOCAL_LDLIBS    := -llog -landroid -lEGL -lGLESv1_CM
    LOCAL_STATIC_LIBRARIES := android_native_app_glue rust-prebuilt

    include $(BUILD_SHARED_LIBRARY)

    $(call import-module,android/native_app_glue)

This requires that you compiled rust code into a library beforehand:-

    rustc --target=arm-linux-androideabi hello_android.rs --android-cross-path=/opt/ndk-standalone-arm/ --staticlib -o jni/librust_android.a

execute ndk-build, ant debug|release etc to compile it into a native code .so and package,deploy it