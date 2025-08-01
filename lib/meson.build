libfuse_sources = ['fuse.c', 'fuse_i.h', 'fuse_loop.c', 'fuse_loop_mt.c',
                   'fuse_lowlevel.c', 'fuse_misc.h', 'fuse_opt.c',
                   'fuse_signals.c', 'buffer.c', 'cuse_lowlevel.c',
                   'helper.c', 'modules/subdir.c', 'mount_util.c',
                   'fuse_log.c', 'compat.c', 'util.c', 'util.h' ]

if host_machine.system().startswith('linux')
   libfuse_sources += [ 'mount.c' ]
else
   libfuse_sources += [ 'mount_bsd.c' ]
endif

deps = [ thread_dep ]
if private_cfg.get('HAVE_ICONV')
   libfuse_sources += [ 'modules/iconv.c' ]
   libiconv = cc.find_library('iconv', required: false)
   if libiconv.found()
      deps += [ libiconv ]
   endif
endif

if private_cfg.get('HAVE_URING', false)
  libfuse_sources += [ 'fuse_uring.c' ]
  deps += [ dependency('liburing') ]
  deps += [ dependency('numa') ]
endif



libdl = cc.find_library('dl', required: false)
if libdl.found()
   deps += [ libdl ]
endif

if host_machine.system().startswith('netbsd')
   deps += [ cc.find_library('perfuse'),
             cc.find_library('puffs') ]
else
   # Required for clock_gettime before glibc 2.17
   deps += cc.find_library('c')
endif

fusermount_path = join_paths(get_option('prefix'), get_option('bindir'))
libfuse = library('fuse3',
                  libfuse_sources,
                  version: base_version,
                  soversion: '4',
                  include_directories: include_dirs,
                  dependencies: deps,
                  install: true,
                  link_depends: 'fuse_versionscript',
                  c_args: [ '-DFUSE_USE_VERSION=317',
                            '-DFUSERMOUNT_DIR="@0@"'.format(fusermount_path) ],
                  link_args: ['-Wl,--version-script,' + meson.current_source_dir()
                              + '/fuse_versionscript' ])

pkg = import('pkgconfig')
pkg.generate(libraries: [ libfuse, '-lc' ],
             libraries_private: '-ldl',
             version: meson.project_version(),
             name: 'fuse3',
             description: 'Filesystem in Userspace',
             subdirs: 'fuse3')

libfuse_dep = declare_dependency(include_directories: include_dirs,
                                 link_with: libfuse, dependencies: deps)
