#                                               -*- Autoconf -*-
# Process this file with autoconf to produce a configure script.

AC_PREREQ([2.63])
AC_INIT([gearbox], [0.1], [gearbox@corybennett.org])
AC_CONFIG_SRCDIR([apache/mod_gearbox/mod_gearbox.cc])
AC_CONFIG_MACRO_DIR([m4])

AC_CONFIG_HEADERS([config.h])
AC_CONFIG_AUX_DIR([auto])

# Checks for programs.
AC_PROG_CXX
AC_PROG_CC
AC_PROG_MAKE_SET
AC_PROG_LIBTOOL

AC_TYPE_LONG_LONG_INT
AC_CHECK_SIZEOF([long])
AC_CHECK_SIZEOF([long long])

AC_ARG_VAR([SHASUM_BIN])
AC_PATH_PROGS([SHASUM_BIN], [sha1sum shasum])
if test -z "${SHASUM_BIN}"; then
   echo "sha1sum or shasum command not found, required for tests"
   exit 1
fi
AC_DEFINE_UNQUOTED([SHASUM_BIN], ["${SHASUM_BIN}"], [Path to sha1sum binary])

AC_ARG_VAR([TRUE_BIN])
AC_PATH_PROG([TRUE_BIN],    [true])
if test -z "${TRUE_BIN}"; then
   echo "true command not found, required for tests"
   exit 1
fi
AC_DEFINE_UNQUOTED([TRUE_BIN], ["${TRUE_BIN}"], [path to true binary])

AC_ARG_VAR([FALSE_BIN])
AC_PATH_PROG([FALSE_BIN],   [false])
if test -z "${FALSE_BIN}"; then
   echo "false command not found, required for tests"
   exit 1
fi
AC_DEFINE_UNQUOTED([FALSE_BIN], ["${FALSE_BIN}"], [path to false binary])

AC_ARG_VAR([TRUE_BIN])
AC_PATH_PROG([TRUE_BIN],   [true])
if test -z "${TRUE_BIN}"; then
   echo "true command not found, required for tests"
   exit 1
fi
AC_DEFINE_UNQUOTED([TRUE_BIN], ["${TRUE_BIN}"], [path to true binary])

AC_DEFINE_UNQUOTED([LOCALSTATEDIR], ["`test "${prefix}" = "NONE" && prefix=; eval echo ${datarootdir}`"], [path gearbox stores state files])
AC_DEFINE_UNQUOTED([SYSCONFDIR], ["`test "${prefix}" = "NONE" && prefix=; eval echo ${sysconfdir}`"], [path to system configuration files])
AC_DEFINE_UNQUOTED([DATADIR], ["`test "${prefix}" = "NONE" && prefix=; eval echo ${datarootdir}`"], [path to data directory])
AC_DEFINE_UNQUOTED([PLUGINSDIR], ["`test "${exec_prefix}" = "NONE" && exec_prefix=${prefix}; test "${exec_prefix}" = "NONE" && exec_prefix=; eval echo ${libexecdir}/gearbox/plugins`"], [path to data directory])

AC_DEFINE_UNQUOTED([SHLIB_EXT], ["`eval echo ${shrext_cmds}`"], [shared libary extension])

AX_BOOST_BASE([1.49], [], [exit -1])
AX_BOOST_PROGRAM_OPTIONS
AX_BOOST_SYSTEM
AX_BOOST_FILESYSTEM
AX_BOOST_GRAPH
AX_BOOST_REGEX
AX_BOOST_IOSTREAMS

PKG_CHECK_MODULES([LOG4CXX], [liblog4cxx], [], [exit 1])
PKG_CHECK_MODULES([YAJL], [yajl], [], [exit 1])
PKG_CHECK_MODULES([CURL], [libcurl], [], [exit 1])
# on OSX:
# export PKG_CONFIG_PATH=/usr/local/opt/ossp-uuid/lib/pkgconfig
PKG_CHECK_MODULES([UUID], [uuid], [], [exit 1])
PKG_CHECK_MODULES([GEARMAN], [gearmand], [], [exit 1])

# Checks for libraries.
AC_CHECK_LIB([crypto], [main], [
  AC_ARG_VAR([CRYPTO_LIB])
  CRYPTO_LIB=-lcrypto
], [exit 1])

