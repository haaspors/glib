project('glib', 'c', 'cpp', default_options : [ 'c_std=none'])

cc = meson.get_compiler('c')
cxx = meson.get_compiler('cpp')

major_version = 2
minor_version = 47
micro_version = 0
interface_age = 0
binary_age = 100 * minor_version + micro_version
glib_version = '@0@.@1@.@2@'.format(major_version, minor_version, micro_version)

inc_dirs = include_directories('.', 'glib', 'gmodule')
build_printf_gnulib = false

zlib = dependency('zlib')
libpcre = dependency('libpcre') # FIXME: Should check for Unicode support, too.
libffi = dependency('libffi', version : '>=3.0.0')


glib_conf = configuration_data()
glibconfig_conf = configuration_data()

glib_conf.set('GLIB_MAJOR_VERSION', major_version)
glib_conf.set('GLIB_MINOR_VERSION', minor_version)
glib_conf.set('GLIB_MICRO_VERSION', micro_version)
glib_conf.set('GLIB_INTERFACE_AGE', interface_age)
glib_conf.set('GLIB_BINARY_AGE', binary_age)
glib_conf.set('GLIB_VERSION', glib_version)

glibconfig_conf.set('GLIB_MAJOR_VERSION', major_version)
glibconfig_conf.set('GLIB_MINOR_VERSION', minor_version)
glibconfig_conf.set('GLIB_MICRO_VERSION', micro_version)


###############################################################################
# Threading
###############################################################################
# FIXME: windows ?
# FIXME: disable thread ?
#        glibconfig_conf.set('g_threads_impl_def', 'NONE')
if host_machine.system() == 'windows'
  glibconfig_conf.set('g_threads_impl_def', 'WIN32')
else
  glibconfig_conf.set('g_threads_impl_def', 'POSIX')
  if cc.has_header('pthread.h')
    glib_conf.set('HAVE_PTHREAD_H', 1)
    thread_lib = find_library('pthread')

    if cc.has_function('pthread_attr_setstacksize', prefix : '#include <pthread.h>')
      glib_conf.set('HAVE_PTHREAD_ATTR_SETSTACKSIZE', 1)
    endif
    if cc.has_function('pthread_condattr_setclock', prefix : '#include <pthread.h>')
      glib_conf.set('HAVE_PTHREAD_CONDATTR_SETCLOCK', 1)
    endif
    if cc.has_function('pthread_cond_timedwait_relative_np', prefix : '#include <pthread.h>')
      glib_conf.set('HAVE_PTHREAD_COND_TIMEDWAIT_RELATIVE_NP', 1)
    endif

    if cc.compiles('''#include <pthread.h>
        int main(int argc, char **argv) {
          (void)argc; (void)argv;
          pthread_setname_np("test");
          return 0;
        }''', name : 'pthread_setname_np("thread x")')
      glib_conf.set('HAVE_PTHREAD_SETNAME_NP_WITHOUT_TID', 1)
    endif
  else
    error('pthread not found')
  endif
endif

###############################################################################
# PLATFORM
###############################################################################
# FIXME: Check for glib_native_win32
if host_machine.system() == 'windows'
  glibconfig_conf.set('glib_os', '''#define G_OS_WIN32
#define G_PLATFORM_WIN32''')
else
  glibconfig_conf.set('glib_os', '#define G_OS_UNIX')
endif

if cc.has_header('Carbon/Carbon.h') and cc.has_header('CoreServices/CoreServices.h')
  glib_conf.set('HAVE_CARBON', 1)
  # FIXME: CARBON_LIBS="-Wl,-framework,Carbon"
endif

glib_detect_cocoa = '''
#include <Cocoa/Cocoa.h>
#ifdef GNUSTEP_BASE_VERSION
#error "Detected GNUstep, not Cocoa"
#endif
'''
if cc.compiles(glib_detect_cocoa)
  glib_conf.set('HAVE_COCOA', 1)
  # FIXME: COCOA_LIBS="-Wl,-framework,Foundation"
endif

###############################################################################
# iconv & gettext
###############################################################################
if cc.has_function('iconv_open', prefix : '#include <iconv.h>')
  libiconv = []
else
  libiconv = find_library('iconv')
  if 1 # FIXME: this is bogus, we need has_function with lib as argument
    glib_conf.set('USE_LIBICONV_NATIVE', 1)
  elif
    glib_conf.set('USE_LIBICONV_GNU', 1)
  endif
endif

if cc.has_header('libintl.h')
  if cc.has_function('ngettext', prefix : '#include <libintl.h>') and cc.has_function('dgettext', prefix : '#include <libintl.h>')
    libintl = []
  else
    libintl = find_library('intl')
    # FIXME: find gettext properly like in m4macros/glib-gettext.m4
    error('Support for gettext required in libc currently.')
  endif
else
  error('GLib requires gettext support (libintl)')
endif

glib_conf.set('HAVE_GETTEXT', 1)
glib_conf.set('ENABLE_NLS', 1)
glib_conf.set('GETTEXT_PACKAGE', '"glib20"')

if cc.has_header('locale.h')
  glib_conf.set('HAVE_LOCALE_H', 1)
  lc_msg_test = '''#include<locale.h>
void foo() { LC_MESSAGES; }
'''
  if cc.compiles(lc_msg_test, name : 'LC_MESSAGES')
    glib_conf.set('HAVE_LC_MESSAGES', 1)
  endif
endif
glib_conf.set('GLIB_LOCALE_DIR', '"share/locale"')


###############################################################################
# Configure checks
###############################################################################
sizeof_char       = cc.sizeof('char')
sizeof_short      = cc.sizeof('short')
sizeof_int        = cc.sizeof('int')
sizeof_long       = cc.sizeof('long')
sizeof_long_long  = cc.sizeof('long long')
if cc.has_type('__int64')
  sizeof_int64    = cc.sizeof('__int64')
else
  sizeof_int64    = 0
endif
sizeof_sizet      = cc.sizeof('size_t')
sizeof_ssizet     = cc.sizeof('ssize_t')
sizeof_voidp      = cc.sizeof('void*')

glib_conf.set('HAVE_LONG_LONG', 1)

align_char        = cc.alignment('char')
align_short       = cc.alignment('short')
align_int         = cc.alignment('int')
align_long        = cc.alignment('long')
align_long_long   = cc.alignment('long long')
align_sizet       = cc.alignment('size_t')
align_voidp       = cc.alignment('void*')

