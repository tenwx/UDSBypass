# UDSBypass

Magisk UDS Detection Bypass Script by Ingan121  

# How to bypass?
[Download this script](https://github.com/Ingan121/UDSBypass/raw/master/bootudsbypass.sh) and put it to /data/adb/service.d  
(Long-tap the link and tap 'Save link' or such thing. Check the filename after downloading, since some browsers add a .txt extension when downloading.)
After doing this, run
> su -c chmod 000 /proc/net/unix

on terminal or simply reboot your device.

# Overview of UDS Detection
UDS detection (Unix Domain Socket detection) is a very stupid and dumb way to detect [Magisk](https://github.com/topjohnwu/Magisk). Magisk obfuscates its socket names to random 32-digit strings. UDS detection sees this random string as a trace of Magisk. To find this trace, it reads '/proc/net/unix', and finds a 32-digit (without \@) socket name starting with \@, then if it exists, it says 'rooted'.

It is first introduced with [the proposed commit request on the Rootbeer repository](https://github.com/scottyab/rootbeer/pull/88). The author closed the request herself and renamed her repository containing a modified Rootbeer with UDS detection to [RootbeerFresh](https://github.com/kimchangyoun/RootbeerFresh). After then, some security solutions like [DxShield](http://www.nshc.net/home/mobile-security/fxshield/) and [Promon](https://promon.co/) was updated to include this Magisk detection method. (As of 6/2, it seems no other apps and security solutions except the prementioned two are either using this method or able to detect Magisk 19+.)

As you can see, it is really dumb. It only checks the string's length. (@ check is useless, just take a look at the contents of /proc/net/unix) There are some known issues with this detection like detecting root ststus falsely, as you can see [here](https://github.com/KimChangYoun/rootbeerFresh/issues/4).

However, there is another problem. This detection method will be unavailable soon. Magisk developer ([\@topjohnwu](https://github.com/topjohnwu)) [said on Twitter](https://twitter.com/topjohnwu/status/1121993377933877248) that he came up with a idea to fundamentally circumvent this detection. Also, importantly, Google made /proc/net (including ./unix) unavailable for apps running on Android Q.

It is even not hard to bypass this detection. Firstly, you can simply revoke 'r' permission of '/proc/net/unix' for apps. Secondly, you can use this script if an app even refuses to run when you revoked the permission (of course, these kinds of apps must change this behavior, in order to make them run on Android Q).

So, I think it'll be better to use other Magisk detection method, not the UDS detection.

# Time attack-based bypass script (Deprecated)

[Download Script](https://github.com/Ingan121/UDSBypass/raw/master/udsbypass)  
(Long-tap the link and tap 'Save link' or such thing. Check the filename after downloading, since some browsers add a .txt extension when downloading.)

You must use this method if you're going to bypass DxShield. See below for more information.

## How to use?
In most cases, you can simply use the method mentioned above. However, some poorly-coded "security" solutions won't pass if /proc/net/unix is inaccessible. If so, you'll have to run the script below with the name of the activity that you want to use, see below for more.

The required string ([package name]/[activity name]) can be obtained easily if you're using Nova Launcher. Enable 'Show component...' in Settings > Lab (Google it if you don't know) > Debug, then open the edit dialog of the app. You can copy the string just by tapping it.

Since this script kills the Magisk daemon (magiskd), some other root apps might lose root before restarting magiskd, and all apps cannot start a new root shell while magiskd is not running. So, DO NOT exit the terminal before relaunching magiskd, otherwise you'll have to reboot to get the root back.

## Command usage
Usage: sh udsbypass [package name]/[activity name] [delay]  
or: sh udsbypass [options]

Delay: sets the delay before restarting magiskd  
leave empty or use -1 to restart it manually

Options:  
--help: shows this message  
--version: shows the version of this script  


## Per-app guides

### [DxShield](http://www.nshc.net/home/mobile-security/fxshield/)
If you see a dialog saying
> \> ROOTED [/ RSH (if root shell is running on any app)]  
> \> MAGISK
>
> Security engine: v5.0.2/5.1.0 (lower versions can be bypassed since Magisk 19.0+)

, then the app is definitely using FxShield (banking apps etc) or GxShield (games).  
Also, if files named dxshield.map and dxshield.sys exist in /data/data/[package name]/files, then it's using *xShield, too.

For these, you can just run
> su  
> sh /sdcard/path/to/udsbypass [package name]/[activity name]

in terminal to bypass it. If it is still not working, run
> su -c chmod 600 /sepolicy /sys/fs/selinux/policy  
on terminal. (In most cases, you can simply ignore the errors on terminal.)

### [Promon](https://promon.co)
Promon can be bypassed only using the method at the top. If it is still crashing immediately, first check if MagiskHide is enabled and check if every terminal app is running a 'su' shell then close the shell (or the terminal itself) if exists.

~~If the app crashes immediately when not using Hide and crashes after a while when using Hide, then probably it is using Promon.  
Since Promon has a wierd detection logic that checks if a root shell is running on terminal apps, you have to run a SSH server using some apps like [SSHDroid](https://play.google.com/store/apps/details?id=berserker.android.apps.sshdroid) and access to it by using any SSH client (e.g. [Termius](https://play.google.com/store/apps/details?id=com.server.auditor.ssh.client), [ConnectBot](https://play.google.com/store/apps/details?id=org.connectbot), [JuiceSSH](https://play.google.com/store/apps/details?id=com.sonelli.juicessh)). Then, you can run this script there.~~ 

Command:  
> su (if it's not a root shell. if # shows in left side of the command, then it's a root shell.)  
> sh /sdcard/path/to/udsbypass [package name]/[activity name] 15

Unlike DxShield, it checks root only when the app starts, so you can restart magiskd after the check is done.