dnl Copyright © 2008-2013 Pioneer Developers. See AUTHORS.txt for details
dnl Licensed under the terms of the GPL v3. See COPYING.txt for details

dnl Process this file with autoconf to create configure.

dnl ###########################################################################
dnl Setup
dnl ###########################################################################
AC_PREREQ(2.59)

AC_INIT([pioneer], [alpha-32-dev], [pioneer-dev@pioneerspacesim.net])
dnl AC_CONFIG_HEADERS([config.h])

AC_DEFUN([MXE_SETUP], [
    AC_ARG_WITH([mxe], AS_HELP_STRING([--with-mxe], [Path to MXE MinGW environment for cross-building]), [MXE_BASE=$withval], [MXE_BASE=])
	MXE_CROSS=
	MXE_CMAKE_TOOLCHAIN_FILE=""
	if test "x$MXE_BASE" != "x" ; then
		if test "x$host" != "x" ; then
			AC_MSG_ERROR([Cannot specify --host with --with-mxe])
		fi
		AC_MSG_CHECKING([for MXE])
		MXE_CMAKE_TOOLCHAIN_FILE="$MXE_BASE/usr/i686-pc-mingw32/share/cmake/mxe-conf.cmake"
		if ! test -f "$MXE_CMAKE_TOOLCHAIN_FILE" ; then
			AC_MSG_RESULT([no])
			AC_MSG_ERROR([MXE not found in $MXE_BASE])
		fi
		AC_MSG_RESULT([$MXE_BASE])
		MXE_CROSS=yes
		MXE_PATH="$MXE_BASE/usr/bin"
		PATH="$MXE_PATH:$PATH"
		host_alias=i686-pc-mingw32
		ac_tool_prefix=$host_alias-
		cross_compiling=yes
		HAVE_WINDRES=yes
		MXE_WINDRES="$MXE_BASE/usr/bin/${ac_tool_prefix}windres"
	fi
	AM_CONDITIONAL([HAVE_WINDRES], [test "$HAVE_WINDRES" = yes])
	AC_SUBST(MXE_BASE)
	AC_SUBST(MXE_CROSS)
	AC_SUBST(MXE_PATH)
	AC_SUBST(MXE_CMAKE_TOOLCHAIN_FILE)
	AC_SUBST(MXE_WINDRES)
])

MXE_SETUP

AM_INIT_AUTOMAKE([1.9 foreign dist-bzip2 dist-xz dist-zip])

AC_CANONICAL_HOST

AC_PROG_CC
AC_PROG_CXX
AC_C_CONST
AC_PROG_RANLIB
#AC_PROG_LIBTOOL

m4_ifdef([AM_SILENT_RULES], [AM_SILENT_RULES])

dnl ###########################################################################
dnl Flags and options
dnl ###########################################################################

EXTRA_CFLAGS=
EXTRA_CXXFLAGS=
EXTRA_CPPFLAGS=

WARN_CFLAGS=
WARN_CXXFLAGS=
WARN_CPPFLAGS=

dnl Check for ccache if they want it
AC_ARG_WITH([ccache], AS_HELP_STRING([--with-ccache], [Compile using ccache]), [AC_PATH_PROG([CCACHE], [ccache])])
if test "x$with_ccache" = xyes; then
	if test "x$CCACHE" = x; then
		AC_MSG_WARN([ccache was requested but was not found. Build will proceed as normal])
	fi
fi

dnl Add a flag, if supported by the compiler or preprocessor
dnl Assumes that -Werror works
AC_DEFUN([_PIONEER_COMPILER_FLAG], dnl Language, prefix (EXTRA), flags variable 'foo' (EXTRA_fooFLAGS), flag, $CC/$CXX/$CPP/similar, if yes, if no
    [AC_LANG_PUSH([$1])
     saved_flags="$][$3][FLAGS"
     ][$3][FLAGS="$saved_flags -Werror $4"
     AC_MSG_CHECKING([to see if $5 supports $4])
     AC_COMPILE_IFELSE([AC_LANG_PROGRAM([[]], [[return 0;]])], [][$2][_][$3][FLAGS="$][$2][_][$3][FLAGS $4"; AC_MSG_RESULT([yes])]m4_default([; $6], []), [AC_MSG_RESULT([no])]m4_default([; $7], []))
     ][$3][FLAGS="$saved_flags"
     AC_LANG_POP([$1])
    ])