if sizeof_short == 2
  gint16 = 'short'
  gint16_modifier='"h"'
  gint16_format='"hi"'
  guint16_format='"hu"'
elif sizeof_int == 2
  gint16 = 'int'
  gint16_modifier='""'
  gint16_format='"i"'
  guint16_format='"u"'
else
  error('No suitable 16bit type found')
endif

if sizeof_int == 4
  gint32 = 'int'
  gint32_modifier='""'
  gint32_format='"i"'
  guint32_format='"u"'
  guint32_align = align_int
elif sizeof_long == 4
  gint32 = 'long'
  gint32_modifier='"l"'
  gint32_format='"li"'
  guint32_format='"lu"'
  guint32_align = align_long
elif sizeof_short == 4
  gint32 = 'short'
  gint32_modifier='"h"'
  gint32_format='"hi"'
  guint32_format='"hu"'
  guint32_align = align_short
else
  error('No suitable 32bit type found')
endif

if sizeof_long_long == 8
  gint64 = 'long long'
  gint64_modifier='"ll"'
  gint64_format='"lli"'
  guint64_format='"llu"'
  glib_extension=''
  gint64_constant='(val##LL)'
  guint64_constant='(val##ULL)'
  guint64_align = align_long_long
elif sizeof_long == 8
  gint64 = 'long'
  gint64_modifier='"l"'
  gint64_format='"li"'
  guint64_format='"lu"'
  glib_extension=''
  gint64_constant='(val##L)'
  guint64_constant='(val##UL)'
  guint64_align = align_long
elif sizeof_int == 8
  gint64 = 'int'
  gint64_modifier='""'
  gint64_format='"i"'
  guint64_format='"u"'
  glib_extension=''
  gint64_constant='(val)'
  guint64_constant='(val)'
  guint64_align = align_int
else
  error('GLib requires a 64 bit type.')
endif
glib_conf.set('SIZEOF_CHAR',      sizeof_char)
glib_conf.set('SIZEOF_SHORT',     sizeof_short)
glib_conf.set('SIZEOF_INT',       sizeof_int)
glib_conf.set('SIZEOF_LONG',      sizeof_long)
glib_conf.set('SIZEOF_LONG_LONG', sizeof_long_long)
glib_conf.set('SIZEOF_SIZE_T',    sizeof_sizet)
glib_conf.set('SIZEOF_VOID_P',    sizeof_voidp)

glib_conf.set('ALIGNOF_UNSIGNED_LONG', align_long)
glib_conf.set('ALIGNOF_GUINT32', guint32_align)
glib_conf.set('ALIGNOF_GUINT64', guint64_align)



glib_conf.set('HAVE_STDLIB_H', 1)
glib_conf.set('HAVE_STRING_H', 1)
glib_conf.set('STDC_HEADERS', 1)

sys_types_h = cc.has_header('sys/types.h')
sys_stat_h = cc.has_header('sys/stat.h')
strings_h = cc.has_header('strings.h')
inttypes_h = cc.has_header('inttypes.h')
stdint_h = cc.has_header('stdint.h')
unistd_h = cc.has_header('unistd.h')

default_includes = '''
#include <stddef.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
'''
if sys_types_h
  default_includes = default_includes + '#include <sys/types.h>\n'
endif
if sys_stat_h
  default_includes = default_includes + '#include <sys/stat.h>\n'
endif
if strings_h
  default_includes = default_includes + '#include <strings.h>\n'
endif
if inttypes_h
  default_includes = default_includes + '#include <inttypes.h>\n'
endif
if stdint_h
  default_includes = default_includes + '#include <stdint.h>\n'
endif
if unistd_h
  default_includes = default_includes + '#include <unistd.h>\n'
endif

if cc.compiles('#define __EXTENSIONS__ 1\n' + default_includes)
  glib_conf.set('__EXTENSIONS__', 1)
  glib_conf.set('_ALL_SOURCE', 1)
  glib_conf.set('_DARWIN_C_SOURCE', 1)
  glib_conf.set('_GNU_SOURCE', 1)
  glib_conf.set('_POSIX_PTHREAD_SEMANTICS', 1)
  glib_conf.set('_TANDEM_SOURCE', 1)
endif

sys_statfs_h = cc.has_header('sys/statfs.h')
sys_poll_h = cc.has_header('sys/poll.h')
alloca_h = cc.has_header('alloca.h')
if alloca_h
  glib_conf.set('HAVE_ALLOCA_H', 1)
  glib_conf.set('HAVE_ALLOCA', 1)
elif cc.has_function('alloca')
  glib_conf.set('HAVE_ALLOCA', 1)
endif

if cc.has_header('sys/param.h')
  glib_conf.set('HAVE_SYS_PARAM_H', 1)
  sysparam_include = '#include <sys/param.h>'
else
  sysparam_include = ''
endif
if cc.has_header('sys/resource.h')
  glib_conf.set('HAVE_SYS_RESOURCE_H', 1)
endif
if cc.has_header('mach/mach_time.h')
  glib_conf.set('HAVE_MACH_MACH_TIME_H', 1)
endif
if cc.has_header('sys/select.h')
  glib_conf.set('HAVE_SYS_SELECT_H', 1)
endif
if stdint_h
  glib_conf.set('HAVE_STDINT_H', 1)
endif
if inttypes_h
  glib_conf.set('HAVE_INTTYPES_H', 1)
endif
if cc.has_header('sched.h')
  glib_conf.set('HAVE_SCHED_H', 1)
endif
if cc.has_header('malloc.h')
  glib_conf.set('HAVE_MALLOC_H', 1)
endif
if cc.has_header('sys/vfs.h')
  glib_conf.set('HAVE_SYS_VFS_H', 1)
  sysvfs_include = '#include <sys/vfs.h>'
else
  sysvfs_include = ''
endif
if cc.has_header('sys/vmount.h')
  glib_conf.set('HAVE_SYS_VMOUNT_H', 1)
endif
if sys_statfs_h
  glib_conf.set('HAVE_SYS_STATFS_H', 1)
  sysstatfs_include = '#include <sys/statfs.h>'
else
  sysstatfs_include = ''
