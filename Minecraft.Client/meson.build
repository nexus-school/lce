exclude_sources = [
  ' ! -path "*/Platform/*"',
  ' ! -name "SurvivalMode.cpp"',
  ' ! -name "CreativeMode.cpp"',
  ' ! -name "GameMode.cpp"',
  ' ! -name "DemoMode.cpp"',
]

client_sources = run_command(
  'sh',
  '-c', 'find "'
  + meson.current_source_dir()
  + '" \\( -name "*.cpp" -o -name "*.c" \\)'
  + ' '.join(exclude_sources),
  check: true,
).stdout().strip().split('\n')

platform_sources = run_command(
  'sh',
  '-c', 'find "'
  + meson.current_source_dir() / 'Platform'
  + '" -maxdepth 1 \\( -name "*.cpp" -o -name "*.c" \\)',
  check: true,
).stdout().strip().split('\n')

exclude_platform_common_sources = [
  ' ! -path "*/Network/Sony/*"',
  ' ! -path "*/XUI/*"',
  ' ! -name "SonyLeaderboardManager.cpp"',
  ' ! -name "UIScene_InGameSaveManagementMenu.cpp"',
]

platform_sources += run_command(
  'sh',
  '-c', 'find "'
  + meson.current_source_dir() / 'Platform/Common'
  + '" \\( -name "*.cpp" -o -name "*.c" \\)'
  + ' '.join(exclude_platform_common_sources),
  check: true,
).stdout().strip().split('\n')

if host_machine.system() == 'linux' or host_machine.system() == 'emscripten'
  platform_sources += run_command(
    'sh',
    '-c', 'find "'
    + meson.current_source_dir() / 'Platform/Linux'
    + '" \\( -name "*.cpp" -o -name "*.c" \\) ',
    check: true,
  ).stdout().strip().split('\n')
endif

client_dependencies = [
  render_dep,
  input_dep,
  profile_dep,
  storage_dep,
  assets_localisation_dep,
  world_dep,
  gl_dep,
  glu_dep,
  thread_dep,
  dl_dep,
  zlib_dep,
  miniaudio_dep,
  stb_dep,
]

if get_option('enable_vsync')
  global_cpp_defs += ['-DENABLE_VSYNC']
endif

if get_option('classic_panorama')
  global_cpp_defs += '-DCLASSIC_PANORAMA'
endif

if get_option('enable_frame_profiler')
  global_cpp_defs += ['-DENABLE_FRAME_PROFILER']
endif

if get_option('ui_backend') == 'shiggy'
  shiggy_dep = dependency(
    'shiggy',
    fallback: ['shiggy', 'shiggy_dep'],
  )

  global_cpp_defs += ['-D_ENABLEIGGY']
  client_dependencies += shiggy_dep
endif

if get_option('ui_backend') == 'java'
  global_cpp_defs += '-DENABLE_JAVA_GUIS'
endif

occlusion_mode = get_option('occlusion_culling')
if occlusion_mode == 'off'
  global_cpp_defs += ['-DOCCLUSION_MODE_NONE']
elif occlusion_mode == 'frustum'
  global_cpp_defs += ['-DOCCLUSION_MODE_FRUSTUM']
elif occlusion_mode == 'bfs'
  global_cpp_defs += ['-DOCCLUSION_MODE_BFS', '-DUSE_OCCLUSION_CULLING']
elif occlusion_mode == 'hardware'
  global_cpp_defs += ['-DOCCLUSION_MODE_HARDWARE', '-DUSE_OCCLUSION_CULLING']
endif


# ✅ FIX: define link args properly
emscripten_link_args = []

if host_machine.system() == 'emscripten'
  emscripten_link_args += [
    '--preload-file=' + meson.project_source_root() + '/Minecraft.Assets@/'
  ]
endif


client = executable(
  'Minecraft.Client',
  client_sources + platform_sources + localisation[1],

  name_suffix: host_machine.system() == 'emscripten' ? 'html' : '',

  include_directories: include_directories(
    'Platform',
    'Platform/Linux/Iggy/include'
  ),

  dependencies: client_dependencies,

  # ✅ FIXED: always applied correctly
  link_args: emscripten_link_args,

  cpp_args: global_cpp_args
  + global_cpp_defs
  + [
    '-DUNICODE',
    '-D_UNICODE',
    '-include', meson.current_source_dir() / 'Platform/stdafx.h',
  ],

  c_args: global_cpp_defs + ['-DUNICODE', '-D_UNICODE'],

  install: true,
  install_dir: '',
)


custom_target(
  'copy_assets_to_client',
  input: [client, media_archive],
  output: 'assets.stamp',
  command: [
    python,
    meson.project_source_root() / 'scripts/copy_assets_to_client.py',
    meson.project_source_root(),
    meson.project_build_root(),
    meson.current_build_dir(),
    '@INPUT1@',
    '@OUTPUT@',
  ],
  build_by_default: true,
)
