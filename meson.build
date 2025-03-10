project('fwupd', 'c',
  version: '1.9.5',
  license: 'LGPL-2.1+',
  meson_version: '>=0.61.0',
  default_options: ['warning_level=2', 'c_std=c11'],
)

fwupd_version = meson.project_version()
varr = fwupd_version.split('.')
fwupd_major_version = varr[0]
fwupd_minor_version = varr[1]
fwupd_micro_version = varr[2]

conf = configuration_data()
conf.set('MAJOR_VERSION', fwupd_major_version)
conf.set('MINOR_VERSION', fwupd_minor_version)
conf.set('MICRO_VERSION', fwupd_micro_version)
conf.set_quoted('PACKAGE_VERSION', fwupd_version)

# get source version, falling back to package version
git = find_program('git', required: false)
tag = false
if git.found()
  source_version = run_command([git, 'describe'], check: false).stdout().strip()
  if source_version == ''
    source_version = fwupd_version
  endif
  tag = run_command([git, 'describe', '--exact-match'], check: false).returncode() == 0
else
  source_version = fwupd_version
endif
conf.set_quoted('SOURCE_VERSION', source_version)

# libtool versioning - this applies to libfwupd
#
# See http://sources.redhat.com/autobook/autobook/autobook_91.html#SEC91 for details
#
# - If interfaces have been changed or added, but binary compatibility
#   has been preserved, change:
#      CURRENT += 1
#      REVISION = 0
#      AGE += 1
# - If binary compatibility has been broken (eg removed or changed
#   interfaces), change:
#      CURRENT += 1
#      REVISION = 0
#      AGE = 0
# - If the interface is the same as the previous version, but bugs are
#   fixed, change:
#      REVISION += 1
libfwupd_lt_current = '2'
libfwupd_lt_revision = '0'
libfwupd_lt_age = '0'
libfwupd_lt_version = '@0@.@1@.@2@'.format(libfwupd_lt_current, libfwupd_lt_age, libfwupd_lt_revision)

# get supported warning flags
warning_flags = [
  '-Waggregate-return',
  '-Wunused',
  '-Warray-bounds',
  '-Wcast-align',
  '-Wclobbered',
  '-Wdeclaration-after-statement',
  '-Wdiscarded-qualifiers',
  '-Wduplicated-branches',
  '-Wduplicated-cond',
  '-Wempty-body',
  '-Wformat=2',
  '-Wformat-nonliteral',
  '-Wformat-security',
  '-Wformat-signedness',
  '-Wignored-qualifiers',
  '-Wimplicit-function-declaration',
  '-Wimplicit-int',
  '-Winit-self',
  '-Wint-conversion',
  '-Wlogical-op',
  '-Wmaybe-uninitialized',
  '-Wmissing-declarations',
  '-Wmissing-format-attribute',
  '-Wmissing-include-dirs',
  '-Wmissing-noreturn',
  '-Wmissing-parameter-type',
  '-Wmissing-prototypes',
  '-Wnested-externs',
  '-Wno-cast-function-type',
  '-Wno-address-of-packed-member',  # incompatible with g_autoptr()
  '-Wno-unknown-pragmas',
  '-Wno-missing-field-initializers',
  '-Wno-strict-aliasing',
  '-Wno-suggest-attribute=format',
  '-Wno-typedef-redefinition',
  '-Wno-unknown-warning-option',
  '-Wno-unused-parameter',
  '-Wold-style-definition',
  '-Woverride-init',
  '-Wpointer-arith',
  '-Wredundant-decls',
  '-Wreturn-type',
  '-Wshadow',
  '-Wsign-compare',
  '-Wstrict-aliasing',
  '-Wstrict-prototypes',
  '-Wswitch-default',
  '-Wtype-limits',
  '-Wundef',
  '-Wuninitialized',
  '-Wunused-but-set-variable',
  '-Wunused-variable',
  '-Wvla',
  '-Wwrite-strings'
]
if get_option('static_analysis') and host_machine.system() != 'windows'
  warning_flags += ['-fanalyzer', '-Wno-analyzer-null-dereference']