AC_DEFUN([PIONEER_CFLAG], [_PIONEER_COMPILER_FLAG([C], [EXTRA], [C], [$1], [$CC], [$2], [$3])])
AC_DEFUN([PIONEER_CXXFLAG], [_PIONEER_COMPILER_FLAG([C++], [EXTRA], [CXX], [$1], [$CXX], [$2], [$3])])
AC_DEFUN([PIONEER_CPPFLAG], [_PIONEER_COMPILER_FLAG([CPP], [EXTRA], [CPP], [$1], [$CPP], [$2], [$3])])

AC_DEFUN([PIONEER_C_CXX_FLAG],
	[_PIONEER_COMPILER_FLAG([C], [EXTRA], [C], [$1], [$CC/$CXX], [EXTRA_CXXFLAGS="$EXTRA_CXXFLAGS $1"])])

AC_DEFUN([PIONEER_WARN_CFLAG], [_PIONEER_COMPILER_FLAG([C], [WARN], [C], [$1], [$CC], [$2], [$3])])
AC_DEFUN([PIONEER_WARN_CXXFLAG], [_PIONEER_COMPILER_FLAG([C++], [WARN], [CXX], [$1], [$CXX], [$2], [$3])])
AC_DEFUN([PIONEER_WARN_CPPFLAG], [_PIONEER_COMPILER_FLAG([CPP], [WARN], [CPP], [$1], [$CPP], [$2], [$3])])

AC_DEFUN([PIONEER_WARN_C_CXX_FLAG],
	[_PIONEER_COMPILER_FLAG([C], [WARN], [C], [$1], [$CC/$CXX], [WARN_CXXFLAGS="$WARN_CXXFLAGS $1"])])

dnl Always add -Wall, where supported
PIONEER_WARN_C_CXX_FLAG([-Wall])

dnl Add -O0 if requested
OPTIMISE=-O3
AC_ARG_WITH([no-optimise], AS_HELP_STRING([--with-no-optimise], [Compile without optimisation]), [test "$withval" != no && OPTIMISE=-O0], [])
AC_SUBST([OPTIMISE])

dnl Debug stuff
AC_ARG_ENABLE([debug], AS_HELP_STRING([--enable-debug], [Enable debugging code (stack checks, debug tools features, etc)]), [EXTRA_CPPFLAGS="$EXTRA_CPPFLAGS -DDEBUG"])
AC_ARG_WITH([fno-inline],  AS_HELP_STRING([--with-fno-inline], [Compile without inlining. Helps debugging segaults that occur in STL code]), [PIONEER_C_CXX_FLAG([-fno-inline])])
AC_ARG_WITH([gprof],       AS_HELP_STRING([--with-gprof], [Compile with gprof profiler]), [PIONEER_C_CXX_FLAG([-pg])])

dnl Dist versioning
AC_ARG_WITH([extra-version], AS_HELP_STRING([--with-extra-version], [Add text to game version shown on splash screen]), [EXTRA_CPPFLAGS="$EXTRA_CPPFLAGS -DPIONEER_EXTRAVERSION=\\\"$withval\\\""])

dnl Compiler warning options
AC_ARG_WITH([extra-warnings], AS_HELP_STRING([--without-extra-warnings], [Disable lots of extra compiler warnings]), [], [with_extra_warnings=yes])
if test "x$with_extra_warnings" != xno; then
  for warn_opt in \
-Wformat				\
-Wstrict-aliasing=2			\
-Wmissing-format-attribute		\
-Wmissing-noreturn			\
-Wdisabled-optimization			\
-Wfloat-equal				\
-Wshadow				\
-Wcast-qual				\
-Wcast-align				\
  ; do
    PIONEER_WARN_C_CXX_FLAG([$warn_opt])
  done
  dnl -Wformat-security requires -Wformat first, so it needs a slightly customised test
  dnl we assume that if this test passes then the test (above) for -Wformat passed
  dnl and therefore -Wformat should already be in WARN_CXXFLAGS
  _PIONEER_COMPILER_FLAG([C], [WARN], [C], [-Wformat -Wformat-security], [$CC/$CXX], [WARN_CXXFLAGS="$WARN_CXXFLAGS -Wformat-security"])
  for warn_opt in \
-Wno-format-zero-length			\
-Werror-implicit-function-declaration	\
  ; do
    PIONEER_WARN_CFLAG([$warn_opt])
  done
  for warn_opt in \
-Wstrict-null-sentinel			\
-Wold-style-cast			\
-Wsign-promo				\
  ; do
    PIONEER_WARN_CXXFLAG([$warn_opt])
  done
