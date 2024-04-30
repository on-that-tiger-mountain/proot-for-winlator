How to build 64 bit termux's proot for winlator.
Use termux-change-repo to select a better net at first.
pkg update
pkg install git -y
git clone https://github.com/termux/proot.git
pkg install build-essential libtalloc binutils -y
cd ~/proot/src
make proot
Okay!proot,loader,loader-m32 is need.
rename proot to libproot.so
rename loader to libproot-loader.so
rename loader-m32 to libproot-loader32.so 
take libtalloc.so from termux's libtalloc deb package directly.
libproot.so,libproot-loader.so,libproot-loader32.so,libtalloc.so is really.
And then ,modify the winlator.
Tips:
termux's proot need libtalloc.so to launch. 
The default path is /data/data/com.termux/files/usr/lib.
Use LD_LIBRARY_PATH= to change it.
Okay!Replace the proot in the winlator app with termux's proot successfully.

proot
=====
[![Travis build status](https://travis-ci.org/termux/proot.svg?branch=master)](https://travis-ci.org/termux/proot)

This is a copy of [the PRoot project](https://github.com/proot-me/PRoot/) with patches applied to work better under [Termux](https://termux.com).