endif
cc = meson.get_compiler('c')
add_project_arguments(cc.get_supported_arguments(warning_flags), language: 'c')

if not meson.is_cross_build()
  add_project_arguments('-fstack-protector-strong', language: 'c')
endif

# enable full RELRO where possible
# FIXME: until https://github.com/mesonbuild/meson/issues/1140 is fixed
global_link_args = []
test_link_args = [
  '-Wl,-z,relro',
  '-Wl,-z,defs',
  '-Wl,-z,now',
  '-Wl,-z,ibt,-z,shstk',
]
foreach arg: test_link_args
  if cc.has_link_argument(arg)
    global_link_args += arg
  endif
endforeach
add_project_link_arguments(
  global_link_args,
  language: 'c'
)

add_project_arguments('-DFWUPD_COMPILATION', language: 'c')

# Needed for realpath(), syscall(), cfmakeraw(), etc.
add_project_arguments('-D_DEFAULT_SOURCE', language: 'c')

# needed for symlink() and BYTE_ORDER
add_project_arguments('-D_BSD_SOURCE', language: 'c')
add_project_arguments('-D__BSD_VISIBLE', language: 'c')
add_project_arguments('-D_XOPEN_SOURCE=700', language: 'c')

# needed for memfd_create()
add_project_arguments('-D_GNU_SOURCE', language: 'c')

# needed for memmem()
add_project_arguments('-D_DARWIN_C_SOURCE=900000', language: 'c')

# sanity check
if get_option('build') == 'all'
  build_standalone = true
  build_daemon = true
elif get_option('build') == 'standalone'
  build_standalone = true
  build_daemon = false
elif get_option('build') == 'library'
  build_standalone = false
  build_daemon = false
endif

prefix = get_option('prefix')

bindir = join_paths(prefix, get_option('bindir'))
libdir = join_paths(prefix, get_option('libdir'))
libexecdir = join_paths(prefix, get_option('libexecdir'))
#this ends up in compiled code, ignore prefix
if host_machine.system() == 'windows'
  sysconfdir = get_option('sysconfdir')
  localstatedir = get_option('localstatedir')
  datadir = get_option('datadir')
  installed_test_bindir = get_option('libexecdir')
  installed_test_datadir = get_option('datadir')
  daemon_dir = get_option('libexecdir')
else
  datadir = join_paths(prefix, get_option('datadir'))
  sysconfdir = join_paths(prefix, get_option('sysconfdir'))
  localstatedir = join_paths(prefix, get_option('localstatedir'))
  installed_test_bindir = join_paths(libexecdir, 'installed-tests', meson.project_name())
  installed_test_datadir = join_paths(datadir, 'installed-tests', meson.project_name())
  daemon_dir = join_paths(libexecdir, 'fwupd')
endif
mandir = join_paths(prefix, get_option('mandir'))
localedir = join_paths(prefix, get_option('localedir'))

diffcmd = find_program('diff')
gio = dependency('gio-2.0', version: '>= 2.68.0')
giounix = dependency('gio-unix-2.0', version: '>= 2.68.0', required: false)
if giounix.found()
  conf.set('HAVE_GIO_UNIX', '1')
endif
gmodule = dependency('gmodule-2.0')
if build_standalone
gudev = dependency('gudev-1.0', version: '>= 232',
                    required: get_option('gudev').disable_auto_if(host_machine.system() != 'linux'))
if gudev.found()
  conf.set('HAVE_GUDEV', '1')
endif
bluez = get_option('bluez').disable_auto_if(host_machine.system() != 'linux')
if bluez.allowed()
  conf.set('HAVE_BLUEZ', '1')
endif
host_cpu = host_machine.cpu_family()
hsi = get_option('hsi').disable_auto_if(host_machine.system() != 'linux').disable_auto_if(host_cpu != 'x86' and host_cpu != 'x86_64').allowed()
if hsi
  conf.set('HAVE_HSI', '1')
