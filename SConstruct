import os
import re
import sys
import string
import excons
import excons.cmake


env = excons.MakeBaseEnv()

out_incdir = excons.OutputBaseDirectory() + "/include"
out_libdir = excons.OutputBaseDirectory() + "/lib"
cfg_deps   = []

cmake_opts = {"PNG_TESTS": 0,
              "PNG_DEBUG": excons.GetArgument("debug", 0, int),
              "CMAKE_INSTALL_LIBDIR": "lib"}

# ZLIB Setup ===================================================================

def ZlibLibname(static):
   return ("z" if sys.platform != "win32" else ("zlib" if static else "zdll"))

def ZlibDefines(static):
   return ([] if static else ["ZLIB_DLL"])

rv = excons.cmake.ExternalLibRequire(cmake_opts, name="zlib", libnameFunc=ZlibLibname, definesFunc=ZlibDefines)
if rv is None:
   excons.PrintOnce("Build zlib from sources ...")
   excons.Call("zlib", imp=["RequireZlib", "ZlibPath"])
   cfg_deps.append(excons.cmake.OutputsCachePath("zlib"))
   zlib_static = (excons.GetArgument("zlib-static", 1, int) != 0)
   cmake_opts["ZLIB_LIBRARY"] = ZlibPath(static=zlib_static)
   cmake_opts["ZLIB_INCLUDE_DIR"] = out_incdir
   def ZlibRequire(env):
      RequireZlib(env, static=zlib_static)
else:
   ZlibRequire = rv

# PNG library ==================================================================

prjs = [
   {  "name": "libpng",
      "type": "cmake",
      "cmake-opts": cmake_opts,
      "cmake-cfgs": excons.CollectFiles(".", patterns=["CMakeLists.txt"], recursive=True, exclude=["zlib"]) + cfg_deps,
      "cmake-srcs": excons.CollectFiles([".", "arm", "intel", "mips", "powerpc"], patterns=["*.c", "*.S"], recursive=False),
   }
]

excons.AddHelpOptions(libpng="""CMAKE PNG OPTIONS
  zlib-static=0|1 : When building zlib from sources, link static version of the library to libpng. [1]

%s""" % excons.ExternalLibHelp("zlib"))
excons.DeclareTargets(env, prjs)

# ==============================================================================

def LibpngName(static=False):
   libname = "png16"
   if sys.platform == "win32":
      libname = "lib" + libname
      if static:
         libname += "_static"
   return libname

def LibpngPath(static=False):
   name = LibpngName(static=static)
   if sys.platform == "win32":
      libname = name + ".lib"
   else:
      libname = "lib" + name + (".a" if static else excons.SharedLibraryLinkExt())
   return out_libdir + "/" + libname

def RequireLibpng(env, static=False):
   env.Append(CPPPATH=[out_incdir])
   env.Append(LIBPATH=[out_libdir])
   excons.Link(env, LibpngName(static), static=static, force=True, silent=True)
   if static:
      ZlibRequire(env)

Export("LibpngName LibpngPath RequireLibpng")