endif
sysstatvfs = cc.has_header('sys/statvfs.h')
if sysstatvfs
  glib_conf.set('HAVE_SYS_STATVFS_H', 1)
endif
if cc.has_header('sys/filio.h')
  glib_conf.set('HAVE_SYS_FILIO_H', 1)
endif
if cc.has_header('mntent.h')
  glib_conf.set('HAVE_MNTENT_H', 1)
endif
if cc.has_header('sys/mnttab.h')
  glib_conf.set('HAVE_SYS_MNTTAB_H', 1)
endif
if cc.has_header('sys/vfstab.h')
  glib_conf.set('HAVE_SYS_VFSTAB_H', 1)
endif
if cc.has_header('sys/mntctl.h')
  glib_conf.set('HAVE_SYS_MNTCTL_H', 1)
endif
if cc.has_header('fstab.h')
  glib_conf.set('HAVE_FSTAB_H', 1)
endif
if cc.has_header('linux/magic.h')
  glib_conf.set('HAVE_LINUX_MAGIC_H', 1)
endif
if cc.has_header('sys/prctl.h')
  glib_conf.set('HAVE_SYS_PRCTL_H', 1)
endif
dirent_h = cc.has_header('dirent.h')
if dirent_h
  glib_conf.set('HAVE_DIRENT_H', 1)
endif
if cc.has_header('sys/time.h')
  glib_conf.set('HAVE_SYS_TIME_H', 1)
endif
if cc.has_header('values.h')
  glib_conf.set('HAVE_VALUES_H', 1)
endif
if cc.has_header('sys/mount.h', prefix : sysparam_include)
  glib_conf.set('HAVE_SYS_MOUNT_H', 1)
  sysmount_include = '#include <sys/mount.h>'
else
  sysmount_include = ''
endif
if cc.has_header('sys/sysctl.h', prefix : sysparam_include)
  glib_conf.set('HAVE_SYS_SYSCTL_H', 1)
endif
if cc.has_function('sysctlbyname', prefix : '''#include <sys/types.h>
#include <sys/sysctl.h>''')
  glib_conf.set('HAVE_SYSCTLBYNAME', 1)
endif

# FIXME: Do something similar to AC_HEADER_MAJOR ?

if cc.has_header('xlocale.h')
  glib_conf.set('HAVE_XLOCALE_H', 1)
endif




if cc.has_function('mmap', prefix : '#include <sys/mman.h>')
  glib_conf.set('HAVE_MMAP', 1)
endif
posix_memalign = cc.has_function('posix_memalign', prefix : '#include <stdlib.h>')
if posix_memalign
  glib_conf.set('HAVE_POSIX_MEMALIGN', 1)
endif
if cc.has_function('memalign', prefix : '#include <malloc.h>')
  glib_conf.set('HAVE_MEMALIGN', 1)
endif
if cc.has_function('valloc', prefix : '#include <stdlib.h>')
  glib_conf.set('HAVE_VALLOC', 1)
endif
if cc.has_function('fsync', prefix : '#include <unistd.h>')
  glib_conf.set('HAVE_FSYNC', 1)
endif
if cc.has_function('pipe2', prefix : '''#define _GNU_SOURCE
#include <fcntl.h>
#include <unistd.h>
''')
  glib_conf.set('HAVE_PIPE2', 1)
endif
if cc.has_function('issetugid', prefix : '#include <unistd.h>')
  glib_conf.set('HAVE_ISSETUGID', 1)
endif
if cc.has_function('timegm', prefix : '#include <time.h>')
  glib_conf.set('HAVE_TIMEGM', 1)
endif
if cc.has_function('localtime_r', prefix : '#include <time.h>')
  glib_conf.set('HAVE_LOCALTIME_R', 1)
endif
if cc.has_function('gmtime_r', prefix : '#include <time.h>')
  glib_conf.set('HAVE_GMTIME_R', 1)
endif
if cc.has_function('strerror_r', prefix : '#include <string.h>')
  glib_conf.set('HAVE_STRERROR_R', 1)
endif

if cc.links('''#include <unistd.h>
    extern int __libc_enable_secure;
    int main() { return __libc_enable_secure; }''', name : '__libc_enable_secure')
  glib_conf.set('HAVE_LIBC_ENABLE_SECURE', 1)
endif
if cc.links('''#include <signal.h>
    #include <sys/types.h>
    sig_atomic_t val = 42;
    int main() { return val == 42 ? 0 : 1; }''', name : 'sig_atomic_t')
  glib_conf.set('HAVE_SIG_ATOMIC_T', 1)
endif

long_long_format = ''
if sizeof_long_long == 8
  sprintf_64b_format = '''#include <stdlib.h>
    #include <stdio.h>
    int main()
    {
      long long b, a = -0x3AFAFAFAFAFAFAFALL;
      char buffer[1000];
      sprintf (buffer, "%@0@u", a);
      sscanf (buffer, "%@0@u", &b);
      exit (b!=a);
    }
  '''
  foreach format : [ 'll', 'q', 'I64' ]
    if cc.run(sprintf_64b_format.format(format)).returncode() == 0
      long_long_format = format
      glib_conf.set('HAVE_LONG_LONG_FORMAT', 1)
      if format == 'I64'
        glib_conf.set('HAVE_INT64_AND_I64', 1)
      endif
    endif
  endforeach
elif sizeof_int64 == 8
  long_long_format = 'I64'
  glib_conf.set('HAVE_LONG_LONG_FORMAT', 1)
  glib_conf.set('HAVE_INT64_AND_I64', 1)
endif

stack_grows = cc.compiles('''
	volatile int *a = 0, *b = 0;
	void f (int i) { volatile int x = 5; if (i == 0) b = &x; else f (i - 1); }
	int main () { volatile int y = 7; a = &y; f (100); return b > a ? 0 : 1; }
''')

have_inline = cc.compiles('''#undef inline
	inline int foo () { return 0; }
	int main () { return foo (); }''', name : '  inline')
have___inline = cc.compiles('''
	__inline int foo () { return 0; }
	int main () { return foo (); }''', name : '__inline')
have___inline__ = cc.compiles('''
	__inline__ int foo () { return 0; }
	int main () { return foo (); }''', name : '__inline__')