fi

dnl Data files directory
if test "$PIONEER_DATA_DIR" = ''; then
  AC_MSG_WARN([Using the default value for PIONEER_DATA_DIR.])
  AC_MSG_WARN([Installing is NOT recommended.])
  AC_MSG_WARN([IF YOU ARE PACKAGING PIONEER, SET THIS VARIABLE!])
  AC_MSG_WARN([To install: make install DESTDIR=/path/to/install/dir/])
  AC_MSG_WARN([The trailing path separator is required.])
  AC_MSG_WARN([The installed Pioneer is only runnable from that directory.])
fi
PIONEER_DATA_DIR=${PIONEER_DATA_DIR:-data}
AC_ARG_VAR([PIONEER_DATA_DIR], [where Pioneer expects its data files to be [data]. Packagers MUST set this!])

dnl ###########################################################################
dnl Library checks
dnl ###########################################################################

dnl FIXME: should really check for headers (if no pkgconfig)
AC_DEFUN([PIONEER_CHECK_LIB],
    [PKG_CHECK_MODULES([$1], [$2], [],
	[AC_CHECK_LIB([$3], [$4], [], [AC_MSG_ERROR([$1 is required])])
	 ][$1][_CFLAGS=
	 ][$1][_LIBS="-l$3"
	 AC_SUBST([$1][_CFLAGS])
	 AC_SUBST([$1][_LIBS])
	])
    ])

AC_ARG_WITH([thirdparty], AS_HELP_STRING([--with-thirdparty], [Location of pioneer-thirdparty repository]), [
	AC_MSG_NOTICE([using pioneer-thirdparty repository in $withval])
    if test "x$MXE_CROSS" = "xyes" ; then
	    PKG_CONFIG_PATH_i686_pc_mingw32="$withval/usr/lib/pkgconfig:$PKG_CONFIG_PATH"
	    export PKG_CONFIG_PATH_i686_pc_mingw32
    else
	    PKG_CONFIG_PATH="$withval/usr/lib/pkgconfig:$PKG_CONFIG_PATH"
	    export PKG_CONFIG_PATH
    fi
	CFLAGS="-isystem $withval/usr/include"
	LIBS="-L$withval/usr/lib -Wl,-rpath,$withval/usr/lib"
])

dnl Check for sigc++
PKG_CHECK_MODULES([SIGC], [sigc++-2.0])

dnl Check for SDL
PKG_CHECK_MODULES([SDL], [sdl SDL_image])

dnl Check for Freetype
PKG_CHECK_MODULES([FREETYPE], [freetype2])

dnl Check for libvorbis
PKG_CHECK_MODULES([VORBIS], [vorbisfile])

dnl Check for libpng
PKG_CHECK_MODULES([PNG], [libpng])

dnl Check for assimp
PKG_CHECK_MODULES([ASSIMP], [assimp])

dnl assimp's pkg-config file has a broken include path. Until upstream fixes
dnl it we work around it here
ASSIMP_CFLAGS=`echo $ASSIMP_CFLAGS | sed -e 's/\/assimp$//'`

if test "x$cross_compiling" = "xno" ; then

	dnl Extra version checks for assimp. Last release was 2.0, we need 3.0+ (git
	dnl version). Unfortunately the pkg-config version has not yet been updated
	dnl upstream, so we have to test the library directly
	AC_MSG_CHECKING(for ASSIMP version >= 3.0)
	dnl Convert -Lfoo to -rpath=, as the wanted dir might not be visible to the dynamic linker
	assimp_rpath=`echo $ASSIMP_LIBS | sed -e 's/-[^L] *[^ ]\+ *//g' | sed -e 's/-L */-Wl,-rpath,/g'`
	old_cxxflags="$CXXFLAGS"
	old_libs="$LIBS"
	CXXFLAGS="$ASSIMP_CFLAGS"
	LIBS="$ASSIMP_LIBS $assimp_rpath"
	AC_LANG_PUSH([C++])
	AC_RUN_IFELSE([AC_LANG_PROGRAM([[#include <assimp/defs.h>
	                                 #include <assimp/version.h>]],
	                            [[do {
	                                if (aiGetVersionMajor() >= 3 && aiGetVersionMinor() >= 0)
	                                    return 0;
	                                return 1;
	                              } while(0)]])],
	                            [AC_MSG_RESULT(yes)
	                                have_assimp_version=yes],
	                            [AC_MSG_RESULT(no)])
	AC_LANG_POP
	if test "x$have_assimp_version" != "xyes" ; then
		AC_MSG_ERROR("assimp version >= 3.0 not found. Get it from https://assimp.sourceforge.net/")
	fi
	CXXFLAGS="$old_cxxflags"
	LIBS="$old_libs"

fi

dnl Optionally use external liblua
AC_ARG_WITH([external-liblua], AS_HELP_STRING([--with-external-liblua], [Use external liblua in place of our internal copy]), [], [with_external_liblua=no; HAVE_LUA=no])
if test "$with_external_liblua" = yes; then
  PKG_CHECK_MODULES([LUA], [lua >= 5.2], [HAVE_LUA=yes], [foundlua=no])
  if test "$foundlua" = no; then
    PKG_CHECK_MODULES([LUA], [lua5.2 >= 5.2], [HAVE_LUA=yes], [AC_MSG_ERROR([External liblua was requested but is not available])])
  fi
fi
AM_CONDITIONAL([HAVE_LUA], [test "$HAVE_LUA" = yes])
case "$host" in
  i686-pc-mingw32|x86_64-pc-mingw64)
	;;
  *)
	LUA_CFLAGS="$LUA_CFLAGS -DLUA_USE_POSIX"
