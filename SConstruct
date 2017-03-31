import os
import re
import sys
import excons



env = excons.MakeBaseEnv()

out_incdir = excons.OutputBaseDirectory() + "/include"
out_libdir = excons.OutputBaseDirectory() + "/lib"

excons.Call("zlib")

zlibname = ("zlibstatic.lib" if sys.platform == "win32" else "libz.a")

env.CMakeConfigure("libpng", opts={"PNG_TESTS": 0,
                                   "PNG_DEBUG": excons.GetArgument("debug", 0, int),
                                   "ZLIB_INCLUDE_DIR": out_incdir,
                                   "ZLIB_LIBRARY": out_libdir + "/" + zlibname})

cmake_in = env.CMakeInputs(dirs=[".", "arm", "intel", "mips", "powerpc"], patterns=[re.compile(r"^.*\.(h|c|S)$")])
cmake_out = env.CMakeOutputs()

target = env.CMake(cmake_out, cmake_in)
env.Depends(target, "zlib")

env.CMakeClean()
env.Alias("libpng", target)

excons.SyncCache()


def RequireLibpng(env, static=False):
   if not static:
      pass
   env.Append(CPPPATH=[out_incdir])
   env.Append(LIBPATH=[out_libdir])
   if sys.platform != "win32":
      if static:
         excons.StaticallyLink(env, "png", silent=True)
         excons.StaticallyLink(env, "z", silent=True)
      else:
         env.Append(LIBS=["png", "z"])
   else:
      if static:
         env.Append(LIBS=["libpng16_static", "zlibstatic"])
      else:
         env.Append(LIBS=["libpng16", "zlib"])

Export("RequireLibpng")