iso_c99_varargs = cc.compiles('''int a(int p1, int p2, int p3);
	#define call_a(...) a(1,__VA_ARGS__)
	int main () { return call_a(2,3); }''', name : 'ISO C99 varargs')
iso_cxx_varargs = cxx.compiles('''int a(int p1, int p2, int p3);
	#define call_a(...) a(1,__VA_ARGS__)
	int main () { return call_a(2,3); }''', name : 'ISO C++ varargs')
gnuc_varargs = cc.compiles('''int a(int p1, int p2, int p3);
	#define call_a(params...) a(1,params)
	int main () { return call_a(2,3); }''', name : 'GNU C varargs')

gnuc_visibility = cc.compiles('''
void __attribute__ ((visibility ("hidden")))    f_hidden (void) {}
void __attribute__ ((visibility ("internal")))  f_internal (void) {}
void __attribute__ ((visibility ("protected"))) f_protected (void) {}
void __attribute__ ((visibility ("default")))   f_default (void) {}
int main () {
	f_hidden();
	f_internal();
	f_protected();
	f_default();
	return 0;
}
''', name : 'GNU C visibility')
sunstudio_visibility = cc.compiles('''
#if defined(__SUNPRO_C) || (__SUNPRO_C >= 0x550)
#else
# include "error: this is not Sun Studio."
#endif
''', name : 'Sun studio visibility')

if cc.has_member('struct stat', 'st_mtimensec', prefix : '#include<sys/stat.h>')
  glib_conf.set('HAVE_STRUCT_STAT_ST_MTIMENSEC', 1)
endif
if cc.has_member('struct stat', 'st_mtim.tv_nsec', prefix : '#include<sys/stat.h>')
  glib_conf.set('HAVE_STRUCT_STAT_ST_MTIM_TV_NSEC', 1)
endif
if cc.has_member('struct stat', 'st_atimensec', prefix : '#include<sys/stat.h>')
  glib_conf.set('HAVE_STRUCT_STAT_ST_ATIMENSEC', 1)
endif
if cc.has_member('struct stat', 'st_atim.tv_nsec', prefix : '#include<sys/stat.h>')
  glib_conf.set('HAVE_STRUCT_STAT_ST_ATIM_TV_NSEC', 1)
endif
if cc.has_member('struct stat', 'st_ctimensec', prefix : '#include<sys/stat.h>')
  glib_conf.set('HAVE_STRUCT_STAT_ST_CTIMENSEC', 1)
endif
if cc.has_member('struct stat', 'st_ctim.tv_nsec', prefix : '#include<sys/stat.h>')
  glib_conf.set('HAVE_STRUCT_STAT_ST_CTIM_TV_NSEC', 1)
endif
if cc.has_member('struct stat', 'st_birthtime', prefix : '#include<sys/stat.h>')
  glib_conf.set('HAVE_STRUCT_STAT_ST_BIRTHTIME', 1)
endif
if cc.has_member('struct stat', 'st_birthtimensec', prefix : '#include<sys/stat.h>')
  glib_conf.set('HAVE_STRUCT_STAT_ST_BIRTHTIMENSEC', 1)
endif
if cc.has_member('struct stat', 'st_birthtim', prefix : '#include<sys/stat.h>')
  glib_conf.set('HAVE_STRUCT_STAT_ST_BIRTHTIM', 1)
endif
if cc.has_member('struct stat', 'st_birthtim.tv_nsec', prefix : '#include<sys/stat.h>')
  glib_conf.set('HAVE_STRUCT_STAT_ST_BIRTHTIM_TV_NSEC', 1)
endif

blkprefix = '''#include <sys/types.h>
#include <sys/stat.h>
#include <unistd.h>
@0@
@1@
@2@
'''.format(sysstatfs_include, sysparam_include, sysmount_include)
if cc.has_member('struct stat', 'st_blksize', prefix : blkprefix)
  glib_conf.set('HAVE_STRUCT_STAT_ST_BLKSIZE', 1)
endif
if cc.has_member('struct stat', 'st_blocks', prefix : blkprefix)
  glib_conf.set('HAVE_STRUCT_STAT_ST_BLOCKS', 1)
endif
if cc.has_member('struct statfs', 'f_fstypename', prefix : blkprefix)
  glib_conf.set('HAVE_STRUCT_STATFS_F_FSTYPENAME', 1)
endif
statfs_f_bavail = cc.has_member('struct statfs', 'f_bavail', prefix : blkprefix)
if statfs_f_bavail
  glib_conf.set('HAVE_STRUCT_STATFS_F_BAVAIL', 1)
endif

if cc.has_member('struct statvfs', 'f_basetype', prefix : '#include <sys/statvfs.h>')
  glib_conf.set('HAVE_STRUCT_STATVFS_F_BASETYPE', 1)
endif
if cc.has_member('struct statvfs', 'f_fstypename', prefix : '#include <sys/statvfs.h>')
  glib_conf.set('HAVE_STRUCT_STATVFS_F_FSTYPENAME', 1)
endif

if cc.has_member('struct tm', 'tm_gmtoff', prefix : '#include <time.h>')
  glib_conf.set('HAVE_STRUCT_TM_TM_GMTOFF', 1)
endif
if cc.has_member('struct tm', '__tm_gmtoff', prefix : '#include <time.h>')
  glib_conf.set('HAVE_STRUCT_TM___TM_GMTOFF', 1)
endif

if dirent_h
  if cc.has_member('struct dirent', 'd_type', prefix : '''#include <sys/types.h>
#include <dirent.h>''')
    glib_conf.set('HAVE_STRUCT_DIRENT_D_TYPE', 1)
  endif
endif

if cc.compiles('''#include <langinfo.h>
    int main() { char * cs = nl_langinfo(CODESET); return (int)cs[0]; }''')
  glib_conf.set('HAVE_LANGINFO_CODESET', 1)
endif
if cc.compiles('''#include <langinfo.h>
    int main() {
      char * cs[] = {
        nl_langinfo(PM_STR), nl_langinfo(D_T_FMT), nl_langinfo(D_FMT),
        nl_langinfo(T_FMT), nl_langinfo(T_FMT_AMPM), nl_langinfo(MON_1),
        nl_langinfo(ABMON_12), nl_langinfo(DAY_1), nl_langinfo(ABDAY_7)
      };
      return (int)cs[0][0];
    }''')
  glib_conf.set('HAVE_LANGINFO_TIME', 1)
