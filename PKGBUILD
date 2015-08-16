# Almost wholly taken from kernel26-ice by Giuseppe Calderaro
# Maintainer: Ng Oon-Ee <ngoonee.talk@gmail.com>
# Source Contributor: Giuseppe Calderaro <giuseppecalderaro@gmail.com>

pkgdesc="The Linux Kernel and modules with tuxonice support and rt-patchset"
depends=('coreutils' 'linux-firmware' 'kmod' 'mkinitcpio>=0.7')
optdepends=('crda: to set the correct wireless channels of your country')
pkgname=linux-rt-ice
backup=(etc/mkinitcpio.d/${pkgname}.preset)
_kernelname=${pkgname#linux}
_basekernel=3.6
pkgver=${_basekernel}.11
pkgrel=2
arch=('i686' 'x86_64')
url="https://rt.wiki.kernel.org"
license=('GPL2')
install=$pkgname.install
makedepends=('xmlto' 'docbook-xsl')
options=('!strip')

### User/Environment defined variables
enable_toi=${enable_toi:-1}
keep_source_code=${keep_source_code:-0}
gconfig=${gconfig:-0}
xconfig=${xconfig:-0}
menuconfig=${menuconfig:-0}
realtime_patch=${realtime_patch:-1}
local_patch_dir="${local_patch_dir:-}"
use_config="${use_config:-}"
use_config_gz=${use_config_gz:-0}
enable_reiser4=${enable_reiser4:-0}
### Compile time defined variables
###

### Files / Versions
file_toi="tuxonice-for-linux-3.6-9-2012-12-09.patch.bz2"
file_rt="patch-3.6.11-rt25.patch.xz"
###

source=("http://www.kernel.org/pub/linux/kernel/v3.x/linux-${_basekernel}.tar.xz"
        "http://www.kernel.org/pub/linux/kernel/v3.x/patch-${pkgver}.xz"
        #"${file_toi}::https://github.com/NigelCunningham/tuxonice-kernel/compare/vanilla-3.4...tuxonice-3.4.patch"
        "http://tuxonice.net/downloads/all/${file_toi}"
        "http://www.kernel.org/pub/linux/kernel/projects/rt/3.6/${file_rt}"
        # the main kernel config files
        'config' 'config.x86_64'
        # standard config files for mkinitcpio ramdisk
        "${pkgname}.preset"
        'change-default-console-loglevel.patch'
        'module-symbol-waiting-3.6.patch'
        'module-init-wait-3.6.patch')
md5sums=('1a1760420eac802c541a20ab51a093d1'
         'bd4bba74093405887d521309a74c19e9'
         'a3e5c2a4b1160df25e7320a63eacc901'
         '0be405891c7fb02b0026d4143f2207da'
         'b26fb7b866c9aecf94efe436fb4e9860'
         '6922b72b75913fd293b9be6a52f5a636'
         '8c3d4c34459ffea3f8e25497f222559d'
         '9d3c56a4b999c8bfbd4018089a62f662'
         '670931649c60fcb3ef2e0119ed532bd4'
         '8a71abc4224f575008f974a099b5cf6f')

build() {
  cd "${srcdir}/linux-${_basekernel}"

  # add upstream patch
  patch -p1 -i "${srcdir}/patch-${pkgver}"

  if [ -n "${local_patch_dir}" ] && [ -d "${local_patch_dir}" ] ; then
    echo "Applying patches from ${local_patch_dir} ..."
    for my_patch in "${local_patch_dir}"/* ; do
      echo -e "Applying custom patch:\t'${my_patch}'" || true
      patch -Np1 -i "${my_patch}" || { echo -e "Failed custom patch:\t'${my_patch}'" ; return 1 ; }
    done
  fi

  # Applying realtime patch
  if [ "$realtime_patch" = "1" ]; then
    echo "Applying real time patch"
    # Strip './Makefile' changes
    gzip -dc ${srcdir}/${file_rt} \
      | sed '/diff --git a\/Makefile b\/Makefile/,/*DOCUMENTATION*/d' \
      | patch -Np1 || { echo "Failed Realtime patch '${file_rt}'"; return 1 ; }
  fi

  # applying tuxonice patch
  if [ "${enable_toi}" = "1" ]; then
    echo "Applying ${file_toi%.bz2}"
      bzip2 -dck ${srcdir}/${file_toi} \
        | sed 's/printk(KERN_INFO "PM: Creating hibernation image:\\n/printk(KERN_INFO "PM: Creating hibernation image: \\n/' \
        | patch -Np1 -F4 || { echo "Failed TOI"; return 1 ; }
  fi


  # remove extraversion
  sed -i 's|^EXTRAVERSION = .*$|EXTRAVERSION =|g' Makefile

  # add latest fixes from stable queue, if needed
  # http://git.kernel.org/?p=linux/kernel/git/stable/stable-queue.git

  # set DEFAULT_CONSOLE_LOGLEVEL to 4 (same value as the 'quiet' kernel param)
  # remove this when a Kconfig knob is made available by upstream
  # (relevant patch sent upstream: https://lkml.org/lkml/2011/7/26/227)
  patch -Np1 -i "${srcdir}/change-default-console-loglevel.patch"

  # fix module initialisation
  # https://bugs.archlinux.org/task/32122
  patch -Np1 -i "${srcdir}/module-symbol-waiting-3.6.patch"
  patch -Np1 -i "${srcdir}/module-init-wait-3.6.patch"

  if [ "${CARCH}" = "x86_64" ]; then
    cat "${srcdir}/config.x86_64" > ./.config
  else
    cat "${srcdir}/config" > ./.config
  fi

  # use custom config instead
  if [ -n "${use_config}" ] ; then
    echo "Using config: '${use_config}'"
    cat "${use_config}" > ./.config
    make oldconfig
  fi

  # use existing config.gz
  if [ "$use_config_gz" = "1" ]; then
    zcat /proc/config.gz > ./.config
    make oldconfig
  fi

  if [ "${_kernelname}" != "" ]; then
    sed -i "s|CONFIG_LOCALVERSION=.*|CONFIG_LOCALVERSION=\"${_kernelname}\"|g" ./.config
    sed -i "s|CONFIG_LOCALVERSION_AUTO=.*|CONFIG_LOCALVERSION_AUTO=n|" ./.config
  fi

  # set extraversion to pkgrel
  sed -ri "s|^(EXTRAVERSION =).*|\1 -${pkgrel}|" Makefile

  # don't run depmod on 'make install'. We'll do this ourselves in packaging
  sed -i '2iexit 0' scripts/depmod.sh

  # get kernel version
  make prepare

  # load configuration
  # Configure the kernel. Replace the line below with one of your choice.
  if [ "$gconfig" = "1" ]; then
    make gconfig
  else
    if [ "$xconfig" = "1" ]; then
      make xconfig
    else
      if [ "$menuconfig" = "1" ]; then
        make menuconfig
      fi
    fi
  fi
  # rewrite configuration
  yes "" | make config >/dev/null
  make prepare # Necessary in case config has been changed

  # save configuration for later reuse
  if [ "${CARCH}" = "x86_64" ]; then
    cat .config > "${startdir}/config.x86_64.last"
  else
    cat .config > "${startdir}/config.last"
  fi

  ####################
  # stop here
  # this is useful to configure the kernel
  #msg "Stopping build"; return 1
  ####################

  # build!
  make ${MAKEFLAGS} LOCALVERSION= bzImage modules
}

package_linux-rt-ice() {

  KARCH=x86
  cd ${srcdir}/linux-${_basekernel}
  if [ "$keep_source_code" = "1" ]; then
    echo -n "Copying source code..."
    # Keep the source code
    cd $startdir
    mkdir -p $pkgdir/usr/src || return 1
    cp -a ${srcdir}/linux-${_basekernel} $pkgdir/usr/src/linux-${_kernver} || return 1

    #Add a link from the modules directory
    mkdir -p $pkgdir/lib/modules/${_kernver} || return 1
    cd $pkgdir/lib/modules/${_kernver} || return 1
    rm -f source
    ln -s ../../../usr/src/linux-${_kernver} source || return 1
    echo "OK"
  fi
   
  # get kernel version
  _kernver="$(make LOCALVERSION= kernelrelease)"

  mkdir -p "${pkgdir}"/{lib/modules,lib/firmware,boot}
  make LOCALVERSION= INSTALL_MOD_PATH="${pkgdir}" modules_install
  cp arch/$KARCH/boot/bzImage "${pkgdir}/boot/vmlinuz-${pkgname}"

  # add vmlinux
  install -D -m644 vmlinux "${pkgdir}/usr/src/linux-${_kernver}/vmlinux"

  # install fallback mkinitcpio.conf file and preset file for kernel
  install -D -m644 "${srcdir}/${pkgname}.preset" "${pkgdir}/etc/mkinitcpio.d/${pkgname}.preset"

  # set correct depmod command for install
  sed \
    -e  "s/KERNEL_NAME=.*/KERNEL_NAME=${_kernelname}/" \
    -e  "s/KERNEL_VERSION=.*/KERNEL_VERSION=${_kernver}/" \
    -i "${startdir}/${pkgname}.install"
  sed \
    -e "1s|'linux.*'|'${pkgbase}'|" \
    -e "s|ALL_kver=.*|ALL_kver=\"/boot/vmlinuz-${pkgbase}\"|" \
    -e "s|default_image=.*|default_image=\"/boot/initramfs-${pkgbase}.img\"|" \
    -e "s|fallback_image=.*|fallback_image=\"/boot/initramfs-${pkgbase}-fallback.img\"|" \
    -i "${pkgdir}/etc/mkinitcpio.d/${pkgname}.preset"

  # remove build and source links
  rm -f "${pkgdir}"/lib/modules/${_kernver}/{source,build}
  # remove the firmware
  rm -rf "${pkgdir}/lib/firmware"
  # gzip -9 all modules to save 100MB of space
  find "${pkgdir}" -name '*.ko' -exec gzip -9 {} \;
  # make room for external modules
  ln -s "../extramodules-${_basekernel}${_kernelname:--rt-ice}" "${pkgdir}/lib/modules/${_kernver}/extramodules"
  # add real version for building modules and running depmod from post_install/upgrade
  mkdir -p "${pkgdir}/lib/modules/extramodules-${_basekernel}${_kernelname:--rt-ice}"
  echo "${_kernver}" > "${pkgdir}/lib/modules/extramodules-${_basekernel}${_kernelname:--rt-ice}/version"

  # Now we call depmod...
  depmod -b "${pkgdir}" -F System.map "${_kernver}"

  # move module tree /lib -> /usr/lib
  mv "${pkgdir}/lib" "${pkgdir}/usr"


  install -dm755 "${pkgdir}/usr/lib/modules/${_kernver}"

  cd "${pkgdir}/usr/lib/modules/${_kernver}"
  ln -sf ../../../src/linux-${_kernver} build

  cd "${srcdir}/linux-${_basekernel}"
  install -D -m644 Makefile \
    "${pkgdir}/usr/src/linux-${_kernver}/Makefile"
  install -D -m644 kernel/Makefile \
    "${pkgdir}/usr/src/linux-${_kernver}/kernel/Makefile"
  install -D -m644 .config \
    "${pkgdir}/usr/src/linux-${_kernver}/.config"

  mkdir -p "${pkgdir}/usr/src/linux-${_kernver}/include"

  for i in acpi asm-generic config crypto drm generated linux math-emu \
    media mtd net pcmcia scsi sound trace video xen; do
    cp -a include/${i} "${pkgdir}/usr/src/linux-${_kernver}/include/"
  done

  # copy arch includes for external modules
  mkdir -p "${pkgdir}/usr/src/linux-${_kernver}/arch/x86"
  cp -a arch/x86/include "${pkgdir}/usr/src/linux-${_kernver}/arch/x86/"

  # copy files necessary for later builds, like nvidia and vmware
  cp Module.symvers "${pkgdir}/usr/src/linux-${_kernver}"
  cp -a scripts "${pkgdir}/usr/src/linux-${_kernver}"

  # fix permissions on scripts dir
  chmod og-w -R "${pkgdir}/usr/src/linux-${_kernver}/scripts"
  mkdir -p "${pkgdir}/usr/src/linux-${_kernver}/.tmp_versions"

  mkdir -p "${pkgdir}/usr/src/linux-${_kernver}/arch/${KARCH}/kernel"

  cp arch/${KARCH}/Makefile "${pkgdir}/usr/src/linux-${_kernver}/arch/${KARCH}/"

  if [ "${CARCH}" = "i686" ]; then
    cp arch/${KARCH}/Makefile_32.cpu "${pkgdir}/usr/src/linux-${_kernver}/arch/${KARCH}/"
  fi

  cp arch/${KARCH}/kernel/asm-offsets.s "${pkgdir}/usr/src/linux-${_kernver}/arch/${KARCH}/kernel/"

  # add headers for lirc package
  mkdir -p "${pkgdir}/usr/src/linux-${_kernver}/drivers/media/video"

  cp drivers/media/video/*.h  "${pkgdir}/usr/src/linux-${_kernver}/drivers/media/video/"

  for i in bt8xx cpia2 cx25840 cx88 em28xx pwc saa7134 sn9c102; do
    mkdir -p "${pkgdir}/usr/src/linux-${_kernver}/drivers/media/video/${i}"
    cp -a drivers/media/video/${i}/*.h "${pkgdir}/usr/src/linux-${_kernver}/drivers/media/video/${i}"
  done

  # add docbook makefile
  install -D -m644 Documentation/DocBook/Makefile \
    "${pkgdir}/usr/src/linux-${_kernver}/Documentation/DocBook/Makefile"

  # add dm headers
  mkdir -p "${pkgdir}/usr/src/linux-${_kernver}/drivers/md"
  cp drivers/md/*.h "${pkgdir}/usr/src/linux-${_kernver}/drivers/md"

  # add inotify.h
  mkdir -p "${pkgdir}/usr/src/linux-${_kernver}/include/linux"
  cp include/linux/inotify.h "${pkgdir}/usr/src/linux-${_kernver}/include/linux/"

  # add wireless headers
  mkdir -p "${pkgdir}/usr/src/linux-${_kernver}/net/mac80211/"
  cp net/mac80211/*.h "${pkgdir}/usr/src/linux-${_kernver}/net/mac80211/"

  # add dvb headers for external modules
  # in reference to:
  # http://bugs.archlinux.org/task/9912
  mkdir -p "${pkgdir}/usr/src/linux-${_kernver}/drivers/media/dvb/dvb-core"
  cp drivers/media/dvb/dvb-core/*.h "${pkgdir}/usr/src/linux-${_kernver}/drivers/media/dvb/dvb-core/"
  # and...
  # http://bugs.archlinux.org/task/11194
  mkdir -p "${pkgdir}/usr/src/linux-${_kernver}/include/config/dvb/"
  cp include/config/dvb/*.h "${pkgdir}/usr/src/linux-${_kernver}/include/config/dvb/"

  # add dvb headers for http://mcentral.de/hg/~mrec/em28xx-new
  # in reference to:
  # http://bugs.archlinux.org/task/13146
  mkdir -p "${pkgdir}/usr/src/linux-${_kernver}/drivers/media/dvb/frontends/"
  cp drivers/media/dvb/frontends/lgdt330x.h "${pkgdir}/usr/src/linux-${_kernver}/drivers/media/dvb/frontends/"
  cp drivers/media/video/msp3400-driver.h "${pkgdir}/usr/src/linux-${_kernver}/drivers/media/dvb/frontends/"

  # add dvb headers
  # in reference to:
  # http://bugs.archlinux.org/task/20402
  mkdir -p "${pkgdir}/usr/src/linux-${_kernver}/drivers/media/dvb/dvb-usb"
  cp drivers/media/dvb/dvb-usb/*.h "${pkgdir}/usr/src/linux-${_kernver}/drivers/media/dvb/dvb-usb/"
  mkdir -p "${pkgdir}/usr/src/linux-${_kernver}/drivers/media/dvb/frontends"
  cp drivers/media/dvb/frontends/*.h "${pkgdir}/usr/src/linux-${_kernver}/drivers/media/dvb/frontends/"
  mkdir -p "${pkgdir}/usr/src/linux-${_kernver}/drivers/media/common/tuners"
  cp drivers/media/common/tuners/*.h "${pkgdir}/usr/src/linux-${_kernver}/drivers/media/common/tuners/"

  # add xfs and shmem for aufs building
  mkdir -p "${pkgdir}/usr/src/linux-${_kernver}/fs/xfs"
  mkdir -p "${pkgdir}/usr/src/linux-${_kernver}/mm"
  cp fs/xfs/xfs_sb.h "${pkgdir}/usr/src/linux-${_kernver}/fs/xfs/xfs_sb.h"

  # copy in Kconfig files
  for i in `find . -name "Kconfig*"`; do
    mkdir -p "${pkgdir}"/usr/src/linux-${_kernver}/`echo ${i} | sed 's|/Kconfig.*||'`
    cp ${i} "${pkgdir}/usr/src/linux-${_kernver}/${i}"
  done

  chown -R root.root "${pkgdir}/usr/src/linux-${_kernver}"
  find "${pkgdir}/usr/src/linux-${_kernver}" -type d -exec chmod 755 {} \;

  # strip scripts directory
  find "${pkgdir}/usr/src/linux-${_kernver}/scripts" -type f -perm -u+w 2>/dev/null | while read binary ; do
    case "$(file -bi "${binary}")" in
      *application/x-sharedlib*) # Libraries (.so)
        /usr/bin/strip ${STRIP_SHARED} "${binary}";;
      *application/x-archive*) # Libraries (.a)
        /usr/bin/strip ${STRIP_STATIC} "${binary}";;
      *application/x-executable*) # Binaries
        /usr/bin/strip ${STRIP_BINARIES} "${binary}";;
    esac
  done

  # remove unneeded architectures
  if [ "$keep_source_code" = "0" ]; then
    rm -rf "${pkgdir}"/usr/src/linux-${_kernver}/arch/{alpha,arm,arm26,avr32,blackfin,c6x,cris,frv,h8300,hexagon,ia64,m32r,m68k,m68knommu,mips,microblaze,mn10300,openrisc,parisc,powerpc,ppc,s390,score,sh,sh64,sparc,sparc64,tile,unicore32,um,v850,xtensa}
  fi
}
