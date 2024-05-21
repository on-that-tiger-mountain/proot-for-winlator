How to build 64 bit termux's proot for winlator.

Use termux-change-repo to select a better net at first.

pkg update

pkg install git -y

git clone https://github.com/termux/proot.git

pkg install build-essential libtalloc binutils -y

cd ~/proot/src

make proot

Okay!proot,loader,loader-m32 is need.

delete libproot.so libproot-loader.so libproot-loader32.so in lib/arm64-v8a directory.

take libtalloc.so.2 from termux's libtalloc deb package directly.

use "tar -I zstd -cvf" to compress "proot,loader,loader-m32,libtalloc.so.2" into proot-64.tzst

And then ,modify GuestProgramLauncherComponent.java

XServerDisplayActivity.java
Tips:

termux's proot need libtalloc.so.2 to launch. 

The default path is /data/data/com.termux/files/usr/lib.

Use LD_LIBRARY_PATH= to change it.

Okay!Replace the proot in the winlator app with termux's proot successfully.

The bug "can not launch container." is fixed.

Now,some phone like xiaomi 14 ultra can use winlator.

other details:

https://github.com/tiger-mountain/pure-64bit-winlator.

proot
=====
[![Travis build status](https://travis-ci.org/termux/proot.svg?branch=master)](https://travis-ci.org/termux/proot)

This is a copy of [the PRoot project](https://github.com/proot-me/PRoot/) with patches applied to work better under [Termux](https://termux.com).