endif
if cc.compiles('''#include <langinfo.h>
    int main() {
      char * cs[] = {
        nl_langinfo(_NL_CTYPE_OUTDIGIT0_MB), nl_langinfo(_NL_CTYPE_OUTDIGIT1_MB),
        nl_langinfo(_NL_CTYPE_OUTDIGIT2_MB), nl_langinfo(_NL_CTYPE_OUTDIGIT3_MB),
        nl_langinfo(_NL_CTYPE_OUTDIGIT4_MB), nl_langinfo(_NL_CTYPE_OUTDIGIT5_MB),
        nl_langinfo(_NL_CTYPE_OUTDIGIT6_MB), nl_langinfo(_NL_CTYPE_OUTDIGIT7_MB),
        nl_langinfo(_NL_CTYPE_OUTDIGIT8_MB), nl_langinfo(_NL_CTYPE_OUTDIGIT9_MB)
      };
      return (int)cs[0][0];
    }''')
  glib_conf.set('HAVE_LANGINFO_OUTDIGIT', 1)
endif

if cc.has_function('lstat', prefix : '''#include <sys/types.h>
#include <sys/stat.h>
#include <unistd.h>''')
  glib_conf.set('HAVE_LSTAT', 1)
endif
if cc.has_function('strsignal', prefix : '#include <string.h>')
  glib_conf.set('HAVE_STRSIGNAL', 1)
endif
if cc.has_function('vsnprintf', prefix : '''#include <stdio.h>
#include <stdarg.h>''')
  glib_conf.set('HAVE_VSNPRINTF', 1)
endif
if cc.has_function('stpcpy', prefix : '#include <string.h>')
  glib_conf.set('HAVE_STPCPY', 1)
endif
if cc.has_function('strcasecmp', prefix : '#include <strings.h>')
  glib_conf.set('HAVE_STRCASECMP', 1)
endif
if cc.has_function('strncasecmp', prefix : '#include <strings.h>')
  glib_conf.set('HAVE_STRNCASECMP', 1)
endif
if cc.has_function('poll', prefix : '#include <poll.h>')
  glib_conf.set('HAVE_POLL', 1)
endif
if cc.has_function('vasprintf', prefix : '''#define _GNU_SOURCE
#include <stdio.h>
#include <stdarg.h>''')
  glib_conf.set('HAVE_VASPRINTF', 1)
endif
if cc.has_function('setenv', prefix : '#include <stdlib.h>')
  glib_conf.set('HAVE_SETENV', 1)
endif
if cc.has_function('unsetenv', prefix : '#include <stdlib.h>')
  glib_conf.set('HAVE_UNSETENV', 1)
endif
if cc.has_function('getc_unlocked', prefix : '#include <stdio.h>')
  glib_conf.set('HAVE_GETC_UNLOCKED', 1)
endif
if cc.has_function('readlink', prefix : '#include <unistd.h>')
  glib_conf.set('HAVE_READLINK', 1)
endif
if cc.has_function('symlink', prefix : '#include <unistd.h>')
  glib_conf.set('HAVE_SYMLINK', 1)
endif
if cc.has_function('fdwalk', prefix : '#include <stdlib.h>')
  glib_conf.set('HAVE_FDWALK', 1)
endif
if cc.has_function('memmem', prefix : '''#define _GNU_SOURCE
#include <string.h>''')
  glib_conf.set('HAVE_MEMMEM', 1)
endif

if cc.has_function('lchmod', prefix : '''#include <sys/stat.h>
#include <unistd.h>''')
  glib_conf.set('HAVE_LCHMOD', 1)
endif
if cc.has_function('lchown', prefix : '#include <unistd.h>')
  glib_conf.set('HAVE_LCHOWN', 1)
endif
if cc.has_function('fchmod', prefix : '#include <sys/stat.h>')
  glib_conf.set('HAVE_FCHMOD', 1)
endif
if cc.has_function('fchown', prefix : '#include <unistd.h>')
  glib_conf.set('HAVE_FCHOWN', 1)
endif
if cc.has_function('utimes', prefix : '''#include <sys/types.h>
#include <utime.h>
#include <sys/time.h>''')
  glib_conf.set('HAVE_UTIMES', 1)
endif
if cc.has_function('getresuid', prefix : '''#define _GNU_SOURCE
#include <unistd.h>''')
  glib_conf.set('HAVE_GETRESUID', 1)
endif
if cc.has_function('getmntent_r', prefix : '''#include <stdio.h>
#include <mntent.h>''')
  glib_conf.set('HAVE_GETMNTENT_R', 1)
endif
if cc.has_function('setmntent', prefix : '''#include <stdio.h>
#include <mntent.h>''')
  glib_conf.set('HAVE_SETMNTENT', 1)
endif
if cc.has_function('endmntent', prefix : '''#include <stdio.h>
#include <mntent.h>''')
  glib_conf.set('HAVE_ENDMNTENT', 1)
endif
if cc.has_function('hasmntopt', prefix : '''#include <stdio.h>
#include <mntent.h>''')
  glib_conf.set('HAVE_HASMNTOPT', 1)
endif
if cc.has_function('getfsstat', prefix : '''#include <sys/param.h>
#include <sys/ucred.h>
#include <sys/mount.h>''')
  glib_conf.set('HAVE_GETFSSTAT', 1)
endif
if cc.has_function('getvfsstat', prefix : '''#include <sys/types.h>
#include <sys/statvfs.h>''')
  glib_conf.set('HAVE_GETVFSSTAT', 1)
endif
if cc.has_function('fallocate', prefix : '''#define _GNU_SOURCE
#include <fcntl.h>''')
  glib_conf.set('HAVE_FALLOCATE', 1)
endif
if cc.has_function('splice', prefix : '''#define _GNU_SOURCE
#include <fcntl.h>''')
  glib_conf.set('HAVE_SPLICE', 1)
endif
if cc.has_function('prlimit', prefix : '''#include <sys/time.h>
#include <sys/resource.h>''')
  glib_conf.set('HAVE_PRLIMIT', 1)
endif

statfsfunc = cc.has_function('statfs', prefix : '#include <sys/statfs.h>')
if sys_statfs_h and statfsfunc
  glib_conf.set('HAVE_STATFS', 1)
  if statfs_f_bavail
    glib_conf.set('USE_STATFS', 1)
  endif
