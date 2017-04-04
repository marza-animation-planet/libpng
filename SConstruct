import os
import re
import sys
import excons


env = excons.MakeBaseEnv()

out_incdir = excons.OutputBaseDirectory() + "/include"
out_libdir = excons.OutputBaseDirectory() + "/lib"

excons.Call("zlib", imp=["RequireZlib", "ZlibName"])

prjs = [
   {  "name": "libpng",
      "type": "cmake",
      "cmake-opts": {"PNG_TESTS": 0,
                     "PNG_DEBUG": excons.GetArgument("debug", 0, int),
                     "CMAKE_INSTALL_LIBDIR": "lib",
                     "ZLIB_INCLUDE_DIR": out_incdir,
                     "ZLIB_LIBRARY": ZlibName(static=True)},
      "cmake-srcs": excons.CollectFiles([".", "arm", "intel", "mips", "powerpc"], patterns=["*.c", "*.S"], recursive=False) +
                    excons.CollectFiles(".", patterns=["CMakeLists.txt"], recursive=True, exclude=["zlib"]),
      "deps": ["zlib"]
   }
]

excons.DeclareTargets(env, prjs)


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