esac

dnl Check for OpenGL libs
if test "x$MXE_CROSS" = "xyes" ; then
	AC_SUBST([GL_CFLAGS], [])
	AC_SUBST([GL_LIBS], [-lopengl32])
	AC_SUBST([GLU_CFLAGS], [])
	AC_SUBST([GLU_LIBS], [-lglu32])
	AC_SUBST([GLEW_CFLAGS], [-DGLEW_BUILD])
	AC_SUBST([GLEW_LIBS], [-lglew32s])
else
	PIONEER_CHECK_LIB([GL],   [gl],   [GL],   [glBegin])
	PIONEER_CHECK_LIB([GLU],  [glu],  [GLU],  [gluSphere])
	PIONEER_CHECK_LIB([GLEW], [glew], [GLEW], [glewInit])
fi

dnl ###########################################################################
dnl Platform selection
dnl ###########################################################################

AC_MSG_CHECKING([platform type])
case "$host" in
  i686-pc-mingw32|x86_64-pc-mingw64)
	PLATFORM=win32
	;;
  *)
	PLATFORM=posix
	;;
esac
AC_SUBST([PLATFORM])
AC_MSG_RESULT($PLATFORM)

AM_CONDITIONAL([BUILD_WIN32], [test "x-$PLATFORM" = "x-win32"])
AM_CONDITIONAL([BUILD_POSIX], [test "x-$PLATFORM" = "x-posix"])

dnl ###########################################################################
dnl System-specific stuff not covered above
dnl ###########################################################################

case "$host" in
  i686-pc-mingw32|x86_64-pc-mingw64)
	MINGW_LIBS="-lshlwapi"
	SDL_LIBS="$SDL_LIBS -ltiff -lpng -ljpeg -lz"
	;;
  *)
	MINGW_LIBS=
	;;
esac
AC_SUBST([MINGW_LIBS])

dnl ###########################################################################
dnl Finished :-)
dnl ###########################################################################

dnl if ccache was wanted, update the compiler vars to match
dnl this is done late so that configure tests aren't run under ccache
if test "x$CCACHE" != x; then
    CC="$CCACHE $CC"
    CXX="$CCACHE $CXX"
fi

AC_SUBST([EXTRA_CFLAGS])
AC_SUBST([EXTRA_CXXFLAGS])
AC_SUBST([EXTRA_CPPFLAGS])

AC_SUBST([WARN_CFLAGS])
AC_SUBST([WARN_CXXFLAGS])
AC_SUBST([WARN_CPPFLAGS])

AC_CONFIG_FILES([
	Makefile
	data/Makefile
	src/Makefile
	src/collider/Makefile
	src/gameui/Makefile
	src/gui/Makefile
	src/graphics/Makefile
	src/scenegraph/Makefile
	src/galaxy/Makefile
	src/terrain/Makefile
	src/text/Makefile
	src/ui/Makefile
	src/win32/Makefile
	src/posix/Makefile
	contrib/Makefile
	contrib/lua/Makefile
	contrib/miniz/Makefile
	contrib/jenkins/Makefile
	contrib/vcacheopt/Makefile
	osx/Makefile
	win32/Makefile])
AC_OUTPUT
