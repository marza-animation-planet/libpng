import os
import re
import sys
import excons
import excons.tools.zlib


env = excons.MakeBaseEnv()

out_incdir = excons.OutputBaseDirectory() + "/include"
out_libdir = excons.OutputBaseDirectory() + "/lib"
png_deps   = []

# ZLIB Setup ===================================================================

# Allow custom zlib setup using flags:
#   with-zlib, with-zlib-inc, with-zlib-lib, zlib-libname, zlib-libsuffix, zlib-static
zlib_path = None
zlib_incdir, zlib_libdir = excons.GetDirs("zlib")

if zlib_incdir and zlib_libdir:
   zlib_libname = excons.GetArgument("zlib-libname", None)
   zlib_static = (excons.GetArgument("zlib-static", 0, int) != 0)

   if zlib_libname is None:
      zlib_libname = ("z" if sys.platform != "win32" else ("zlib" if zlib_static else "zdll"))
      zlib_libname += excons.GetArgument("zlib-libsuffix", "")

   if sys.platform == "win32":
      basename = zlib_libname + ".lib"
   else:
      basename = "lib" + zlib_libname + (".a" if zlib_static else ".so")

   zlib_path = "%s/%s" % (zlib_libdir, basename)
   if not os.path.isfile(zlib_path):
      zlib_path = None
      if sys.platform != "win32" and excons.arch_dir == "x64":
         zlib_path = "%s64/%s" % (zlib_libdir, basename)
         if not os.path.isfile(zlib_path):
            zlib_path = None

   # Setup defines in environment to be picked up by configure
   if zlib_path and not zlib_static:
      cflags = os.environ.get("CFLAGS", "") + " -DZLIB_DLL"
      os.environ["CFLAGS"] = cflags

   def RequireZlib(env):
      env.Append(CPPPATH=[zlib_incdir])
      env.Append(LIBPATH=[zlib_libdir])
      if not zlib_static:
         env.Append(CPPDEFINES=["ZLIB_DLL"])
         env.Append(LIBS=[zlib_libname])
      else:
         if not excons.StaticallyLink(env, zlib_libname, silent=True):
            env.Append(LIBS=[zlib_libname])

if zlib_path is None:
   excons.PrintOnce("Build zlib from sources ...")
   excons.Call("zlib", imp=["RequireZlib", "ZlibName"])
   zlib_incdir, zlib_libdir, zlib_path = out_incdir, out_libdir, ZlibName(static=True)
   png_deps = ["zlib"]

# PNG library ==================================================================

prjs = [
   {  "name": "libpng",
      "type": "cmake",
      "cmake-opts": {"PNG_TESTS": 0,
                     "PNG_DEBUG": excons.GetArgument("debug", 0, int),
                     "CMAKE_INSTALL_LIBDIR": "lib",
                     "ZLIB_INCLUDE_DIR": zlib_incdir,
                     "ZLIB_LIBRARY": zlib_path},
      "cmake-srcs": excons.CollectFiles([".", "arm", "intel", "mips", "powerpc"],
                                        patterns=["*.c", "*.S"], recursive=False) +
                    excons.CollectFiles(".", patterns=["CMakeLists.txt"],
                                        recursive=True, exclude=["zlib"]),
      "deps": png_deps
   }
]

excons.AddHelpOptions(libpng="EXTERNAL "+excons.tools.zlib.GetOptionsString())
excons.DeclareTargets(env, prjs)

# ==============================================================================

def RequireLibpng(env, static=False):
   env.Append(CPPPATH=[out_incdir])
   env.Append(LIBPATH=[out_libdir])
   if not static:
      env.Append(CPPDEFINES=["PNG_USE_DLL"])
      env.Append(LIBS=["libpng16" if sys.platform == "win32" else "png16"])
   else:
      if sys.platform == "win32":
         env.Append(LIBS=["libpng16_static"])
      else:
         if not excons.StaticallyLink(env, "png16", silent=True):
            env.Append(LIBS=["png16"])
      RequireZlib(env, static=True)

def LibpngName(static=False):
   basename = "libpng16"
   if sys.platform == "win32":
      if static:
         basename += "_static"
      basename += ".lib"
   else:
      basename += (".a" if static else ".so")
   return out_libdir + "/" + basename

Export("RequireLibpng LibpngName")