endif
if sysstatvfs
  if cc.has_function('statvfs', prefix : '#include <sys/statvfs.h>')
    glib_conf.set('HAVE_STATVFS', 1)
    if not statfs_f_bavail
      glib_conf.set('USE_STATVFS', 1)
    endif
  endif
endif

if cc.has_header('crt_externs.h')
  glib_conf.set('HAVE_CRT_EXTERNS_H', 1)
  if cc.has_function('_NSGetEnviron', prefix : '#include <crt_externs.h>')
    glib_conf.set('HAVE__NSGETENVIRON', 1)
  endif
endif

if cc.has_function('newlocale', prefix : '#include <locale.h>')
  glib_conf.set('HAVE_NEWLOCALE', 1)
endif
if cc.has_function('uselocale', prefix : '#include <locale.h>')
  glib_conf.set('HAVE_USELOCALE', 1)
endif

if cc.has_function('strtod_l', prefix : '''#include <stdlib.h>
#include <xlocale.h>''')
  glib_conf.set('HAVE_STRTOD_L', 1)
endif
if cc.has_function('strtoll_l', prefix : '''#include <stdlib.h>
#include <limits.h>
#include <xlocale.h>''')
  glib_conf.set('HAVE_STRTOLL_L', 1)
endif
if cc.has_function('strtoull_l', prefix : '''#include <stdlib.h>
#include <limits.h>
#include <xlocale.h>''')
  glib_conf.set('HAVE_STRTOULL_L', 1)
endif

if cc.has_function('getprotobyname_r', prefix : '#include <netdb.h>')
  glib_conf.set('HAVE_GETPROTOBYNAME_R', 1)
endif
if cc.has_function('endservent', prefix : '#include <netdb.h>')
  glib_conf.set('HAVE_ENDSERVENT', 1)
endif
if cc.has_function('if_nametoindex', prefix : '#include <net/if.h>')
  glib_conf.set('HAVE_IF_NAMETOINDEX', 1)
endif
if cc.has_function('if_indextoname', prefix : '#include <net/if.h>')
  glib_conf.set('HAVE_IF_INDEXTONAME', 1)
endif
if cc.has_function('sendmmsg', prefix : '''#define _GNU_SOURCE
#include <sys/socket.h>''')
  glib_conf.set('HAVE_SENDMMSG', 1)
endif
if cc.has_function('recvmmsg', prefix : '''#define _GNU_SOURCE
#include <sys/socket.h>''')
  glib_conf.set('HAVE_RECVMMSG', 1)
endif
if cc.has_header('linux/netlink.h')
  glib_conf.set('HAVE_NETLINK', 1)
endif
if cc.has_type('struct ip_mreqn', prefix : '#include <netinet/in.h>')
  glib_conf.set('HAVE_IP_MREQN', 1)
endif

if statfsfunc
  check_statfs_args_prefx = '''#include <unistd.h>
    @0@
    @1@
    @2@
    @3@
  '''.format(sysparam_include, sysvfs_include, sysmount_include, sysstatfs_include)

  if cc.compiles(check_statfs_args_prefx + 'int main() { struct statfs st; statfs(NULL, &st); return 0 }')
    glib_conf.set('STATFS_ARGS', 2)
  endif
  if cc.compiles(check_statfs_args_prefx + 'int main() { struct statfs st; statfs(NULL, &st, sizeof(st), 0); return 0 }')
    glib_conf.set('STATFS_ARGS', 4)
  endif
endif

if cc.compiles('''#include <fcntl.h>
      #include <sys/types.h>
      #include <sys/stat.h>
      int main() { return open(0, O_DIRECTORY, 0) > 0; }''')
  glib_conf.set('HAVE_OPEN_O_DIRECTORY', 1)
endif

if cc.run('''#include <stdio.h>
      #include <stdarg.h>
      int doit(char * s, ...)
      {
        char buffer[32];
        va_list args;
        int r;

        va_start(args, s);
        r = vsnprintf(buffer, 5, s, args);
        va_end(args);

        if (r != 7)
          return 1;

        /* AIX 5.1 and Solaris seems to have a half-baked vsnprintf()
           implementation. The above will return 7 but if you replace
           the size of the buffer with 0, it borks! */
        va_start(args, s);
        r = vsnprintf(buffer, 0, s, args);
        va_end(args);

        if (r != 7)
          return 1;

        return 0;
      }
      int main() { return doit("1234567"); }''').returncode() == 0
  glib_conf.set('HAVE_C99_VSNPRINTF', 1)
else
  build_printf_gnulib = true
endif
if cc.run('''#include <stdio.h>
      int main()
      {
        char buffer[32];

        if (snprintf(buffer, 5, "1234567") != 7)
          return 1;
        if (snprintf(buffer, 0, "1234567") != 7)
          return 1;
        if (snprintf(NULL, 0, "1234567") != 7)
          return 1;

        return 0;
      }''').returncode() == 0
  glib_conf.set('HAVE_C99_SNPRINTF', 1)
endif
if cc.run('''#include <stdio.h>
      int main(void)
      {
        char buffer[128];
        sprintf (buffer, "%2\$d %3\$d %1\$d", 1, 2, 3);
        return strcmp ("2 3 1", buffer);
      }''').returncode() == 0
  glib_conf.set('HAVE_UNIX98_PRINTF', 1)
else
  build_printf_gnulib = true
endif

if not build_printf_gnulib and sizeof_long_long == 8 and long_long_format == ''
  build_printf_gnulib = true
endif

if not build_printf_gnulib
  glib_conf.set('HAVE_GOOD_PRINTF', 1)
endif

# FIXME: checks needed for gnulib vasnprintf

