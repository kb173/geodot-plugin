#!python
import os, subprocess
from shutil import copyfile

opts = Variables([], ARGUMENTS)

# Gets the standard flags CC, CCX, etc.
env = DefaultEnvironment()

# Local dependency paths, adapt them to your setup
godot_headers_path = "godot-cpp/godot_headers/"
cpp_bindings_path = "godot-cpp/"
cpp_library = "libgodot-cpp"

rte_cpp_path = "src/raster-tile-extractor/"
rte_libpath = "src/raster-tile-extractor/build/"
rte_library = "libRasterTileExtractor"

vector_cpp_path = "src/vector-extractor/"
vector_libpath = "src/vector-extractor/build/"
vector_library = "libVectorExtractor"

demo_path = "demo/addons/geodot/"

lib_file_ending = ""

# Define our options
opts.Add(EnumVariable('target', "Compilation target", 'debug', ['d', 'debug', 'r', 'release']))
opts.Add(EnumVariable('platform', "Compilation platform", '', ['', 'windows', 'x11', 'linux', 'osx']))
opts.Add(EnumVariable('p', "Compilation target, alias for 'platform'", '', ['', 'windows', 'x11', 'linux', 'osx']))
opts.Add(BoolVariable('use_llvm', "Use the LLVM / Clang compiler", 'no'))
opts.Add(PathVariable('target_path', 'The path where the lib is installed.', demo_path))
opts.Add(PathVariable('target_name', 'The library name.', 'libgeodot', PathVariable.PathAccept))

# only support 64 at this time..
bits = 64

# Updates the environment with the option variables.
opts.Update(env)

# Process some arguments
if env['use_llvm']:
    env['CC'] = 'clang'
    env['CXX'] = 'clang++'

if env['p'] != '':
    env['platform'] = env['p']

if env['platform'] == '':
    print("No valid target platform selected.")
    quit();


# Build the extractor libraries
subprocess.call("cd " + rte_cpp_path + " && scons platform=" + env['platform'], shell=True)
subprocess.call("cd " + vector_cpp_path + " && scons platform=" + env['platform'], shell=True)

# For the reference:
# - CCFLAGS are compilation flags shared between C and C++
# - CFLAGS are for C-specific compilation flags
# - CXXFLAGS are for C++-specific compilation flags
# - CPPFLAGS are for pre-processor flags
# - CPPDEFINES are for pre-processor defines
# - LINKFLAGS are for linking flags

# Check our platform specifics
if env['platform'] == "osx":
    lib_file_ending = ".dylib"
    env['target_path'] += 'osx/'
    cpp_library += '.osx'
    if env['target'] in ('debug', 'd'):
        env.Append(CCFLAGS=['-g', '-O2', '-arch', 'x86_64'])
        env.Append(LINKFLAGS=['-arch', 'x86_64'])
    else:
        env.Append(CCFLAGS=['-g', '-O3', '-arch', 'x86_64'])
        env.Append(LINKFLAGS=['-arch', 'x86_64'])

elif env['platform'] in ('x11', 'linux'):
    lib_file_ending = ".so"
    env['target_path'] += 'x11/'
    cpp_library += '.linux'

    env.Append(LINKFLAGS=[
        '-Wl,-rpath,\'$$ORIGIN\''
    ])

    env.Append(CXXFLAGS=['-std=c++17'])

    if env['target'] in ('debug', 'd'):
        env.Append(CCFLAGS=['-fPIC', '-g3', '-Og'])
    else:
        env.Append(CCFLAGS=['-fPIC', '-g', '-O3'])

elif env['platform'] == "windows":
    lib_file_ending = ".dll"
    env['target_path'] += 'win64/'
    cpp_library += '.windows'
    # This makes sure to keep the session environment variables on windows,
    # that way you can run scons in a vs 2017 prompt and it will find all the required tools
    env.Append(ENV=os.environ)

    env.Append(CXXFLAGS=['-std:c++17'])

    env.Append(CCFLAGS = ['-DWIN32', '-D_WIN32', '-D_WINDOWS', '-W3', '-GR', '-D_CRT_SECURE_NO_WARNINGS'])
    if env['target'] in ('debug', 'd'):
        env.Append(CCFLAGS = ['-EHsc', '-D_DEBUG', '-MDd'])
    else:
        env.Append(CCFLAGS = ['-O2', '-EHsc', '-DNDEBUG', '-MD'])

if env['target'] in ('debug', 'd'):
    cpp_library += '.debug'
else:
    cpp_library += '.release'

cpp_library += '.' + str(bits)

# make sure our binding library is properly includes
env.Append(CPPPATH=['.', godot_headers_path, cpp_bindings_path + 'include/', cpp_bindings_path + 'include/core/', cpp_bindings_path + 'include/gen/', rte_cpp_path, vector_cpp_path])
env.Append(LIBPATH=[cpp_bindings_path + 'bin/', rte_libpath, vector_libpath])
env.Append(LIBS=[cpp_library, rte_library, vector_library])

# tweak this if you want to use different folders, or more folders, to store your source code in.
env.Append(CPPPATH=['src/'])
env.Append(CPPPATH=['src/global/'])
sources = Glob('src/*.cpp')

library = env.SharedLibrary(target=env['target_path'] + env['target_name'] , source=sources)

Default(library)

# Generates help for the -h scons option.
Help(opts.GenerateHelpText(env))

# Copy the libRasterTileExtractor dependency
# First, make sure the target path exists
if not os.path.exists(env['target_path']):
    os.makedirs(env['target_path'])
copyfile(os.path.join(rte_libpath, rte_library) + lib_file_ending, os.path.join(env['target_path'], rte_library) + lib_file_ending)
copyfile(os.path.join(vector_libpath, vector_library) + lib_file_ending, os.path.join(env['target_path'], vector_library) + lib_file_ending)
