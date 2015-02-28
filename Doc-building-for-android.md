Rust has been ported to Android OS running on ARM architecture. Android on other architectures is not yet supported.

**Important**: Right now, this only works correctly on linux. It can be made to work on OSX, but if you're on OSX, make things easy by setting up a VM. It has been successfully tested on ubuntu 14.04.2 in virtualbox, if you want something super safe, but most any debian-based thing will almost certainly work.

In the issue tracker, [Android-related issues](https://github.com/rust-lang/rust/issues?labels=A-android) are labelled `A-android`, and [ARM-related issues](https://github.com/rust-lang/rust/issues?labels=A-ARM) are labelled `A-ARM`.

Instructions to build a cross compiler to Android on ARM, to cross compile Rust sources, and to run resulting binaries follow.

1. Setup Android NDK standalone tool chain with your specific [API Level](http://developer.android.com/guide/topics/manifest/uses-sdk-element.html#ApiLevels)

    Download android NDK version _r9b_ or later (you might not be able to build jemalloc with previous versions) from http://developer.android.com/tools/sdk/ndk/index.html. for example:

        $ wget http://dl.google.com/android/ndk/android-ndk-r10d-linux-x86_64.bin
        $ chmod +x android-ndk-r10d-linux-x86_64.bin
        $ ./android-ndk-r10d-linux-x86_64.bin
        $ ls android-ndk-r10d -d
        android-ndk-r10d
    
    example command to setup standalone tool chain:
    
        ~/android-ndk-<your-version-here>/build/tools/make-standalone-toolchain.sh \
            --platform=android-14 \
            --arch=arm \
            --install-dir=~/ndk-standalone-14-arm
        
    `--platform`    indicates the Android API level - has successfully been tested with 18 as well  
    `--install-dir` indicates the directory where the toolchain will be installed. You can pick
    more or less anything you feel like.  
    `--arch`        must be set to `arm`, or it won't know what toolchain to use. (Imagine that!)  
    `--toolchain`   can also be set to one of the names of the directories in
      `android-ndk-<yourvers>/toolchains/` - keeping in mind it must be one of the arm ones.
      Only do this if you know why you want to - otherwise, the default of ...-4.8 will work fine.

    If you're on 64bit linux, the android ndk needs 32bit linux libraries, such as `/lib32/libc.so.6`, `/usr/lib32/libz.so.1`, and `/usr/lib32/libstdc++.so.6.0.17`, so make sure they're installed.
    If you're using ubuntu 64, this command will do:

        e.g.) 
        

    You can simple install at Ubuntu System by 

        apt-get install libc6-i386 lib32z1 lib32stdc++6

	Some of the tool chain will need to be placed in your path, if you plan to build with cargo.

		e.g.)
		ln -s /opt/ndk_standalone/bin/arm-linux-androideabi-gcc /usr/bin/arm-linux-androideabi-gcc
		ln -s /opt/ndk_standalone/bin/arm-linux-androideabi-ar /usr/bin/arm-linux-androideabi-ar

2. Download rustc from git repository

        git clone https://github.com/rust-lang/rust.git
    
3. Configure with ndk toolchain path

        mkdir build; cd build

        ../configure --target=arm-linux-androideabi --android-cross-path=[path of standalone toolchain dir]

4. Build

        make
  
        make install (Optional) 

    It will copy rustc binaries and libraries into /usr/local (or as defined with --prefix)
    
5. How to cross compile

		With rustc:
	        rustc --target=arm-linux-androideabi -C linker=[path of standalone toolchain dir]/bin/arm-linux-androideabi-gcc hello.rs

		With cargo:
			Create a .cargo directory in your project's root dir
			Create a config file inside the .cargo directory (proj/.cargo/config)
			Place your deps/linker information inside that file
			e.g)
			[target.arm-linux-androideabi]
			linker = "/opt/ndk_standalone/bin/arm-linux-androideabi-gcc"
			
			Compile:
			cargo build --target=arm-linux-androideabi
 
6. How to run on Android

    Remount your device for get the read/write permission on /system partition

        adb remount

    Push your binary

        adb push hello /system/bin
        chmod 777 hello

    Run using adb shell

        adb shell /system/bin/hello


### How to add rust code into an ndk project to create an APK

Use rustc with --crate-type=staticlib etc to emit rust code which can be linked with C/C++ sources compiled by the android standalone toolchain's g++. From here is it possible to create an APK as with NDK examples. Use #[no_mangle] extern "C" rust functions to export functions which can be called by android frameworks

Sample of modifications to the android-ndk "native-activity" sample makefile (jni/Android.mk) :-

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

This requires that you compiled rust code into a library beforehand, eg if 'hello_android.rs' is your crate root

    rustc --target=arm-linux-androideabi hello_android.rs --android-cross-path=/opt/ndk-standalone-arm/ --staticlib -o jni/librust_android.a

