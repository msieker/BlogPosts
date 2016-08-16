Title: Gentoo? On my Raspberry Pi?
Category: RaspberryPi
Date: 2013-1-18 18:30
Status: draft

Being somewhat of a control freak when it comes to Linux distros (I'm not going
to print from this box, why do I need libcups?), I tend to favor Gentoo when it's
a box for something not quite everyday use (servers, firewalls, other dedicated
systems).

As such, I'm trying to get Gentoo running on my Raspberry Pi board, and I thought
I'd document some of my process in case it might help others.

One of my goals is to have the Pi do as little work as possible for the emerges,
so I want another box to do cross-compiling and making binary packages for as
many things as possible.

First is setting up the cross compile environment
    
    :::bashsession
    $ emerge crossdev
    $ crossdev -v -t armv6j-hardfloat-linux-gnueabi
    
And then go off and do something for awhile while that builds.

Once that's done, grab a stage 3 armv6j hardfp tarball from your friendly neighborhood
Gentoo mirror, and put that into the crossdev root (Please note that the URL below won't
work beyond when I'm writing this, since the tarball has the build date on it, even if
it's in current):

    :::bashsession
    $ wget http://mirror.mcs.anl.gov/pub/gentoo/releases/arm/autobuilds/current-stage3-armv6j_hardfp/stage3-armv6j_hardfp-20130117.tar.bz2
    $ tar xvjpf stage3-armv6j_hardfp-20130117.tar.bz2 -C /usr/armv6j-hardfloat-linux-gnueabi
    
Set up the profile:

    :::bashsession
    $ ln -s /usr/portage/profiles/default/linux/arm/10.0 /usr/armv6j-hardfloat-linux-gnueabi/etc/make.profile
    
Tweak /usr/etc/armv6j-hardfloat-linux-gnueabi/make.conf a bit:
 
    :::conf
    # These settings were set by the catalyst build script that automatically
    # built this stage.
    # Please consult /usr/share/portage/config/make.conf.example for a more
    # detailed example.
    CHOST="armv6j-hardfloat-linux-gnueabi"
    CBUILD=x86_64-pc-linux-gnu
    ARCH=arm


    ROOT=/usr/${CHOST}/
    CFLAGS="-O2 -pipe -march=armv6j -mfpu=vfp -mfloat-abi=hard -I${ROOT}usr/include/ -I${ROOT}include/"
    CXXFLAGS="${CFLAGS}"
    # WARNING: Changing your CHOST is not something that should be done lightly.
    # Please consult http://www.gentoo.org/doc/en/change-chost.xml before changing.
    E_MACHINE=EM_ARM
    ACCEPT_KEYWORDS="arm ~arm"
    MAKEOPTS="-j5"
    LDFLAGS="-L${ROOT}lib -L${ROOT}usr/lib"
    PKGDIR=${ROOT}packages/
    PORTAGE_TMPDIR=${ROOT}tmp/
    USE="-ipv6"

    LIBDIR_arm="lib"
    LIBDIR_amd64=lib64
    PORTDIR="/usr/portage"
    DISTDIR="/usr/portage/distfiles"
    PORTDIR_OVERLAY="/usr/local/portage"



(Adjust MAKEOPTS for your own environment. I'm building on a quad core box)

And, as a sanity check, emerge something simple and useful, like NTP. Note the -b option on
emerge, since I'm wanting to make a binary package server.

    :::bashsession
    $ emerge-armv6j-hardfloat-linux-gnueabi -b ntp