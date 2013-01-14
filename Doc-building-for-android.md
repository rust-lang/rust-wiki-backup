========================== Steps for Android Target addtion =======================

1. setup android ndk standalone tool chain with platform=14 option

    Android NDK can be downloaded from http://developer.android.com/tools/sdk/ndk/index.html
    
    example command to setup standalone tool chain:
    
       ~/work/toolchains/android-ndk-r8c/build/tools$ ./make-standalone-toolchain.sh --platform=android-14 --install-dir=/home/ubuntu/work/toolchains/ndk_standalone --ndk-dir=/home/ubuntu/work/toolchains/android-ndk-r8c

2. Download rustc from git repository

    a. git clone  http://github.com/webconv/rust.git
    
    b. cd rust
    
    c. mkdir build
    
    d. cd build

3. Create Makefile using CMake

    cmake ../ -DTargetOsType=android -DTargetCpuType=arm -DToolchain=path_of_standalone_toolchain_dir

4. Build libuv and  llvm

    make libuv

    make llvm 


5. Create Makefile again (the Makefile made in step 3 does not contain information of llvm)

    cmake ../

6. Build Rustc ( make and make install have been separated)

    a.make
    
    b.make install  [ it will copy ARM libraries into /usr/local/lib/rustc/arm-unknown-android/lib
    

7. How to cross compiler
    
    rustc --target arm-unknown-android hello.rs
 

8. How to run on Android

    use adb -e push command to push all arm libs as specified in 6 b
   
    push your binary
   
    set LD_LIBRARY_PATH
   
    run using adb shell  
