tmpflags = ['-DGLIB_MAJOR_VERSION=2', '-DGLIB_MINOR_VERSION=36', '-DGLIB_MICRO_VERSION=0']
gthread_hack = '-Dg_thread_init(x)='

t = executable('atomic-test', 'atomic-test.c',
 include_directories : inc_dirs,
 link_with : libglib,
 c_args : tmpflags,
)
test('atomic-test', t)

t = executable('bit-test', 'bit-test.c',
 include_directories : inc_dirs,
 link_with : libglib,
 c_args : tmpflags,
)
test('bit-test', t)

t = executable('child-test', 'child-test.c',
 include_directories : inc_dirs,
 link_with : libglib,
 c_args : tmpflags,
)
# Disabled for now because it takes too long to run.
#test('child-test', t)

t = executable('completion-test', 'completion-test.c',
 include_directories : inc_dirs,
 link_with : libglib,
)
test('completion-test', t)

t = executable('dirname-test', 'dirname-test.c',
 include_directories : inc_dirs,
 link_with : libglib,
)
test('dirname-test', t)

t = executable('file-test', 'file-test.c',
 include_directories : inc_dirs,
 link_with : libglib,
)
test('file-test', t)

t = executable('env-test', 'env-test.c',
 include_directories : inc_dirs,
 link_with : libglib,
)
test('env-test', t)

t = executable('gio-test', 'gio-test.c',
 include_directories : inc_dirs,
 link_with : libglib,
)
test('gio-test', t)

t = executable('iochannel-test', 'iochannel-test.c',
 include_directories : inc_dirs,
 link_with : libglib,
)
test('iochannel-test', t)

t = executable('mainloop-test', 'mainloop-test.c',
 include_directories : inc_dirs,
 link_with : libglib,
 c_args : gthread_hack,
)
test('mainloop-test', t)

t = executable('mapping-test', 'mapping-test.c',
 include_directories : inc_dirs,
 link_with : libglib,
)
test('mapping-test', t)

#t = executable('module-test', 'module-test.c',
# include_directories : inc_dirs,
# link_with : libglib,
#)
#test('module-test', t)

t = executable('onceinit-test', 'onceinit.c',
 include_directories : inc_dirs,
 link_with : libglib,
 c_args : gthread_hack,
)
test('onceinit-test', t)

t = executable('asyncqueue-test', 'asyncqueue-test.c',
 include_directories : inc_dirs,
 link_with : libglib,
 c_args : gthread_hack,
)
# disabled because it takes too long to run
#test('asyncqueue-test', t)

t = executable('qsort-test', 'qsort-test.c',
 include_directories : inc_dirs,
 link_with : libglib,
)
test('qsort-test', t)

t = executable('relation-test', 'relation-test.c',
 include_directories : inc_dirs,
 link_with : libglib,
)
test('relation-test', t)

t = executable('slice-test', 'slice-test.c', 'memchunks.c',
 include_directories : inc_dirs,
 link_with : libglib,
 c_args : gthread_hack,
)
test('slice-test', t)

t = executable('slice-color', 'slice-color.c',
 include_directories : inc_dirs,
 link_with : libglib,
)
test('slice-color', t)

t = executable('slice-concurrent', 'slice-concurrent.c',
 include_directories : inc_dirs,
 link_with : libglib,
)
test('slice-concurrent', t)

t = executable('slice-threadinit', 'slice-threadinit.c',
 include_directories : inc_dirs,
 link_with : libglib,
 c_args : gthread_hack,
)
test('slice-threadinit', t)

t = executable('sources', 'sources.c',
 include_directories : inc_dirs,
 link_with : libglib,
)
test('sources', t)

t = executable('spawn-test', 'spawn-test.c',
 include_directories : inc_dirs,
 link_with : libglib,
)
test('spawn-test', t)

t = executable('thread-test', 'thread-test.c',
 include_directories : inc_dirs,
 link_with : libglib,
 c_args : gthread_hack,
)
# disabled because it takes too long to run
#test('thread-test', t)

t = executable('threadpool-test', 'threadpool-test.c',
 include_directories : inc_dirs,
 link_with : libglib,
 c_args : gthread_hack,
)
# disabled because it takes too long to run
#test('threadpool-test', t)

t = executable('type-test', 'type-test.c',
 include_directories : inc_dirs,
 link_with : libglib,
)
test('type-test', t)

t = executable('unicode-caseconv', 'unicode-caseconv.c',
 include_directories : inc_dirs,
 link_with : libglib,
)
test('unicode-caseconv', t)

t = executable('unicode-encoding', 'unicode-encoding.c',
 include_directories : inc_dirs,
 link_with : libglib,
)
test('unicode-encoding', t)

