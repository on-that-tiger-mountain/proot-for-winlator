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

place proot-64.tzst into assets directory。

And then ,modify GuestProgramLauncherComponent.java:

change1:
```
    private int execGuestProgram() {
        Context context = environment.getContext();
        ImageFs imageFs = environment.getImageFs();
        File prootDir = new File(context.getFilesDir(), "proot-android");
        File rootDir = imageFs.getRootDir();
```
change2:
```
        String command = prootDir+"/proot";

change3:

        envVars.put("PROOT_TMP_DIR", tmpDir);
        envVars.put("LD_LIBRARY_PATH", prootDir+"/");
        envVars.put("PROOT_LOADER", prootDir+"/loader");
        envVars.put("PROOT_LOADER_32", prootDir+"/loader-m32");

XServerDisplayActivity.java:
```
change1:

        final File rootDir = imageFs.getRootDir();
        final File prootDir = new File(activity.getFilesDir(), "proot-android");

change2:

            if (success) {
                imageFs.createImgVersionFile(LATEST_VERSION);
                resetContainerImgVersions(activity);

TarCompressorUtils.extract(TarCompressorUtils.Type.ZSTD, activity, "proot-64.tzst", prootDir);
            }
            else AppUtils.showToast(activity, R.string.unable_to_install_system_files);

Tips:

termux's proot need libtalloc.so.2 to launch. 

The default path is /data/data/com.termux/files/usr/lib.

Use LD_LIBRARY_PATH= to change it.

Okay!Replace the proot in the winlator app with termux's proot successfully.

The bug "can not launch container." is fixed.

Now,some phone like xiaomi 14 ultra can use winlator.

other details:

https://github.com/on-that-tiger-mountain/pure-64bit-winlator-app

https://github.com/on-that-tiger-mountain/pure-64bit-winlator

proot
=====
[![Travis build status](https://travis-ci.org/termux/proot.svg?branch=master)](https://travis-ci.org/termux/proot)

This is a copy of [the PRoot project](https://github.com/proot-me/PRoot/) with patches applied to work better under [Termux](https://termux.com).