if posix_memalign
  if cc.run('''#define _XOPEN_SOURCE 600
      #include <stdlib.h> /* posix_memalign() should be defined here */
                          /* some systems break if #include <malloc.h> used */
      static void test_memalign (size_t boundary, size_t size) {
          void *mem = 0;
          if (posix_memalign (&mem, boundary, size) != 0 || !mem)
            exit (1);
          free (mem);
      }
      int main() {
          test_memalign (  128,   128 - 2 * sizeof (void*));
          test_memalign (  256,   256 - 2 * sizeof (void*));
          test_memalign (  512,   512 - 2 * sizeof (void*));
          test_memalign ( 1024,  1024 - 2 * sizeof (void*));
          test_memalign ( 2048,  2048 - 2 * sizeof (void*));
          test_memalign ( 4096,  4096 - 2 * sizeof (void*));
          test_memalign ( 8192,  8192 - 2 * sizeof (void*));
          test_memalign (16384, 16384 - 2 * sizeof (void*));
          test_memalign (32768, 32768 - 2 * sizeof (void*));
          exit (0); /* success */
      }''').returncode() == 0
    glib_conf.set('POSIX_MEMALIGN_WITH_COMPLIANT_ALLOCS', 1)
  endif
endif

if cc.run('''#include <stdlib.h>
    #include <string.h>
    int main() {
      char p[10];
      (void) strlcpy (p, "hi", 10);
      if (strlcat (p, "bye", 0) != 3)
        return 1;
      return 0;
    }''').returncode() == 0
  glib_conf.set('HAVE_STRLCPY', 1)
endif

have_vacopy = cc.compiles('''#include <stdarg.h>
#include <stdlib.h>
void f (int i, ...) {
va_list args1, args2;
va_start (args1, i);
va_copy (args2, args1);
if (va_arg (args2, int) != 42 || va_arg (args1, int) != 42)
  exit (1);
va_end (args1); va_end (args2);
}
int main() {
  f (0, 42);
  return 0;
}
''')

have___vaacopy = cc.compiles('''#include <stdarg.h>
#include <stdlib.h>
void f (int i, ...) {
va_list args1, args2;
va_start (args1, i);
__va_copy (args2, args1);
if (va_arg (args2, int) != 42 || va_arg (args1, int) != 42)
  exit (1);
va_end (args1); va_end (args2);
}
int main() {
  f (0, 42);
  return 0;
}
''')

if not have_vacopy
  glib_conf.set('G_VA_COPY_AS_ARRAY', 1)
endif
if have_vacopy
  glib_conf.set('G_VA_COPY', 'va_copy')
elif have___vacopy
  glib_conf.set('G_VA_COPY', '__va_copy')
endif

if cc.run('''
    #include <errno.h>
    #include <pwd.h>
    int main () {
      char buffer[10000];
      struct passwd pwd, *pwptr = &pwd;
      int error;
      errno = 0;
      error = getpwuid_r (0, &pwd, buffer, sizeof (buffer), &pwptr);
      return (error < 0 && errno == ENOSYS) || error == ENOSYS;
    }''', name : 'getpwuid_r (POSIX)').returncode() == 0
  glib_conf.set('HAVE_POSIX_GETPWUID_R', 1)
elif cc.links('''#include <pwd.h>
    int main () {
      char buffer[10000];
      struct passwd pwd;
      getpwuid_r (0, &pwd, buffer, sizeof (buffer));
      return 0;
    }''', name : 'getpwuid_r (non-POSIX)')
  glib_conf.set('HAVE_NONPOSIX_GETPWUID_R', 1)
endif

# FIXME: GModule checks

# FIXME: GSpawn checks

# FIXME: GIO checks



###############################################################################
# Configure glibconfig.h
###############################################################################
glibconfig_conf.set('gintbits',   sizeof_int * 8)
glibconfig_conf.set('glongbits',  sizeof_long * 8)
glibconfig_conf.set('gsizebits',  sizeof_sizet * 8)
glibconfig_conf.set('gssizebits', sizeof_ssizet * 8)

glibconfig_conf.set('gint16', gint16)
glibconfig_conf.set('gint16_modifier', gint16_modifier)
glibconfig_conf.set('gint16_format', gint16_format)
glibconfig_conf.set('guint16_format', guint16_format)
glibconfig_conf.set('gint32', gint32)
glibconfig_conf.set('gint32_modifier', gint32_modifier)
glibconfig_conf.set('gint32_format', gint32_format)
glibconfig_conf.set('guint32_format', guint32_format)

glibconfig_conf.set('gint64', gint64)
glibconfig_conf.set('gint64_modifier', gint64_modifier)
glibconfig_conf.set('gint64_format', gint64_format)
glibconfig_conf.set('guint64_format', guint64_format)
glibconfig_conf.set('gint64_constant', gint64_constant)
glibconfig_conf.set('guint64_constant', guint64_constant)

if host_machine.system() == 'windows'
  glibconfig_conf.set('g_module_suffix', 'dll')
  glibconfig_conf.set('g_pid_type', 'void*')
else
  glibconfig_conf.set('g_module_suffix', 'so')
  glibconfig_conf.set('g_pid_type', 'int')
endif

if sizeof_sizet == sizeof_short
  glibconfig_conf.set('glib_size_type_define', 'short')
  glibconfig_conf.set('gsize_modifier', '"h"')
  glibconfig_conf.set('gsize_format', '"hu"')
  glibconfig_conf.set('gssize_format', '"hi"')
  glibconfig_conf.set('glib_msize_type', 'SHRT')
elif sizeof_sizet == sizeof_int
  glibconfig_conf.set('glib_size_type_define', 'int')
  glibconfig_conf.set('gsize_modifier', '""')
  glibconfig_conf.set('gsize_format', '"u"')
  glibconfig_conf.set('gssize_format', '"i"')
  glibconfig_conf.set('glib_msize_type', 'INT')
elif sizeof_sizet == sizeof_long
  glibconfig_conf.set('glib_size_type_define', 'long')
  glibconfig_conf.set('gsize_modifier', '"l"')
  glibconfig_conf.set('gsize_format', '"lu"')
  glibconfig_conf.set('gssize_format', '"li"')
  glibconfig_conf.set('glib_msize_type', 'LONG')
elif sizeof_sizet == sizeof_long_long
  glibconfig_conf.set('glib_size_type_define', 'long long')
  glibconfig_conf.set('gsize_modifier', '"I64"')
  glibconfig_conf.set('gsize_format', '"I64u"')
  glibconfig_conf.set('gssize_format', '"I64i"')
  glibconfig_conf.set('glib_msize_type', 'INT64')
else
  error('Could not determine size of size_t.')
endif

