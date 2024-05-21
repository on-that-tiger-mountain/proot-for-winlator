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

And then ,modify GuestProgramLauncherComponent.java:

    private int execGuestProgram() {
        Context context = environment.getContext();
        ImageFs imageFs = environment.getImageFs();
        File prootDir = new File(context.getFilesDir(), "proot-android");
        File rootDir = imageFs.getRootDir();
        File tmpDir = environment.getTmpDir();
        String nativeLibraryDir = context.getApplicationInfo().nativeLibraryDir;

        EnvVars envVars = new EnvVars();
        addBox86EnvVars(envVars);
        addBox64EnvVars(envVars);
        envVars.put("HOME", ImageFs.HOME_PATH);
        envVars.put("USER", ImageFs.USER);
        envVars.put("TMPDIR", "/tmp");
        envVars.put("LANG", "zh_CN.UTF-8");
        envVars.put("DISPLAY", ":0");
        envVars.put("PATH", imageFs.getWinePath()+"/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin");
        envVars.put("LD_LIBRARY_PATH", "/usr/lib/aarch64-linux-gnu:/usr/lib/arm-linux-gnueabihf");
        envVars.put("LD_PRELOAD", "libandroid-sysvshm.so");
        envVars.put("ANDROID_SYSVSHM_SERVER", UnixSocketConfig.SYSVSHM_SERVER_PATH);
        if (this.envVars != null) envVars.putAll(this.envVars);

        boolean bindSHM = envVars.get("WINEESYNC").equals("1");

        String command = prootDir+"/proot";
        command += " --kill-on-exit";
        command += " --rootfs="+rootDir;
        command += " --cwd="+ImageFs.HOME_PATH;
        command += " --bind=/dev";

        if (bindSHM) {
            File shmDir = new File(rootDir, "/tmp/shm");
            shmDir.mkdirs();
            command += " --bind="+shmDir.getAbsolutePath()+":/dev/shm";
        }

        command += " --bind=/proc";
        command += " --bind=/sys";

        if (bindingPaths != null) {
            for (String path : bindingPaths) command += " --bind="+(new File(path)).getAbsolutePath();
        }

        command += " /usr/bin/env "+envVars.toEscapedString()+" box64 "+guestExecutable;

        envVars.clear();
        envVars.put("PROOT_TMP_DIR", tmpDir);
        envVars.put("LD_LIBRARY_PATH", prootDir+"/");
        envVars.put("PROOT_LOADER", prootDir+"/loader");
        envVars.put("PROOT_LOADER_32", prootDir+"/loader-m32");

        if (MainActivity.DEBUG_LEVEL >= 1) ProcessHelper.debugMode = true;
//        return ProcessHelper.exec(command, envVars.toStringArray(), rootDir, (status) -> {
        //改为根据条件重定向输入输出流
        return ExtraFeatures.PRootShell.exec(context, command, envVars.toStringArray(), rootDir, (status) -> {
            synchronized (lock) {
                pid = -1;
            }
            if (terminationCallback != null) terminationCallback.call(status);
        });
    }

XServerDisplayActivity.java:

TarCompressorUtils.extract(TarCompressorUtils.Type.ZSTD, this, "pulseaudio.tzst", new File(getFilesDir(), "pulseaudio"));

TarCompressorUtils.extract(TarCompressorUtils.Type.ZSTD, this, "proot-64.tzst", new File(getFilesDir(), "proot-android"));

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