endif
libxmlb = dependency('xmlb', version: '>= 0.3.6', fallback: ['libxmlb', 'libxmlb_dep'])
gusb = dependency('gusb', version: '>= 0.3.8', fallback: ['gusb', 'gusb_dep'], required: get_option('gusb'))
if gusb.found()
  conf.set('HAVE_GUSB', '1')
endif
sqlite = dependency('sqlite3', required: get_option('sqlite'))
if sqlite.found()
  conf.set('HAVE_SQLITE', '1')
endif
passim = dependency('passim', required: get_option('passim'), fallback: ['passim', 'passim_dep'])
if passim.found()
  conf.set('HAVE_PASSIM', '1')
endif
libarchive = dependency('libarchive', required: get_option('libarchive'))
if libarchive.found()
  conf.set('HAVE_LIBARCHIVE', '1')
  if cc.has_header_symbol('archive.h', 'archive_write_add_filter_zstd')
    conf.set('HAVE_LIBARCHIVE_WRITE_ADD_FILTER_ZSTD', '1')
  endif
endif
endif
libjcat = dependency('jcat', version: '>= 0.1.4', fallback: ['libjcat', 'libjcat_dep'])
libjsonglib = dependency('json-glib-1.0', version: '>= 1.6.0')
valgrind = dependency('valgrind', required: false)
libcurl = dependency('libcurl', version: '>= 7.62.0', required: get_option('curl'))
if libcurl.found()
  conf.set('HAVE_LIBCURL', '1')
endif
polkit = dependency('polkit-gobject-1', version: '>= 0.103',
         required: get_option('polkit').disable_auto_if(host_machine.system() != 'linux'))
if polkit.found()
  conf.set('HAVE_POLKIT', '1')
  if polkit.version().version_compare('>= 0.114')
    conf.set('HAVE_POLKIT_0_114', '1')
  endif
  conf.set_quoted ('POLKIT_ACTIONDIR', polkit.get_variable(pkgconfig: 'actiondir'))
endif
if build_daemon
  if not polkit.found()
    warning('Polkit is disabled, the daemon will allow ALL client actions')
  endif
  udevdir = get_option('udevdir')
  if udevdir == '' and host_machine.system() == 'linux'
    udev = dependency('udev')
    udevdir = udev.get_variable(pkgconfig: 'udevdir')
  endif
endif
libm = cc.find_library('m', required: false)
libgcab = dependency('libgcab-1.0', version: '>= 1.0', fallback: ['gcab', 'gcab_dep'])
if libgcab.type_name() == 'pkgconfig'
  if cc.has_function('gcab_file_set_bytes', dependencies: libgcab)
    conf.set('HAVE_GCAB_FILE_SET_BYTES', '1')
  endif
  if cc.has_function('gcab_cabinet_add_allowed_compression', dependencies: libgcab)
    conf.set('HAVE_GCAB_CABINET_ADD_ALLOWED_COMPRESSION', '1')
  endif
endif

bashcomp = dependency('bash-completion', required: false)
python3 = import('python').find_installation('python3')

gnutls = dependency('gnutls', version: '>= 3.6.0', required: get_option('gnutls'))
if gnutls.found()
  conf.set('HAVE_GNUTLS', '1')
endif

lzma = dependency('liblzma', required: get_option('lzma'))
if lzma.found()
  conf.set('HAVE_LZMA', '1')
endif

cbor = dependency('libcbor', version: '>= 0.7.0', required: get_option('cbor'))
if cbor.found()
  conf.set('HAVE_CBOR', '1')
endif

platform_deps = []
if get_option('default_library') != 'static'
  if host_machine.system() == 'windows'
    platform_deps += cc.find_library('shlwapi')
  endif
  if host_machine.system() == 'freebsd'
    platform_deps += dependency('efivar')
  endif
endif

if valgrind.found()
  conf.set('HAVE_VALGRIND', '1')
endif

libsystemd = dependency('libsystemd',
                        required: get_option('systemd').disable_auto_if(host_machine.system() != 'linux'))
offline = get_option('offline').require(libsystemd.found(),
    error_message: '-Doffline=enabled requires systemd')
if offline.allowed()
  conf.set('HAVE_FWUPDOFFLINE', '1')
endif

