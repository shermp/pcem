# configure.ac for PCem
#

AC_PREREQ([2.61])
m4_include([m4/ax_compare_version.m4])
AC_INIT(PCem, v17, Sarah Walker <pcem@pcem-emulator.co.uk>, pcem)
: ${CFLAGS=""}
: ${CXXFLAGS=""}

AC_CANONICAL_HOST

AC_PROG_CC
AC_PROG_CXX

AC_MSG_CHECKING([whether to build for release])
AC_ARG_ENABLE(release_build,
          AC_HELP_STRING([--enable-release-build], [build for release]))
if test "$enable_release_build" == "yes"; then
   AC_DEFINE([RELEASE_BUILD])
   AC_MSG_RESULT([yes])
else
   AC_MSG_RESULT([no])
fi

AC_MSG_CHECKING([whether to enable debugging])
AC_ARG_ENABLE(debug,
          AC_HELP_STRING([--enable-debug], [build debug executable]))
if test "$enable_debug" == "yes"; then
   CFLAGS="-Wall -O0 -g -fcommon"
   CXXFLAGS="-Wall -O0 -g -fcommon"
   AC_DEFINE([_DEBUG])
   AC_MSG_RESULT([yes])
else
   CFLAGS="-O3 -fcommon"
   CXXFLAGS="-O3 -fcommon"
   AC_MSG_RESULT([no])
fi

AC_MSG_CHECKING([whether to statically link wxWidgets])
AC_ARG_ENABLE(wx_static,
          AC_HELP_STRING([--enable-wx-static], [enable wxWidgets static linking]))
if test "$enable_wx_static" == "yes"; then
   AC_MSG_RESULT([yes])
else
   AC_MSG_RESULT([no])
fi

AC_MSG_CHECKING([whether to enable networking])
AC_ARG_ENABLE(networking,
          AC_HELP_STRING([--disable-networking], [disable networking support]))
if test "$enable_networking" != "no"; then
   AC_SUBST([USE_NETWORKING], [1])
   AC_MSG_RESULT([yes])
else
   AC_MSG_RESULT([no])
fi

AC_MSG_CHECKING([whether to use ALSA for MIDI output])
AC_ARG_ENABLE(alsa,
          AC_HELP_STRING([--disable-alsa], [don't ALSA for MIDI on Linux]))
if test "$enable_alsa" != "no"; then
   AC_MSG_RESULT([yes])
else
   AC_MSG_RESULT([no])
fi

AC_MSG_CHECKING([for cpu])
case "${host_cpu}" in
    i?86)
    CFLAGS="$CFLAGS -march=i686 -msse2"
    CXXFLAGS="$CXXFLAGS -march=i686 -msse2"
    AC_SUBST([CPU], [i386])
    AC_MSG_RESULT(${host_cpu})
    ;;
    x86_64)
    AC_SUBST([CPU], [x86_64])
    AC_MSG_RESULT(${host_cpu})
    ;;
    armv7l)
    AC_SUBST([CPU], [arm])
    AC_MSG_RESULT(${host_cpu})
    ;;
    aarch64)
    AC_SUBST([CPU], [arm64])
    AC_MSG_RESULT(${host_cpu})
    ;;
    *)
    AC_MSG_ERROR([Unsupported CPU. ${host_cpu}])
    ;;
esac

AC_MSG_CHECKING([for off64_t $CPU])
AC_LINK_IFELSE([AC_LANG_PROGRAM([[#include <sys/types.h>]],[[off64_t n=0;]])],[has_lfs="yes"],[has_lfs="no"])
case "$host" in
    *-*-freebsd*) # has off64_t but no fopen64/etc.
       has_lfs="no"
       ;;
esac
if test "$has_lfs" == "no"; then
   AC_DEFINE([off64_t],[off_t])
   AC_DEFINE([fopen64],[fopen])
   AC_DEFINE([fseeko64],[fseek])
   AC_DEFINE([ftello64],[ftell])
fi
AC_MSG_RESULT($has_lfs)

#AC_MSG_CHECKING([for libz])
#AX_CHECK_ZLIB

AC_CHECK_LIB([pthread], [pthread_create])

build_os="other"

case "$host" in
    *-*-darwin*)
       LIBS="$LIBS -framework OpenGL"
       LIBS="$LIBS -framework OpenAL"
       AC_DEFINE([PCEM_RENDER_WITH_TIMER])
       AC_DEFINE([PCEM_RENDER_TIMER_LOOP])
       build_os="macosx"
       ;;
    *-*-cygwin* | *-*-mingw32*)
       LIBS="$LIBS -lwinmm"
       test "$enable_networking" != "no" && LIBS="$LIBS -lwsock32 -liphlpapi"
       build_os="win"
       AC_SUBST(SET_MAKE, 'MAKE=mingw32-make')
       ;;
    *-*-linux*)
       build_os="linux"
       ;;
    *-*-freebsd*)
       CFLAGS="$CFLAGS -I/usr/local/include"
       CXXFLAGS="$CXXFLAGS -I/usr/local/include"
       LDFLAGS="$LDFLAGS -L/usr/local/lib"
       ;;
    *)
       ;;
esac

AC_CHECK_PROG(sdl_cfg, [sdl2-config], [yes], [no])
if test "$sdl_cfg" == "yes"; then 
   sdl_ver=$(sdl2-config --version)
   AX_COMPARE_VERSION($sdl_ver,[ge],[2.0.1],
   [AC_MSG_NOTICE([Ok])],
   [AC_MSG_ERROR([SLD2 version < 2.0.1])])

   CFLAGS="$CFLAGS $(sdl2-config --cflags)"
   LIBS="$LIBS $(sdl2-config --libs)"
else
   AC_MSG_ERROR([sdl2-config not found.])
fi

AC_CHECK_PROG(wx_cfg, [wx-config], [yes], [no])
if test "$wx_cfg" == "yes"; then
   wx_inst=$(wx-config --version)
   AX_COMPARE_VERSION($wx_inst,[ge],[3.0.0],
   [AC_MSG_NOTICE([Ok])],
   [AC_MSG_ERROR([WX version < 3.0.0])])

   wx_cfg=wx-config
   test "$enable_wx_static" == "yes" && wx_cfg="wx-config --static"

   CFLAGS="$CFLAGS $(${wx_cfg} --cflags)"
   CXXFLAGS="$CXXFLAGS $(${wx_cfg} --cxxflags)"
   LIBS="$LIBS $(${wx_cfg} --libs)"

   if test "$build_os" == "win"; then
      windres=$(wx-config --rescomp)
      AC_SUBST([WINDRES], [$windres])
   fi
else
   AC_MSG_ERROR([wx-config not found.])
fi

# ALSA only makes sense on Linux
if test "$build_os" == "linux" && test "$enable_alsa" != "no" ; then
   AC_CHECK_LIB(asound, snd_pcm_open, [],
            [echo "You need to install the ALSA library."
             exit -1])
   AC_SUBST([USE_ALSA], [1])
fi
if test "$build_os" == "win"; then
   AC_CHECK_LIB([opengl32], [main], [],
           [echo "You need to install the OpenGL library."
            exit -1])
else
   AC_CHECK_LIB([GL], [glGetError], [],
           [echo "You need to install the OpenGL library."
            exit -1])
fi
AC_CHECK_LIB([openal], [alGetError], [],
          [echo "You need to install the OpenAL library."
           exit -1])

AC_SUBST([BUILD_OS],[$build_os])

AC_OUTPUT([Makefile])
