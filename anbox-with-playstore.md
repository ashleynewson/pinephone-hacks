# Anbox with Play Store

This is a bit of a multi-layered problem.

## Getting Anbox to run

I am using Manjaro (Phosh). Sometimes, the pre-packaged Anbox from Manjaro ARM just doesn't work by itself.

The best way to diagnose this error is to look at the errors spewed out by `anbox session-manager`.

Common problems:

### Installed version of lxc is incompatible

If you get something complaining about containers or the anbox-container-manager (which should be running), it may well be the case that the lxc package is too new. Anbox has been re-packaged a number of times by the Manjaro team in order to keep it vaguely up-to-date with lxc, but it's not always very prompt.

One way to get around this _might_ be to build Anbox yourself. It might not work, though. You can also try downgrading lxc for now (but on your head be it). `1:4.0.6-1` seemed to work at some point.

## Getting an Anbox image with Play Store on it

You will need to build your own Anbox image for this. For legal reasons, I can't give you one, because Open GApps (which contains Play Store) has a restrictive licence.

However, I _can_ give you the PKGBUILD I have that builds such an image. Unfortunately, you might need to tweak a few version numbers and checksums (or set them to `'SKIP'`) depending on what's available to download at the time.

Check out the `anbox-image-custom-aarch64` directory.

You may need to uninstall the stock `anbox-image-aarch64` package first.

## Getting Anbox to run with Play Store without crashing

If you've tried to run Anbox with your shiny new opengapps image, you might observe it work for a few seconds and then suddenly crashing soon after. Or it might just crash before anything shows up. Either way, it doesn't work.

Again, run `anbox session-manager` and you probably have this problem:

```
Stack trace (most recent call last) in thread 7761:
#7    Object "/usr/lib/ld-2.32.so", at 0xffffffffffffffff, in 
#6    Object "/usr/lib/libc-2.32.so", at 0xffff910fa95b, in thread_start
#5    Object "/usr/lib/libpthread-2.32.so", at 0xffff914e4f43, in start_thread
#4    Object "/usr/bin/anbox", at 0xaaaaab54ea13, in emugl::Thread::thread_main(void*)
#3    Object "/usr/bin/anbox", at 0xaaaaab43db43, in RenderThread::main()
#2    Object "/usr/bin/anbox", at 0xaaaaab54630b, in renderControl_decoder_context_t::decode(void*, unsigned long, IOStream*)
#1    Object "/usr/bin/anbox", at 0xaaaaab4be7f7, in 
#0    Object "/usr/lib/libc-2.32.so", at 0xffff910a7e8c, in strlen
Segmentation fault (Address not mapped to object [0x1f00])
Segmentation fault (core dumped)
```

If you Google this, you'll probably instead find some similar-looking errors which aren't actually helpful at all. Compare your error with the above carefully. Does it crash in `strlen`?

Got the same error I did? Good. I'll tell you what's gone wrong. I had to use gdb to figure it out.

Something about the Open GApps we've installed in our image wants to use GLESv1. Diving into the Anbox source code, we can see that when using host GLES drivers that no GLESv1 library is selected in the default library list. This means that all of Anbox's GLESv1 stuff is just dummy functions (or rather all just one dummy function). When it tries to get the list of GLES extensions available, it erroneously asks this dummy function. The caller thinks it's getting a string, but the dummy function doesn't actually return anything, so some falsely initialised value gets used. Ultimately this gets passed to `strlen`, which then reads invalid memory.

So, let's modify the Anbox source code and build it ourselves. See the files in the `anbox` directory.

But our journey is still not over because our Pinephone host doesn't actually have any `libGLES_CM.so` (the GLESv1 library).

libglvnd provides our EGL and GLESv2 libraries, and it _could_ be supplying a GLESv1 library, but the Arch/Manjaro package has been built with GLESv1 support explicitly disabled. I presume this is for maximum compatibility across all systems, but our Pinephone can handle GLESv1 just fine... I think.

Again, it seems we need to build something ourselves. See the `libglvnd` directory.

### It should work now

Well, that was a faff. But now, it seems that Play Store runs on our Pinephone. And it doesn't immediately crash.

Anbox is still quite buggy though, partly lending to the fact it's running on Wayland and Phosh.

You might also notice that you can't find some apps in the Play Store. This is typically because the Play Store believes the app to be incompatible with the device.
