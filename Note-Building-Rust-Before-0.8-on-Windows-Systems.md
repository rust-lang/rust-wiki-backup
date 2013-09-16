**Please note that this is a in-progress and temporary guide only intended for versions of Rust before 0.8 on Windows systems.**

This temporary guide will cover building Rust < 0.8 on a Windows system through MSYS & MinGW using gcc 4.8 .

## MSYS & MinGW Installation

1. Download the MinGW installer:
 * Latest: [http://sourceforge.net/projects/mingw/files/latest/download?source=files](http://sourceforge.net/projects/mingw/files/latest/download?source=files)
2. Run the `mingw-get-setup` installer.
 * For simplicity use the default path of `C:\MinGW` , pretty please.
 * Run with Administrator privileges if possible.
3. Make sure MinGW is on your system path.
 * `;C:\MinGW\bin` for example should be appended to the PATH environment variable, similar to these instructions: [http://stackoverflow.com/a/6318188/458550](http://stackoverflow.com/a/6318188/458550) .
 * Be careful not to disturb the existing entries.
4. It is pretty handy to create a shortcut on your desktop to `C:\MinGW\msys\1.0\msys.bat`

## MSYS & MinGW Packages

We have a few packages to add.

1. Run the mingw-get GUI from your desktop by double clicking the "MinGW Installer" shortcut.
2. Right click the following packages (bin) and select "Mark for Installation" using the left side for navigation
	* Basic Setup
 		* `mingw-developer-toolkit`
 		* `mingw32-base`
 		* `mingw32-gcc-g++`
	* All Packages
		* `msys-wget`
		* MinGW Libraries
			* MinGW Standard Libraries
				* `mingw32-libpthread-old`
				* `mingw32-libpthreadgc`
3. From the GUI menu navigate to Installation > Apply Changes
	* This will take some time to complete.

## Git Installation

If you already have Git installed and on the path please skip this section. You should be able to use the git command from your msys shell.

1. Install git for windows from [http://git-scm.com/download/win](http://git-scm.com/download/win)
2. Uncheck Associate .sh files to be run with Bash from within the components screen.
3. Select the option to Run Git from the Windows Command Prompt.
4. It is recommended to choose the option to Checkout as-is, commit Unix-style line endings.
5. Open a new msys console and type `git --version` to verify installation.

* Note: if your shell (explorer) does not return  after installation, start the task manager and navigate to File > Run New Task... then enter `explorer`.

## Python27 Installation

If you already have python installed and on the path, you may skip this section.

1. Get the Python27 installer from http://www.python.org/getit/
 * I am using "Python 2.7.5 Windows X86-64 Installer"
2. Accept the defaults, please.
3. Add python to your system path: [http://stackoverflow.com/a/6318188/458550](http://stackoverflow.com/a/6318188/458550)

## Clone Rust

1. Navigate to a place on your file system where you feel comfortable downloading the Rust repository.
	* `C:\projects` is a nice cozy spot!
2. execute `git clone git://github.com/mozilla/rust.git` to clone the Rust repository into a new `rust` directory.

While the clone is taking place you can continue on to patching.

## Patches

There are some issues with our environment that we need to patch up. This gets a little shady but it should get you down the path of building Rust on Windows. I won't lie, this is some greasy stuff but it will get you further down the compilation path. **Thanks to klutzy for all of these.**

### Patch for `WSAPOLLFD`

If you receive errors regarding WSAPOLLFD (haha, you will!) you must patch `/mingw/include/winsock2.h` by inserting the following at line 915:

    typedef struct pollfd {
      SOCKET fd;
      short  events;
      short  revents;
    } WSAPOLLFD, *PWSAPOLLFD, *LPWSAPOLLFD;

From: [http://sourceforge.net/p/mingw/bugs/1980/](http://sourceforge.net/p/mingw/bugs/1980/)

### Patch for `FE_ALL_EXCEPT`

For errors regarding FE_ALL_EXCEPT you must patch `\MinGW\lib\gcc\mingw32\4.8.1\include\c++\mingw32\bits\c++config.h` by inserting the following at line 597:

    #define _GLIBCXX_HAVE_FENV_H 1

### Patch for `FILE_FLAG_FIRST_PIPE_INSTANCE`

If you have errors regarding the definition of `FILE_FLAG_FIRST_PIPE_INSTANCE` you must patch `\MinGW\include\winbase.h` by changing line 1986 from `#if (NTDDK_VERSION >= NTDDI_WIN2KSP2)` to the following:

	#if (_WIN32_WINNT >= 0x0500)

## Configure & Build

1. Navigate to the directory where you cloned rust from within the msys shell. For me that is `cd /C/Projects/rust`.
2. Run the command `./configure`
	* This will take a while!
3. When that completes run the command `make`
	* This will take much longer!
4. Get your sad face ready because this build will fail. Eventually you should get an error such as this:

		---------------------------
		rustc.exe - Application Error
		---------------------------
		The application was unable to start correctly (0xc0000142). Click OK to close the application.
		---------------------------
		OK  
		---------------------------

5. Download `libstdc++-4.6.2-1-mingw32-dll-6.tar.lzma` from [http://sourceforge.net/projects/mingw/files/MinGW/Base/gcc/Version4/gcc-4.6.2-1/libstdc%2B%2B-4.6.2-1-mingw32-dll-6.tar.lzma/download](http://sourceforge.net/projects/mingw/files/MinGW/Base/gcc/Version4/gcc-4.6.2-1/libstdc%2B%2B-4.6.2-1-mingw32-dll-6.tar.lzma/download) and extract the contained libstdc++-6.dll into the `\rust\i686-pc-mingw32\stage0\bin` folder.
	* Try running `rustc.exe` to confirm the fix.
7. Run the `make` command again.
