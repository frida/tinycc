project('tcc', 'c',
  version: '0.9.27-frida',
)

cc = meson.get_compiler('c')

sources = [
  'libtcc.c',
  'tccpp.c',
  'tccgen.c',
  'tccelf.c',
  'tccasm.c',
  'tccrun.c',
]

cdata = configuration_data()
cdata.set_quoted('TCC_VERSION', meson.project_version())

tccdir = get_option('prefix') / get_option('libdir') / 'tcc'
cdata.set_quoted('CONFIG_TCCDIR', tccdir)

host_os = host_machine.system()
if host_os == 'windows'
  sources += 'tccpe.c'
  cdata.set('TCC_TARGET_PE', true)
elif host_os == 'darwin'
  sources += 'tccmacho.c'
  cdata.set('TCC_TARGET_MACHO', true)
endif

host_arch = host_machine.cpu_family()
if host_arch == 'x86'
  sources += [
    'i386-gen.c',
    'i386-link.c',
    'i386-asm.c',
  ]

  cdata.set('TCC_TARGET_I386', true)
elif host_arch == 'x86_64'
  sources += [
    'x86_64-gen.c',
    'x86_64-link.c',
    'i386-asm.c',
  ]
  if host_os != 'windows'
    sources += 'lib/va_list.c'
  endif

  cdata.set('TCC_TARGET_X86_64', true)
elif host_arch == 'arm'
  sources += [
    'arm-gen.c',
    'arm-link.c',
    'arm-asm.c',
  ]

  cdata.set('TCC_TARGET_ARM', true)
  cdata.set('TCC_ARM_EABI', true)
  cdata.set('TCC_ARM_VFP', true)

  is_hardfloat_src = '''
    #ifndef __ARM_PCS_VFP
    # error Not hardfloat
    #endif
  '''
  if cc.compiles(is_hardfloat_src, name: 'hardfloat ABI')
    cdata.set('TCC_ARM_HARDFLOAT', true)
  endif
elif host_arch == 'aarch64'
  sources += [
    'arm64-gen.c',
    'arm64-link.c',
    'arm-asm.c',
  ]

  cdata.set('TCC_TARGET_ARM64', true)

  have_ptrauth_src = '''
    #ifdef __clang__
    # if __has_feature(ptrauth_calls)
    #  define HAVE_PTRAUTH 1
    # endif
    #endif

    #ifndef HAVE_PTRAUTH
    # error Pointer authentication not supported
    #endif
  '''
  if cc.compiles(have_ptrauth_src, name: 'pointer authentication')
    cdata.set('HAVE_PTRAUTH', true)
  endif
endif

install_headers('libtcc.h')

runtime_headers = [
  'tcclib.h',
  'include/float.h',
  'include/ptrauth.h',
  'include/stdalign.h',
  'include/stdarg.h',
  'include/stdbool.h',
  'include/stddef.h',
  'include/stdnoreturn.h',
  'include/varargs.h',
]
install_headers(runtime_headers, install_dir: tccdir / 'include')

configure_file(
  output: 'config.h',
  configuration: cdata,
)

threads_dep = dependency('threads')
dl_dep = cc.find_library('dl', required: false)

tcc = library('tcc', sources,
  c_args: ['-DONE_SOURCE=0'],
  install: true,
  dependencies: [threads_dep, dl_dep],
)
tcc_dep = declare_dependency(
  link_with: tcc,
  include_directories: include_directories('.'),
  variables: {'libdir': get_option('libdir')},
)

pkg = import('pkgconfig')
pkg.generate(tcc,
  name: 'libtcc',
  description: 'Tiny C Compiler',
  url: 'https://bellard.org/tcc/',
)

meson.override_dependency('libtcc', tcc_dep)