if cc.has_header('sys/utsname.h')
  conf.set('HAVE_UTSNAME_H', '1')
endif
if cc.has_header('sys/inotify.h')
  conf.set('HAVE_INOTIFY_H', '1')
endif
if cc.has_header('sys/ioctl.h')
  conf.set('HAVE_IOCTL_H', '1')
endif
if cc.has_header('errno.h')
  conf.set('HAVE_ERRNO_H', '1')
endif
if cc.has_header('sys/socket.h')
  conf.set('HAVE_SOCKET_H', '1')
endif
if cc.has_header('scsi/sg.h')
  conf.set('HAVE_SCSI_SG_H', '1')
endif
if cc.has_header('sys/select.h')
  conf.set('HAVE_SELECT_H', '1')
endif
if cc.has_header('sys/io.h') and cc.has_function('outb', prefix: '#include <sys/io.h>')
  conf.set('HAVE_IO_H', '1')
endif
if cc.has_header('linux/ethtool.h')
  conf.set('HAVE_ETHTOOL_H', '1')
endif
if cc.has_header('linux/mei.h')
  conf.set('HAVE_MEI_H', '1')
endif
if cc.has_header('mtd/mtd-user.h')
  conf.set('HAVE_MTD_USER_H', '1')
endif
if cc.has_header('linux/hidraw.h')
  conf.set('HAVE_HIDRAW_H', '1')
endif
if cc.has_header('sys/mman.h')
  conf.set('HAVE_MMAN_H', '1')
endif
if cc.has_header('poll.h')
  conf.set('HAVE_POLL_H', '1')
endif
if cc.has_header('kenv.h')
  conf.set('HAVE_KENV_H', '1')
endif
if cc.has_header('malloc.h')
  conf.set('HAVE_MALLOC_H', '1')
  if cc.has_function('malloc_trim', prefix: '#include <malloc.h>')
	 conf.set('HAVE_MALLOC_TRIM', '1')
  endif
endif
has_cpuid = cc.has_header_symbol('cpuid.h', '__get_cpuid_count', required: get_option('plugin_msr'))
if has_cpuid
  conf.set('HAVE_CPUID_H', '1')
endif
if cc.has_function('getuid')
  conf.set('HAVE_GETUID', '1')
endif
if cc.has_function('realpath')
  conf.set('HAVE_REALPATH', '1')
endif
if cc.has_function('memmem')
  conf.set('HAVE_MEMMEM', '1')
endif
if cc.has_function('sigaction')
  conf.set('HAVE_SIGACTION', '1')
endif
if cc.has_function('memfd_create')
  conf.set('HAVE_MEMFD_CREATE', '1')
endif
if cc.has_header_symbol('locale.h', 'LC_MESSAGES')
  conf.set('HAVE_LC_MESSAGES', '1')
endif
if cc.has_header('linux/ipmi.h')
  have_linux_ipmi = true
  conf.set('HAVE_LINUX_IPMI_H', '1')
else
  have_linux_ipmi = false
endif
if cc.has_header_symbol('fcntl.h', 'F_WRLCK')
  conf.set('HAVE_WRLCK', '1')
endif
if cc.has_header_symbol('fcntl.h', 'F_OFD_SETLK')
  conf.set('HAVE_OFD', '1')
endif
if cc.has_function('pwrite', args: '-D_XOPEN_SOURCE')
  conf.set('HAVE_PWRITE', '1')
endif
if cc.has_header_symbol('sys/mount.h', 'BLKSSZGET')
  conf.set('HAVE_BLKSSZGET', '1')
endif

if host_machine.system() == 'freebsd'
  if cc.has_type('struct efi_esrt_entry_v1', prefix: '#include <sys/types.h>\n#include <sys/efiio.h>')
    conf.set('HAVE_FREEBSD_ESRT', '1')
  endif
endif

# this is way less hassle than including TargetConditionals.h and looking for TARGET_OS_MAC=1
if host_machine.system() == 'darwin'
  conf.set('HOST_MACHINE_SYSTEM_DARWIN', '1')
endif

