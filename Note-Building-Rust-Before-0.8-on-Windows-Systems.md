**Please note that this is an in-progress and temporary guide only intended for versions of Rust before 0.8 on Windows systems.**

This temporary guide will cover building Rust < 0.8 on a Windows system through MSYS & MinGW using gcc 4.8 .

NOTE
- Rust requires gcc < 4.6 to build, but recently mingw has upgraded whole system and it is currently not easy to match package versions. This guide directly uses gcc 4.8.
- Latest mingw w32api package (4.0.0-1) contains some bugs. See "Patches" section below for instruction.
- There are some abi change between gcc 4.6 and 4.8, and some tests are known to fail due to this. (https://github.com/mozilla/rust/issues/9205)

## MSYS & MinGW Installation

1. Download the MinGW installer:
 * Latest: [http://sourceforge.net/projects/mingw/files/latest/download?source=files](http://sourceforge.net/projects/mingw/files/latest/download?source=files)
2. Run the `mingw-get-setup` installer.
 * For simplicity use the default path of `C:\MinGW` , pretty please.
 * I like to install for all users.
 * Run with Administrator privileges if possible.
3. Make sure MinGW is on your system path.
 * `;C:\MinGW\bin` for example should be appended to the PATH environment variable, similar to these instructions: [http://stackoverflow.com/a/6318188/458550](http://stackoverflow.com/a/6318188/458550) .
 * Be careful not to disturb the existing entries.
4. It is pretty handy to create a shortcut on your desktop to `C:\MinGW\msys\1.0\msys.bat`

## MSYS & MinGW Packages

We have a few packages to add.

1. Run the mingw-get GUI from your desktop by double clicking the "MinGW Installer" shortcut.
2. Right click the following list of packages (bin) within the right pane and select "Mark for Installation". Use the left navigation tree to filter categories.
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
	* Currently some packages should be downgraded (https://github.com/mozilla/rust/issues/5878)

## Git Installation

If you already have Git installed and available on the path you may skip this section. You should be able to use the git command from your msys shell.

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
	* This will take a really long time!

## Rust is Ready

    User@Machine /c/projects/rust/i686-pc-mingw32/stage2/bin
    $ rustc.exe hello.rs
    
    User@Machine /c/projects/rust/i686-pc-mingw32/stage2/bin
    $ hello
    Hello, world.

### Moving Rust

If you want to move rust to a more permanent location on your system (for example C:\Program Files) you must preserve a little bit of structure. If you fail to do this correctly you may receive errors such as ``can't find crate for `std` ``. The contents of the `stage2/bin` folder must be contained within a folder named `bin`. The simplest way to do this is the following:

* Copy the contents of `stage2/bin` to `C:\Program Files\rust\bin`
  * Be sure to include the `rustc` folder too
* Add `;C:\Program Files\rust\bin` to your system path