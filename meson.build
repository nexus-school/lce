project(
  '4jcraft',
  ['cpp', 'c'],
  version: '0.1.0',
  meson_version: '>= 1.7',
  default_options: [
    'cpp_std=c++23',
    'warning_level=0',
    'buildtype=debug',

    # 🔥 TURN OFF (debugging clarity)
    'unity=off',

    'b_pch=true',
  ],
)

pymod = import('python')
python = pymod.find_installation('python3', required: true)

cc = meson.get_compiler('cpp')

use_gles = get_option('renderer') == 'gles'

# =========================
# GLOBAL DEFINES
# =========================
global_cpp_defs = [
  '-DSPLIT_SAVES',
  '-D_EXTENDED_ACHIEVEMENTS',
  '-D_DEBUG_MENUS_ENABLED',
  '-D_DEBUG',
  '-DDEBUG',
]

if host_machine.system() != 'emscripten'
  global_cpp_defs += '-D_LARGE_WORLDS'
endif

if host_machine.system() == 'linux' or host_machine.system() == 'emscripten'
  global_cpp_defs += ['-Dlinux', '-D__linux', '-D__linux__']
endif

if use_gles
  global_cpp_defs += ['-DGLES']
endif

add_project_arguments(global_cpp_defs, language: ['cpp', 'c'])

add_project_arguments(
  [
    '-fpermissive',
    '-Wshift-count-overflow',
    '-pipe',
  ],
  language: 'cpp'
)

# =========================
# EMSCRIPTEN CONFIG (CLEAN)
# =========================
if host_machine.system() == 'emscripten'

  # 🔧 compile flags
  add_project_arguments(
    [
      '-D__EMSCRIPTEN__',
      '-DNO_ATOMIC',
      '-DMA_NO_ALSA',
      '-DMA_NO_WEBAUDIO',   # 🔥 avoids miniaudio JS issues
      '-DSDL_MAIN_HANDLED',

      # 🔥 REQUIRED for real SDL
      '-sUSE_SDL=2',
    ],
    language: ['c', 'cpp']
  )

  # 🔗 linker flags
  add_project_link_arguments(
    [
      '-sUSE_SDL=2',
      '-sUSE_PTHREADS=1',
      '-sPTHREAD_POOL_SIZE=4',
      '-sFULL_ES3=1',
      '-sALLOW_MEMORY_GROWTH=1',
      '-sERROR_ON_UNDEFINED_SYMBOLS=0',
      '-sASSERTIONS=0',
      '-sFORCE_FILESYSTEM=1',

      '--use-port=zlib',

      # 🔥 FIXED preload (split args)
      '--preload-file',
      meson.project_source_root() + '/Minecraft.Assets@/',
    ],
    language: ['c', 'cpp']
  )

  # deps handled by emscripten
  sdl2_dep = declare_dependency()
  zlib_dep = declare_dependency()
  thread_dep = declare_dependency()
  dl_dep = declare_dependency()

  # fix atomic linker issues
  meson.override_dependency('atomic', declare_dependency())

else
  # native build
  sdl2_dep = dependency('sdl2')
  zlib_dep = dependency('zlib')
  thread_dep = dependency('threads')
  dl_dep = cc.find_library('dl', required: true)
endif

# =========================
# GRAPHICS
# =========================
if use_gles
  if host_machine.system() != 'emscripten'
    gl_dep = dependency('glesv2', required: true)
  else
    gl_dep = declare_dependency()
  endif
  glu_dep = dependency('', required: false)
else
  gl_dep = dependency('gl', required: true)
  glu_dep = dependency('glu', required: true)
endif

# =========================
# SUBPROJECTS
# =========================
stb = subproject('stb').get_variable('stb_inc')
stb_dep = declare_dependency(include_directories: stb)

miniaudio_dep = dependency(
  'miniaudio',
  fallback: ['miniaudio', 'miniaudio_dep'],
)

# =========================
# MODULES
# =========================
subdir('4J.Render')
subdir('4J.Input')
subdir('4J.Profile')
subdir('4J.Storage')

subdir('Minecraft.Assets')
subdir('Minecraft.World')
subdir('Minecraft.Client')