execute ndk-build, ant debug|release etc to compile it into a native code .so and package,deploy it

### How to integrate as a Shared Library into Android Studio using NDK and Gradle
1. Edit your Cargo.toml to build as a dylib

		[lib]
		name = "my-awesome-lib"
		crate_type = ["dylib"]
2. Create a `jni` and `jniLibs` directory in your Android Studio project in `proj/app/src/main`
3. Copy your `libmy-awesome-lib-SOME_HASH.so` to `proj/app/src/main/jniLibs` without the hash extension cargo automagically generates
4. Place your C++ shim source in `proj/app/src/main/jni`
5. Since we have no headers to register symbols, you will need to register all of the `#[no_mangle] pub extern` Rust functions in your C++ shim before you call them.

        e.g.)

        Rust
        -----
        #[no_mangle]
        pub extern an_int() -> c_int {
            123 as c_int
        }

        C++ Shim
        ---------
        extern "C" {

        // Rust function prototypes
        int an_int();

        jint
        Java_com_namespace_appname_MainActivity_callRustFunction(JNIEnv* env, jobject obj)
        {
            int some_int = an_int();
            return some_int;
        }

        } // End extern

6. Android Studio will automatically include all `.so` files in `jniLibs` in the APK, but if you want to use a custom shim to wrap the Rust code in order to get the JNI name mangling correct, you will need a custom `Android.mk` with your shim source in your `jni` directory:

        LOCAL_PATH := $(call my-dir)

        include $(CLEAR_VARS)
        LOCAL_MODULE := my-awesome-lib
        LOCAL_SRC_FILES := ../jniLibs/$(TARGET_ARCH_ABI)/libmy-awesome-lib.so
        include $(PREBUILT_SHARED_LIBRARY)

        include $(CLEAR_VARS)
        LOCAL_MODULE    := app-name
        LOCAL_SRC_FILES := shim.cpp
        LOCAL_SHARED_LIBRARIES := my-awesome-lib
        include $(BUILD_SHARED_LIBRARY)
7. By default, Android Studio generates a makefile in order to link against libs and headers included with the NDK and will generate one for all of your code in the `jni` directory.  Unfortunately, this will not allow us to use ours, which includes our shared lib.  A custom build.gradle will need created in order to manually build using `Android.mk` and copy the generated libs into the `jniLibs` directory.

        import org.apache.tools.ant.taskdefs.condition.Os

        def getPlatformNdkBuildCommand() {
            def rootDir = project.rootDir
            def localProperties = new File(rootDir, "local.properties")
            if (localProperties.exists()) {
                Properties properties = new Properties()
                localProperties.withInputStream {
                    instr -> properties.load(instr)
                }
                def ndkDir = properties.getProperty('ndk.dir')
                if (ndkDir == null) {
                    throw new GradleException("The ndk.dir property in local.propeties is not set")
                }
                def ndkBuild = Os.isFamily(Os.FAMILY_WINDOWS) ? "$ndkDir/ndk-build.cmd" : "$ndkDir/ndk-build"
                return ndkBuild
            } else {
                throw new GradleException("The local.properties file does not exist")
            }
        }

        apply plugin: 'com.android.application'

        android {
            compileSdkVersion 21
            buildToolsVersion "21.1.2"

            sourceSets.main {
                jniLibs.srcDir 'src/main/jniLibs'
                jni.srcDirs = [] //disable automatic ndk-build call
            }

            defaultConfig {
                applicationId "com.namespace.appname"
                minSdkVersion 15
                targetSdkVersion 21
                versionCode 1
                versionName "1.0"
            }

            buildTypes {
                release {
                    minifyEnabled false
                    proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
                }
            }

            // Compile cpp code from Android.mk
            task buildCppShim(type: Exec) {
                def ndkBuild = getPlatformNdkBuildCommand()
                commandLine "$ndkBuild", '-j8', '-C', file('src/main/jni').absolutePath
            }

            // Copy shared libs into jniLibs folder (Hacky workaround)
            task copySharedLibs(type: Copy) {
                from 'src/main/libs'
                into 'src/main/jniLibs'
            }

            tasks.withType(JavaCompile) {
                compileTask -> compileTask.dependsOn buildCppShim
            }

            copySharedLibs.dependsOn buildCppShim
            copySharedLibs.execute()
        }

        dependencies {
            compile fileTree(dir: 'lib', include: ['*.jar'])
            compile 'com.android.support:appcompat-v7:21.0.3'
        }
8. Add the location of the NDK to your `local.properties` file

        ndk.dir=/abs/path/to/extracted/android-ndk-r10d
9. Load the libraries in your Activity

        static {
            System.loadLibrary("my-awesome-lib");
            System.loadLibrary("cpp-shim");
        }
