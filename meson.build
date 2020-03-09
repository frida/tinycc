tcc_c_args = [
  '-DONE_SOURCE=0',
]

tcc_c_args += cc.get_supported_arguments([
  '-Wno-format-truncation',
  '-Wno-pointer-sign',
  '-Wno-string-plus-int',
  '-Wno-unused-result',
])

tcc_sources = [
  'libtcc.c',
  'tccpp.c',
  'tccgen.c',
  'tccelf.c',
  'tccasm.c',
  'tccrun.c',
]

if host_os_family == 'windows'
  tcc_c_args += [
    '-DTCC_TARGET_PE',
  ]
  tcc_sources += [
    'tccpe.c',
  ]
endif

if host_arch == 'x86'
  tcc_c_args += [
    '-DTCC_TARGET_I386',
  ]
  tcc_sources += [
    'i386-gen.c',
    'i386-link.c',
    'i386-asm.c',
  ]
endif

if host_arch == 'x86_64'
  tcc_c_args += [
    '-DTCC_TARGET_X86_64',
  ]
  tcc_sources += [
    'x86_64-gen.c',
    'x86_64-link.c',
    'i386-asm.c',
  ]
  if host_os_family != 'windows'
    tcc_sources += [
      'lib/va_list.c',
    ]
  endif
endif

if host_arch == 'arm'
  tcc_c_args += [
    '-DTCC_TARGET_ARM',
    '-DTCC_ARM_EABI',
    '-DTCC_ARM_HARDFLOAT',
    '-DTCC_ARM_VFP',
  ]
  tcc_sources += [
    'arm-gen.c',
    'arm-link.c',
    'arm-asm.c',
  ]
endif

if host_arch == 'arm64'
  tcc_c_args += [
    '-DTCC_TARGET_ARM64',
  ]
  tcc_sources += [
    'arm64-gen.c',
    'arm64-link.c',
  ]
endif

ptrauth = '''#ifdef __clang__
# if __has_feature(ptrauth_calls)
#  define HAVE_PTRAUTH 1
# endif
#endif

#ifndef HAVE_PTRAUTH
# error Pointer authentication not supported
#endif
'''
if cc.compiles(ptrauth, name : 'pointer authentication')
  tcc_c_args += '-DHAVE_PTRAUTH'
endif

tinycc = library('tinycc', tcc_sources,
  c_args: frida_component_cflags + tcc_c_args,
  include_directories: include_directories('./pregenerated'),
)

tinycc_dep = declare_dependency(
  include_directories: include_directories('.'),
)