# EFI
if build_standalone
  efi_app_location = join_paths(libexecdir, 'fwupd', 'efi')
  conf.set_quoted('EFI_APP_LOCATION', efi_app_location)
  if host_cpu == 'x86'
    EFI_MACHINE_TYPE_NAME = 'ia32'
  elif host_cpu == 'x86_64'
    EFI_MACHINE_TYPE_NAME = 'x64'
  elif host_cpu == 'arm'
    EFI_MACHINE_TYPE_NAME = 'arm'
  elif host_cpu == 'aarch64'
    EFI_MACHINE_TYPE_NAME = 'aa64'
  else
    EFI_MACHINE_TYPE_NAME = ''
  endif
  conf.set_quoted('EFI_MACHINE_TYPE_NAME', EFI_MACHINE_TYPE_NAME)
endif

umockdev = dependency('umockdev-1.0', required: false)

if build_standalone
  libflashrom = dependency('flashrom', 
                            fallback: ['flashrom', 'flashrom_dep'], 
                            required: get_option('plugin_flashrom').disable_auto_if(host_machine.system() != 'linux'))
endif

if libsystemd.found()
  systemd = dependency('systemd', version: '>= 211', required: get_option('systemd'))
  conf.set('HAVE_SYSTEMD' , '1')
  conf.set('HAVE_LOGIND' , '1')
  systemd_root_prefix = get_option('systemd_root_prefix')
  if systemd_root_prefix == ''
    systemdunitdir = systemd.get_variable(pkgconfig: 'systemdsystemunitdir')
    systemd_shutdown_dir = systemd.get_variable(pkgconfig: 'systemdshutdowndir')
    systemd_modules_load_dir = systemd.get_variable(pkgconfig: 'modulesloaddir')
  else
    systemdunitdir = systemd.get_variable(pkgconfig: 'systemdsystemunitdir', pkgconfig_define: ['rootprefix', systemd_root_prefix])
    systemd_root_prefix_varname = 'root_prefix'
    if systemd.version().version_compare('< 246')
      systemd_root_prefix_varname = 'rootprefix'
    endif
    systemd_shutdown_dir = systemd.get_variable(pkgconfig: 'systemdshutdowndir', pkgconfig_define: [systemd_root_prefix_varname, systemd_root_prefix])
    systemd_modules_load_dir = systemd.get_variable(pkgconfig: 'modulesloaddir', pkgconfig_define: [systemd_root_prefix_varname, systemd_root_prefix])
  endif
endif

elogind = dependency('systemd', 'libelogind', required: get_option('elogind'))
if elogind.found()
  conf.set('HAVE_LOGIND' , '1')
endif

if get_option('consolekit').disable_auto_if(host_machine.system() != 'linux').allowed()
  conf.set('HAVE_CONSOLEKIT' , '1')
endif

if get_option('supported_build').disable_auto_if(not tag).allowed()
    conf.set('SUPPORTED_BUILD', '1')
endif

# compatibility with behavior in < kernel 6.3
if get_option('thinklmi_compat')
  conf.set('FU_THINKLMI_COMPAT', '1')
endif

gnome = import('gnome')
i18n = import('i18n')

conf.set_quoted('FWUPD_PREFIX', prefix)
conf.set_quoted('FWUPD_BINDIR', bindir)
conf.set_quoted('FWUPD_LIBDIR', libdir)
conf.set_quoted('FWUPD_LIBEXECDIR', libexecdir)
conf.set_quoted('FWUPD_DATADIR', datadir)
conf.set_quoted('FWUPD_LOCALSTATEDIR', localstatedir)
conf.set_quoted('FWUPD_SYSCONFDIR', sysconfdir)
conf.set_quoted('FWUPD_LOCALEDIR', localedir)

if build_standalone
if host_machine.system() == 'windows'
  libdir_pkg = 'fwupd-@0@'.format(fwupd_version)
else
  libdir_pkg = join_paths(libdir, 'fwupd-@0@'.format(fwupd_version))
endif
conf.set_quoted('FWUPD_LIBDIR_PKG', libdir_pkg)
endif

