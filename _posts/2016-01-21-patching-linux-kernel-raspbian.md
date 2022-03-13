---
layout: post
location: Clermont-Fd, France
tldr: false
title: "Patching the Linux kernel (Raspbian &amp; CVE-2016-0728)"
tweet_id: 690069917241057280
updates:
  - date: 2022-03-11
    content: I proofread this article and removed dead links.
---

[CVE-2016-0728][] has been disclosed earlier this week and it is a [serious
security issue](https://threatpost.com/serious-linux-kernel-vulnerability-patched/115923/).
The vulnerability affects most of the Linux kernel versions (3.8 and above).
Although the exploit seems tricky to successfully use, it is still a flaw that
has to be patched ASAP.

I use a few Raspberry Pis for a while now and they all run
[Raspbian](https://www.raspbian.org/), a Debian-based distribution for Raspberry
Pi. I tried to `apt-get update && apt-get (dist-)upgrade` one of them but
nothing new was available, _i.e._ no patched version available.

At the time of writing, there was only [this single unanswered
issue](https://github.com/raspberrypi/linux/issues/1264) on the Raspberry Pi
kernel GitHub repository. I looked into the [Kernel source code][] and the code
seemed vulnerable to me (according to the patch and what I understood from the
[report][]).

I wanted to run a patched kernel version therefore I decided to [compile the
Linux kernel][]. You will find the different steps I followed to build, install
and run a patched Linux kernel below:

1. First, the `bc` package is needed (`apt-get install bc`), then the kernel
   sources have to be cloned:

   ```
   $ git clone --depth=1 https://github.com/raspberrypi/linux
   ```

   Longest git checkout ever!

2. In order to compile a new kernel version, we have to slighty update its name.
   I edited the `EXTRAVERSION` variable in the `Makefile`:

   ```
   $ head Makefile -n 4

   VERSION = 4
   PATCHLEVEL = 1
   SUBLEVEL = 15
   EXTRAVERSION = +will
   ```

3. Now let's fetch the [patch][] for this vulnerability, and apply it:

   ```
   $ curl https://git.kernel.org/cgit/linux/kernel/git/torvalds/linux.git/patch/?id=23567fd052a9abb6d67fe8e7a9ccdd9800a540f2 | patch -p1
   ```

4. So far so good. Before compiling the kernel, we have to instruct which kernel
   we wish to build, then we can build the related configuration:

   ```
   $ export KERNEL=kernel7
   $ make bcm2709_defconfig
   ```

5. Time to compile the kernel and its modules:

   ```
   $ make -j4 zImage modules dtbs
   ```

6. I started to write this blog post while it was still compiling... At some
   point, compilation successfully ended. Let's install this brand new kernel:

   ```
   $ sudo make modules_install
   $ sudo cp arch/arm/boot/dts/*.dtb /boot/
   $ sudo cp arch/arm/boot/dts/overlays/*.dtb* /boot/overlays/
   $ sudo cp arch/arm/boot/dts/overlays/README /boot/overlays/
   $ sudo scripts/mkknlimg arch/arm/boot/zImage /boot/$KERNEL.img
   ```

7. And now, time to try it for real (fingers crossed):

   ```
   $ uname -a
   Linux raspberrypi 4.1.15-v7+ #831 SMP Tue Jan 19 18:39:46 GMT 2016 armv7l GNU/Linux
   ```

   ```
   $ sudo reboot
   ```

   ```
   $ uname -a
   Linux raspberrypi 4.1.15+will-v7+ #1 SMP Thu Jan 21 02:09:58 CET 2016 armv7l GNU/Linux
   ```

Achievement unlocked \o/

[compile the linux kernel]: https://www.raspberrypi.com/documentation/computers/linux_kernel.html
[cve-2016-0728]: https://nvd.nist.gov/vuln/detail/CVE-2016-0728
[kernel source code]: https://github.com/raspberrypi/linux/blob/d51c7d840b002a6b26089d8b45679d9331880060/security/keys/process_keys.c#L796-L799
[patch]: https://git.kernel.org/cgit/linux/kernel/git/torvalds/linux.git/patch/?id=23567fd052a9abb6d67fe8e7a9ccdd9800a540f2
[report]: http://perception-point.io/2016/01/14/analysis-and-exploitation-of-a-linux-kernel-vulnerability-cve-2016-0728/