# FIXME: Replace `main' with a function in `-lzookeeper_st':
AC_CHECK_LIB([zookeeper_st], [main], [], [])

# apache stuff from https://github.com/timfel/mod_auth_openid/blob/master/configure.ac
# this will look for apxs command - put it in $APXS, fail on failure
AX_WITH_APXS()
# find apr-config binary
AC_ARG_WITH(apr_config, AC_HELP_STRING([[--with-apr-config=FILE]], [Path to apr-config program]),
			[ apr_config="$withval" ],
			[AC_PATH_PROGS(apr_config,
				[apr-config apr-0-config apr-1-config], 
				[no], 
				[$PATH:/usr/sbin/:/usr/local/apache2/bin]
			)]
)

if test "$apr_config" = "no"; then
   AC_MSG_ERROR(Could not find the apr-config program.  You can specify a location with the --with-apr-config=FILE option.  It may be named apr-0-config or apr-1-config and can be found in your apache2 bin directory.)
fi

$apr_config --cppflags &> /dev/null
if test "$?" != "0"; then
  AC_MSG_ERROR($apr_config is not a valid apr-config program)
fi

APR_LDFLAGS="`${apr_config} --link-ld --libs`"
AC_SUBST(APR_LDFLAGS)

APACHE_CFLAGS="-I`${APXS} -q INCLUDEDIR` -I`${apr_config} --includedir`"
AC_SUBST(APACHE_CFLAGS)

# soci
AC_CHECK_LIB([soci_core], [main], [
  AC_ARG_VAR([SOCI_CORE_LIB])
  SOCI_CORE_LIB=-lsoci_core
], [exit 1])


AC_PROG_SWIG(2.0.0)

# perl stuff
AC_PATH_PROG([PERL_BIN], [perl])
AC_ARG_VAR([PERL_ARCHNAME])
PERL_ARCHNAME=`${PERL_BIN} -MConfig -e 'print $Config{archname}'`
AC_ARG_VAR([PERL_ARCH])
PERL_ARCH=`${PERL_BIN} -MConfig -e 'print $Config{archflags}'`
AC_ARG_VAR([PERL_LDFLAGS])
PERL_LDFLAGS=`${PERL_BIN} -MExtUtils::Embed -e ldopts`
AC_ARG_VAR([PERL_CFLAGS])
PERL_CFLAGS=`${PERL_BIN} -MExtUtils::Embed -e ccopts`
AC_ARG_VAR([PERL_SITELIBDIR])
PERL_SITELIBDIR=`${PERL_BIN} -MConfig -e 'print $Config{sitelib}'`
AC_ARG_VAR([PERL_SITEARCHDIR])
PERL_SITEARCHDIR=`${PERL_BIN} -MConfig -e 'print $Config{sitearch}'`

# php stuff
AC_ARG_WITH([php], AS_HELP_STRING([--with-php], [enable building with php]), [
  AC_PATH_PROG([PHP_CONFIG], [php-config])
  AC_ARG_VAR([PHP_CFLAGS])
  PHP_CFLAGS=`${PHP_CONFIG} --includes`
  AC_ARG_VAR([PHP_LDFLAGS])
  PHP_LDFLAGS=`${PHP_CONFIG} --ldflags`
],[])
AM_CONDITIONAL([WITH_PHP], [test x$with_php = xtrue])

# Checks for header files.
AC_CHECK_HEADERS([arpa/inet.h fcntl.h limits.h stdint.h stdlib.h string.h sys/file.h sys/ioctl.h sys/socket.h sys/time.h unistd.h])

# Checks for typedefs, structures, and compiler characteristics.
AC_HEADER_STDBOOL
AC_C_INLINE
AC_TYPE_INT32_T
AC_TYPE_INT64_T
AC_TYPE_MODE_T
AC_TYPE_PID_T
AC_TYPE_SIZE_T
AC_TYPE_UID_T
AC_TYPE_UINT32_T
AC_TYPE_UINT64_T

# Checks for library functions.
AC_FUNC_ERROR_AT_LINE
AC_FUNC_FORK
AC_FUNC_MALLOC
AC_CHECK_FUNCS([dup2 floor gettimeofday memset mkfifo select socket strerror])

AM_INIT_AUTOMAKE([1.11 subdir-objects no-define foreign])

AC_CONFIG_FILES([Makefile])
AC_OUTPUT


