AC_PREREQ([2.64])
AC_INIT([miltertest], [1.0.0], [https://github.com/thegushi/miltertest/issues])
AC_CONFIG_SRCDIR([miltertest.c])
AC_CONFIG_HEADERS([build-config.h])
AC_CONFIG_AUX_DIR([build-aux])
AC_CONFIG_MACRO_DIR([m4])
AM_INIT_AUTOMAKE([foreign])

#
# Checks for programs
#
AC_PROG_CC
AC_PROG_INSTALL

#
# pthreads -- needed by libmilter
#
AC_SEARCH_LIBS([pthread_create], [pthread])
PTHREAD_CFLAGS="-pthread"
AC_SUBST([PTHREAD_CFLAGS])

#
# libnsl -- inet_addr on some older systems
#
saved_LIBS="$LIBS"
LIBS=""
AC_SEARCH_LIBS([inet_addr], [nsl])
LIBNSL_LIBS="$LIBS"
LIBS="$saved_LIBS"
AC_SUBST([LIBNSL_LIBS])

#
# Network functions
#
AC_SEARCH_LIBS([socket], [socket])
AC_SEARCH_LIBS([inet_pton], [resolv],
               AC_DEFINE([HAVE_INET_PTON], 1,
                         [Define to 1 if you have the inet_pton() function.]))
AC_SEARCH_LIBS([inet_ntop], [resolv nsl],
               AC_DEFINE([HAVE_INET_NTOP], 1,
                         [Define to 1 if you have the inet_ntop() function.]))
AC_SEARCH_LIBS([getaddrinfo], [resolv],
               AC_DEFINE([HAVE_GETADDRINFO], 1,
                         [Define to 1 if you have the getaddrinfo() function.]))

#
# Types
#
AC_CHECK_TYPES([useconds_t])

#
# strlcpy / strlcat
#
bsdstrl_h_found="no"
strl_h_found="no"
libstrl_found="no"
strl_found="no"

AC_CHECK_HEADERS([bsd/string.h], [bsdstrl_h_found="yes"])

AC_CHECK_FUNCS([strlcat strlcpy], [strl_found="yes"],
               [
                 saved_LIBS="$LIBS"
                 AC_SEARCH_LIBS([strlcat], [bsd strl],
                                [libstrl_found="yes"], [strl_found="no"])
                 AC_SEARCH_LIBS([strlcpy], [bsd strl],
                                [libstrl_found="yes"], [strl_found="no"])
                 STRL_LIBS="$LIBS"
                 LIBS="$saved_LIBS"
               ])

if test x"$libstrl_found" = x"yes"
then
	LIBS="$LIBS $STRL_LIBS"
fi

if test x"$strl_found" != x"yes" -a x"$libstrl_found" != x"yes"
then
	AC_MSG_ERROR([no strlcpy/strlcat found])
fi

AC_CHECK_HEADERS([strl.h], [strl_h_found="yes"])
if test x"$strl_h_found" != x"yes"
then
	strldirs="/usr/local/include/strl /usr/local/include /usr/include/strl"
	for d in $strldirs
	do
		AC_MSG_CHECKING([for strl.h in $d])
		if test -f $d/strl.h
		then
			unset ac_cv_header_strl_h
			saved_CPPFLAGS="$CPPFLAGS"
			CPPFLAGS="$CPPFLAGS -I$d"
			AC_CHECK_HEADERS([strl.h], [strl_h_found="yes"])
			CPPFLAGS="$saved_CPPFLAGS"
		fi
		if test x"$strl_h_found" = x"yes"
		then
			CPPFLAGS="$CPPFLAGS -I$d"
			break
		fi
	done
fi

if test x"$strl_h_found" = x"yes" -a x"$bsdstrl_h_found" = x"no"
then
	AC_DEFINE([USE_STRL_H], 1,
	          [Define to 1 if you need <strl.h> for strlcat()/strlcpy().])
fi

if test x"$bsdstrl_h_found" = x"yes"
then
	AC_DEFINE([USE_BSD_H], 1,
	          [Define to 1 if you need <bsd/string.h> for strlcat()/strlcpy().])
fi

#
# struct sockaddr_un.sun_len
#
AC_CHECK_MEMBER([struct sockaddr_un.sun_len],
                AC_DEFINE([HAVE_SUN_LEN], 1,
                          [Define if sockaddr_un has a sun_len member]),
                [],
                [[#include <sys/un.h>]])

#
# libmilter
#
AC_MSG_CHECKING([for milter library and includes])
AC_ARG_WITH([milter],
            AS_HELP_STRING([--with-milter],
                           [location of milter includes and library]),
            [milterpath="$withval"], [milterpath="auto"])

if test "$milterpath" = "auto" -o "$milterpath" = "yes"
then
	milterdirs="/usr/local /opt/local /usr"
	for d in $milterdirs
	do
		if test -f $d/include/libmilter/mfapi.h
		then
			milterpath=$d
			break
		fi
	done
fi

milter_explicit="no"
case "$milterpath" in
	auto)
		AC_MSG_ERROR([libmilter not found])
		;;
	*/*)
		if ! test -f $milterpath/include/libmilter/mfapi.h
		then
			AC_MSG_ERROR([milter includes not found at $milterpath])
		fi
		AC_MSG_RESULT([$milterpath])
		milter_explicit="yes"
		;;
	*)
		AC_MSG_ERROR([libmilter not found])
		;;
esac

LIBMILTER_INCDIRS="-I$milterpath/include"
LIBMILTER_LIBDIRS=""
LIBMILTER_LIBS=""

saved_CC="$CC"
saved_CFLAGS="$CFLAGS"
saved_CPPFLAGS="$CPPFLAGS"
saved_LDFLAGS="$LDFLAGS"
saved_LIBS="$LIBS"

CC="$CC $PTHREAD_CFLAGS"
CPPFLAGS="$LIBMILTER_INCDIRS $saved_CPPFLAGS"
CFLAGS="$PTHREAD_CFLAGS $saved_CFLAGS"

if test x"$milter_explicit" = x"yes"
then
	LIBMILTER_LIBDIRS="-L$milterpath/lib"
	LIBMILTER_LIBS="-lmilter"
	LDFLAGS="$PTHREAD_CFLAGS $LIBMILTER_LIBDIRS $saved_LDFLAGS"
	LIBS="$PTHREAD_LIBS $LIBMILTER_LIBS $saved_LIBS"
else
	LIBS="$PTHREAD_LIBS $saved_LIBS"
	LDFLAGS="$PTHREAD_CFLAGS $saved_LDFLAGS"
	breakloop="no"
	for d in lib lib64 lib/libmilter
	do
		unset ac_cv_search_smfi_register
		LDFLAGS="$PTHREAD_CFLAGS -L$milterpath/$d $saved_LDFLAGS"
		AC_SEARCH_LIBS([smfi_register], [milter],
		               [
		               	LIBMILTER_LIBDIRS="-L$milterpath/$d"
		               	LIBMILTER_LIBS="-lmilter"
		               	breakloop="yes"
		               ])
		if test x"$breakloop" = x"yes"
		then
			break
		fi
	done
	if test x"$LIBMILTER_LIBDIRS" = x""
	then
		AC_MSG_ERROR([libmilter not found])
	fi
	LDFLAGS="$PTHREAD_CFLAGS $LIBMILTER_LIBDIRS $saved_LDFLAGS"
	LIBS="$PTHREAD_LIBS $LIBMILTER_LIBS $saved_LIBS"
fi

CC="$saved_CC"
CFLAGS="$saved_CFLAGS"
CPPFLAGS="$saved_CPPFLAGS"
LDFLAGS="$saved_LDFLAGS"
LIBS="$saved_LIBS"

AC_SUBST([LIBMILTER_INCDIRS])
AC_SUBST([LIBMILTER_LIBDIRS])
AC_SUBST([LIBMILTER_LIBS])

#
# Lua (required)
#
# Detection strategy: try pkg-config first (versioned names in preference
# order), then fall back to AC_SEARCH_LIBS for systems without pkg-config.
# The user can pass LUA_CFLAGS and LUA_LIBS on the command line to override.
#
lua_found="no"

AS_IF([test x"$PKG_CONFIG" != x""], [
  for luapkg in lua lua5.4 lua-5.4 lua5.3 lua-5.3 lua5.2 lua-5.2 lua5.1 lua-5.1
  do
    AS_IF([test x"$lua_found" != x"yes"], [
      PKG_CHECK_MODULES([LUA], [$luapkg], [lua_found="yes"], [:])
    ])
  done
  AS_IF([test x"$lua_found" = x"no"],
    [AC_MSG_WARN([pkg-config for Lua not found, trying manual search...])])
])

AS_IF([test x"$lua_found" != x"yes"], [
  AC_SEARCH_LIBS([lua_load],
                 [lua lua5.4 lua-5.4 lua5.3 lua-5.3 lua5.2 lua-5.2 lua5.1 lua-5.1],
    [ lua_found="yes"
      AS_IF([test "x$ac_cv_search_lua_load" != "xnone required"],
        [LUA_LIBS="$ac_cv_search_lua_load"])
    ])
])

AS_IF([test x"$lua_found" != x"yes"],
  [AC_MSG_ERROR([Lua not found - miltertest requires Lua 5.1 or later])])

saved_CPPFLAGS="$CPPFLAGS"
CPPFLAGS="$CPPFLAGS $LUA_CFLAGS"
AC_MSG_CHECKING([Lua version])
AC_COMPILE_IFELSE([AC_LANG_SOURCE([
#include <lua.h>
#if !defined(LUA_VERSION_NUM) || LUA_VERSION_NUM < 501
# error Lua version 5.1 or later is required
#endif
int main(int argc, char** argv) { return 0; }
])],
  AC_MSG_RESULT([ok]),
  AC_MSG_ERROR([Lua version 5.1 or later required]))
CPPFLAGS="$saved_CPPFLAGS"

AC_SUBST([LUA_CFLAGS])
AC_SUBST([LUA_LIBS])

AC_CONFIG_FILES([Makefile])
AC_OUTPUT