if sizeof_voidp == sizeof_int
  glibconfig_conf.set('glib_intptr_type_define', 'int')
  glibconfig_conf.set('gintptr_modifier', '""')
  glibconfig_conf.set('gintptr_format', '"i"')
  glibconfig_conf.set('guintptr_format', '"u"')
  glibconfig_conf.set('glib_gpi_cast', '(gint)')
  glibconfig_conf.set('glib_gpui_cast', '(guint)')
elif sizeof_voidp == sizeof_long
  glibconfig_conf.set('glib_intptr_type_define', 'long')
  glibconfig_conf.set('gintptr_modifier', '"l"')
  glibconfig_conf.set('gintptr_format', '"li"')
  glibconfig_conf.set('guintptr_format', '"lu"')
  glibconfig_conf.set('glib_gpi_cast', '(glong)')
  glibconfig_conf.set('glib_gpui_cast', '(gulong)')
elif sizeof_voidp == sizeof_long_long
  glibconfig_conf.set('glib_intptr_type_define', 'long long')
  glibconfig_conf.set('gintptr_modifier', '"I64"')
  glibconfig_conf.set('gintptr_format', '"I64i"')
  glibconfig_conf.set('guintptr_format', '"I64u"')
  glibconfig_conf.set('glib_gpi_cast', '(gint64)')
  glibconfig_conf.set('glib_gpui_cast', '(guint64)')
else
  error('Could not determine size of void*.')
endif

glibconfig_conf.set('glib_void_p', sizeof_voidp)
glibconfig_conf.set('glib_long', sizeof_long)
glibconfig_conf.set('glib_size_t', sizeof_sizet)
if host_machine.endian() == 'big'
  glibconfig_conf.set('g_byte_order', 'G_BIG_ENDIAN')
  glibconfig_conf.set('g_bs_native', 'BE')
  glibconfig_conf.set('g_bs_alien', 'LE')
else
  glibconfig_conf.set('g_byte_order', 'G_LITTLE_ENDIAN')
  glibconfig_conf.set('g_bs_native', 'LE')
  glibconfig_conf.set('g_bs_alien', 'BE')
endif

if have_inline
  glibconfig_conf.set10('G_HAVE_INLINE', 1)
endif
if have___inline
  glibconfig_conf.set10('G_HAVE___INLINE', 1)
endif
if have___inline__
  glibconfig_conf.set10('G_HAVE___INLINE__', 1)
endif
# FIXME: this isn't absolutly correct...
if have_inline or have___inline or have___inline__
  glibconfig_conf.set10('G_CAN_INLINE', 1)
endif

if iso_c99_varargs
  glibconfig_conf.set('g_have_iso_c_varargs', '''#ifndef __cplusplus
# define G_HAVE_ISO_VARARGS 1
#endif''')
else
  glibconfig_conf.set('g_have_iso_c_varargs', '')
endif

if iso_cxx_varargs
  glibconfig_conf.set('g_have_iso_cxx_varargs', '''#ifdef __cplusplus
# define G_HAVE_ISO_VARARGS 1
#endif''')
else
  glibconfig_conf.set('g_have_iso_cxx_varargs', '')
endif

if gnuc_varargs
  glibconfig_conf.set('g_have_gnuc_varargs', '''
/* gcc-2.95.x supports both gnu style and ISO varargs, but if -ansi
 * is passed ISO vararg support is turned off, and there is no work
 * around to turn it on, so we unconditionally turn it off.
 */
#if __GNUC__ == 2 && __GNUC_MINOR__ == 95
#  undef G_HAVE_ISO_VARARGS
#endif

#define G_HAVE_GNUC_VARARGS 1
''')
else
  glibconfig_conf.set('g_have_gnuc_varargs', '')
endif


if have_vacopy
  glibconfig_conf.set('G_VA_COPY', 'va_copy')
elif have___vacopy
  glibconfig_conf.set('G_VA_COPY', '__va_copy')
endif

if stack_grows
  glibconfig_conf.set('G_HAVE_GROWING_STACK', 1)
endif

if gnuc_visibility
  glibconfig_conf.set('G_HAVE_GNUC_VISIBILITY', 1)
endif
if alloca_h
  glibconfig_conf.set('GLIB_HAVE_ALLOCA_H', 1)
endif
if sys_poll_h
  glibconfig_conf.set('GLIB_HAVE_SYS_POLL_H', 1)
endif

if sys_poll_h and sys_types_h
  templ = '''#include <sys/poll.h>
#include <sys/types.h>
#include <stdio.h>
int main(int argc, char **argv) {
  printf("%d\n", (int)@0@);
  return 0;
}'''
  value_POLLIN = cc.run(templ.format('POLLIN')).stdout().strip()
  value_POLLOUT = cc.run(templ.format('POLLOUT')).stdout().strip()
  value_POLLPRI = cc.run(templ.format('POLLPRI')).stdout().strip()
  value_POLLERR = cc.run(templ.format('POLLERR')).stdout().strip()
  value_POLLHUP = cc.run(templ.format('POLLHUP')).stdout().strip()
  value_POLLNVAL = cc.run(templ.format('POLLNVAL')).stdout().strip()
else
  value_POLLIN = 1
  value_POLLOUT = 4
  value_POLLPRI = 2
  value_POLLERR = 8
  value_POLLHUP = 16
  value_POLLNVAL = 32
endif

glibconfig_conf.set('g_pollin', value_POLLIN)
glibconfig_conf.set('g_pollout', value_POLLOUT)
glibconfig_conf.set('g_pollpri', value_POLLPRI)
glibconfig_conf.set('g_pollerr', value_POLLERR)
glibconfig_conf.set('g_pollhup', value_POLLHUP)
glibconfig_conf.set('g_pollnval', value_POLLNVAL)

glibconfig_conf.set('GLIB_USING_SYSTEM_PRINTF', 1)

# FIXME: Add archs!
if host_machine.cpu() != 'x86' and host_machine.cpu() != 'x86_64' and host_machine.cpu() != 'arm'
  glibconfig_conf.set('G_ATOMIC_OP_MEMORY_BARRIER_NEEDED', 1)
endif

if cc.compiles('''void func() {
  volatile int atomic = 2;
  __sync_bool_compare_and_swap (&atomic, 2, 3);
}''')
  glibconfig_conf.set('G_ATOMIC_LOCK_FREE', 1)
endif

configure_file(input : 'config.h.meson', output : 'config.h', configuration : glib_conf)

subdir('glib')
subdir('tests')

