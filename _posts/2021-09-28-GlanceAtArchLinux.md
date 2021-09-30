---
layout: post
title:  "A glance at Arch Linux (and I'm not happy with it!)" 
author: Luca Ferrari
tags:
- linux
- arch
permalink: /:year/:month/:day/:title.html
---
I tried Arch Linux, and throw it away really soon!

# A glace at Arch Linux (and I'm not happy with it!)

So, I finally took some time to try [Arch Linux](https://archlinux.org/){:target="_blank"}, a distribution that has a big momentum and is often accomunated to *being for hackers*.
<br/>
I have to say: I installed, booted, and decided it is not what I want for me.
<br/>
And it's not because of the installer, no way. It is because **I don't get the feeling of being comfortable using it**.
<br/>
If you find yourself at home using Arch Linux, please continue. 
<br/>
If you believe I'm not enough "hacker" to use it, please go browse away.
<br/>
If you feel there is something less than awesome with Arch, please continue reading.

## Look ma, I'm an hacker!

I often listen to podcasts and video tutorial that tell people how Arch Linux is great due to its basic command-line installer: "it will teach you how Linux works!".
<br/>
True.
<br/>
But if you have being around the Unix world for quite a long time, there is no hype here. I mean, after all the advatanges of *Anaconda*, *Calamaris*, and name another few, was that the sysadmin did not have anymore to type a quite long and difficult set of commands to format the hard drive, mount it, copy the software on it, `chroot` into it, and so on. Quite frankly, I've done it enough times in the past, and today I don't care really much about the installation process.
<br/>
Clearly, this is great if you need or want to fully customize your installation process, but in the era of the virtualization, it sounds to me a lot faster to snapshot a virtual machine than to produce a fully customized installation script. *Even if I feel more comfortable with the latter!*
<br/>
FreeBSD, and in general, all other BSDs, have basic installars, and can drop you into a shell to do whatever you want to your system during installation. A common case is to build a software RAID mirror before the system actually installs.
<br/>
Therefore, in the long run, I don't feel much impressed by the command line installer, and surely this is not what can push me towards or backwards Arch Linux!

## The Installation Guide

Arch Linux has a very complete wiki, and the [installation guide](https://wiki.archlinux.org/title/Installation_guide#Set_the_keyboard_layout){:target="_blank"} is a glorious resource.
<br/>
However, it looks to me not complete as expected. For example, there is no room for setting a boot loader, rather it just tells you "install a bootloader", leaving the user to research which one to install and how to.
<br/>
After all, the installation guide just list a set of commands to almost-blindly repeat on your installation to get it bootable.
<br/>
I believe there is a great difference between *being short* and *being complete**, and this wiki page does not look to me as it is complete.
<br/>
Moreover, the list of software suggested to be installed is really a skeleton part, and while I appreciate it, probably the user should have a couple of suggestions for extra software to install on the barebone system before it reboots.


## The Documentation

While the Arch Wiki is great, after booting a freshly installed system a bad surprise happened to me:


<br/>
<br/>
<center>
<img src="/images/posts/linux/arch/arch_man.png" />
</center>
<br/>
<br/>

**The `man` is not installed?** 
<br/>
How am I supposed to learn how to interact on a system without `man` installed?
<br/>
I can search for within the wiki, but that's not either a Unix approach, nor an "hacky" way. The system must be self-documented to let the administrator to work with it even in the dramatic situation where no internet is available!
<br/>
I'm used to BSD systems here, and that's a very good point of thos systems.

## Pacman

The beloved package manager [pacman](https://archlinux.org/pacman/pacman.8.html){:target="_blank"}.
<br/>
I'm sure it is great, but why it is asking me to use `-S` at every time? And why this option is not explained in the base systsem lacking of the `man`?
<br/>
I mean, if the `-S` *sync* option is the preferred one, why is not enabled by default?


## AUR

The [Arch User Repository (AUR)](https://aur.archlinux.org/){:target="_blank"} is a set of user defined *scripts* that build and install third party software that has not been packaged yet (or will not be packaged).
<br/>
As an example, the following is the [AUR script `PKGBUILD` for `pgbackrest`](https://aur.archlinux.org/cgit/aur.git/tree/PKGBUILD?h=pgbackrest){:target="_blank"}:


<br/>
<br/>
```shell
# Maintainer: Shalygin Konstantin <k0ste@k0ste.ru>
# Contributor: Shalygin Konstantin <k0ste@k0ste.ru>

pkgname='pgbackrest'
pkgver='2.35'
pkgrel='2'
pkgdesc='Reliable PostgreSQL Backup & Restore'
arch=('x86_64')
url="https://github.com/${pkgname}/${pkgname}"
license=('MIT')
depends=('openssl' 'libxml2' 'icu' 'gcc-libs' 'xz' 'zstd' 'perl' 'postgresql-libs' 'libyaml' 'bzip2')
source=("$pkgname-$pkgver.tar.gz::${url}/archive/release/${pkgver}.tar.gz")
sha256sums=('ecaf8fc8ec68ac59143a7b4c0c53fded28d1e1396771a9456bb3fdca327e5718')
backup=("etc/${pkgname}/${pkgname}.conf")

build() {
  cd "${srcdir}/${pkgname}-release-${pkgver}/src"
  ./configure \
    --prefix="/usr"
  make
}

package() {
  cd "${srcdir}/${pkgname}-release-${pkgver}/src"
  make DESTDIR="${pkgdir}" install
  install -Dm644 ../LICENSE "$pkgdir/usr/share/licenses/$pkgname/LICENSE"
}

```
<br/>
<br/>


Despite the content, you can see how it does `make` and `install` after a classical run of `configure`. I don't want to over-simplify here, but again this reminds me something we already had: **BSD ports!**
<br/>
As an example, the following is the FreeBSD port for the very same `pgbackrest` package:


<br/>
<br/>
```shell
PORTNAME=	pgbackrest
DISTVERSION=	2.33
CATEGORIES=	databases

MAINTAINER=	schoutm@gmail.com
COMMENT=	Reliable PostgreSQL Backup & Restore

LICENSE=	MIT
LICENSE_FILE=	${WRKSRC}/../LICENSE

LIB_DEPENDS=	liblz4.so:archivers/liblz4

USES=		gmake gnome pkgconfig pgsql ssl
USE_GNOME=	libxml2
GNU_CONFIGURE=	yes

USE_GITHUB=	yes
GH_TAGNAME=	release/${DISTVERSION}

WRKSRC_SUBDIR=	src

ALL_TARGET=

CONFIGURE_ARGS=	--with-configdir="${LOCALBASE}/etc/pgbackrest"

OPTIONS_DEFINE=	ZSTD

ZSTD_LIB_DEPENDS=	libzstd.so:archivers/zstd
ZSTD_CONFIGURE_OFF=	ac_cv_lib_zstd_ZSTD_isError=no
ZSTD_CONFIGURE_ON=	ac_cv_lib_zstd_ZSTD_isError=yes

post-install:
	${STRIP_CMD} ${STAGEDIR}${PREFIX}/bin/pgbackrest
	${MKDIR} ${STAGEDIR}${PREFIX}/etc/pgbackrest

.include <bsd.port.mk>
```
<br/>
<br/>


The difference is that the above is `Makefile`, not a runnable script, but the idea is the very same: have a source to download and build on your system to get the software installed.


# Conclusions

I truly believ Arch Linux is a very good operating system, simply I don't see the hype in a system that gives me the same stuff I already have on my operating systems (e.g., BSDs) with less effort, same flexibility and, to some extent, a richer documentation.
<br/>
That's why I'm not really interested in trying it unless I have enough spare time.