# sanity check, otherwise there is not point building
if build_standalone and host_machine.system() == 'windows' and not gusb.found()
  error('gusb feature is required for Windows build')
endif

conf.set_quoted('GETTEXT_PACKAGE', meson.project_name())
conf.set_quoted('PACKAGE_NAME', meson.project_name())
conf.set_quoted('VERSION', meson.project_version())

motd_file = '85-fwupd'
motd_dir = 'motd.d'
conf.set_quoted('MOTD_FILE', motd_file)
conf.set_quoted('MOTD_DIR', motd_dir)

configure_file(
  output: 'config.h',
  configuration: conf
)

protobufc = dependency('libprotobuf-c', required: get_option('plugin_logitech_bulkcontroller'))
protoc = find_program('protoc', 'protoc-c', required: get_option('plugin_logitech_bulkcontroller'))

root_incdir = include_directories('.')

fwupd_gir = []
gir_dep = dependency('gobject-introspection-1.0', required: get_option('introspection'))
introspection = get_option('introspection').disable_auto_if(host_machine.system() != 'linux').disable_auto_if(not gir_dep.found())

gidocgen_dep = dependency('gi-docgen',
  version: '>= 2021.1',
  native: true,
  fallback: ['gi-docgen', 'dummy_dep'],
  required: get_option('docs'),
)
gidocgen_app = find_program('gi-docgen', required: gidocgen_dep.found())
build_docs = gidocgen_dep.found() and gidocgen_app.found()

if build_docs and gidocgen_dep.version().version_compare('< 2022.2')
  markdown_version = run_command(
    [python3, '-c', 'import markdown; print(markdown.__version__)'],
    check: true,
  ).stdout().strip()
  build_docs = get_option('docs').require(
    markdown_version.version_compare('>=3.2'),
    error_message: 'docs=enabled requires at least markdown >= 3.2'
  ).allowed()
endif

jinja2 = run_command(
  [python3, '-c', 'import jinja2; print(jinja2.__version__)'],
  check: true,
)
if jinja2.stderr().strip() != ''
  error('Python module jinja2 not found')
endif

# using "meson configure -Db_sanitize=address,undefined" is super useful in finding corruption,
# but it does not work with our GMainContext-abuse tests...
if get_option('b_sanitize') in ['address,undefined', 'address', 'undefined', 'leak']
  run_sanitize_unsafe_tests = false
else
  run_sanitize_unsafe_tests = true
endif

# take foo.rs and generate foo-struct.c and foo-struct.h files like protobuf_c
rustgen = generator(python3,
  output  : ['@BASENAME@-struct.c', '@BASENAME@-struct.h'],
  arguments : [
    join_paths(meson.project_source_root(), 'libfwupdplugin', 'rustgen.py'),
    '@INPUT@',
    '@OUTPUT0@',
    '@OUTPUT1@',
  ],
)

subdir('generate-build')
subdir('libfwupd')
if polkit.found()
  subdir('policy')
endif
if build_standalone
  plugin_quirks = []
  gcab = find_program('gcab', required: get_option('tests'))
  subdir('data')
  subdir('po')
  subdir('libfwupdplugin')
  subdir('contrib')

  # common to all plugins
  plugin_builtins = []
  plugin_incdirs = [
    root_incdir,
    fwupd_incdir,
    fwupdplugin_incdir,
  ]
  plugin_libs = [
    fwupd,
    fwupdplugin,
  ]
  subdir('plugins')
  subdir('src')
  subdir('docs')

  # append all the quirks into one big file and gzip it
  custom_target('builtin-quirk-gz',
    input: plugin_quirks,
    output: 'builtin.quirk.gz',
    command: [
      generate_quirk_builtin,
      '@OUTPUT@',
      '@INPUT@',
    ],
    install: true,
    install_dir: join_paths(datadir, 'fwupd', 'quirks.d'),
  )
endif

if build_daemon and libsystemd.found() and offline.allowed()
  install_symlink('fwupd-offline-update.service',
    install_dir: join_paths(systemdunitdir, 'system-update.target.wants'),
    pointing_to: join_paths('..', 'fwupd-offline-update.service')
  )
endif
