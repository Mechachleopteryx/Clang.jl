## Clang

[![Build Status](https://travis-ci.org/JuliaInterop/Clang.jl.svg?branch=master)](https://travis-ci.org/JuliaInterop/Clang.jl)
[![Build Status](https://github.com/JuliaInterop/Clang.jl/workflows/CI/badge.svg)](https://github.com/JuliaInterop/Clang.jl/actions)
![TagBot](https://github.com/JuliaInterop/Clang.jl/workflows/TagBot/badge.svg)
[![codecov](https://codecov.io/gh/JuliaInterop/Clang.jl/branch/master/graph/badge.svg)](https://codecov.io/gh/JuliaInterop/Clang.jl)
[![](https://img.shields.io/badge/docs-stable-blue.svg)](https://JuliaInterop.github.io/Clang.jl/stable)
[![](https://img.shields.io/badge/docs-dev-blue.svg)](https://JuliaInterop.github.io/Clang.jl/dev)

This package provides a Julia language wrapper for libclang: the stable, C-exported
interface to the LLVM Clang compiler. The [libclang API documentation](http://clang.llvm.org/doxygen/group__CINDEX.html)
provides background on the functionality available through libclang, and thus
through the Julia wrapper. The repository also hosts related tools built
on top of libclang functionality.

## Installation
This package is under an overhaul. It's highly recommended to use the master branch and the new generator API. 

If you'd like to use the old version, please checkout [this branch](https://github.com/JuliaInterop/Clang.jl/tree/old-generator) for the documentation. Should you have any questions on how to upgrade the generator script, feel free to submit a post/request in the [Discussions](https://github.com/JuliaInterop/Clang.jl/discussions) area.

```
pkg> dev Clang
```

## C-bindings generator

Clang.jl provides a module `Clang.Generators` for generating 

### Quick start

Write a config file `generator.toml`:
```
[general]
library_name = "libclang"
output_file_path = "./LibClang.jl"
module_name = "LibClang"
jll_pkg_name = "Clang_jll"
export_symbol_prefixes = ["CX", "clang_"]
```

and a Julia script `generator.jl`:
```julia
using Clang.Generators
using Clang.LibClang.Clang_jll

cd(@__DIR__)

include_dir = joinpath(Clang_jll.artifact_dir, "include") |> normpath
clang_dir = joinpath(include_dir, "clang-c")

options = load_options(joinpath(@__DIR__, "generator.toml"))

# add compiler flags, e.g. "-DXXXXXXXXX"
args = get_default_args()
push!(args, "-I$include_dir")

headers = [joinpath(clang_dir, header) for header in readdir(clang_dir) if endswith(header, ".h")]
# there is also an experimental `detect_headers` function for auto-detecting top-level headers in the directory
# headers = detect_headers(clang_dir, args)

# create context
ctx = create_context(headers, args, options)

# run generator
build!(ctx)
```

Please refer to [this toml file](https://github.com/JuliaInterop/Clang.jl/blob/master/gen/generator.toml) for a full list of configuration options.

### Examples

The generator is currently used by several projects and you can take them as examples.

- [JuliaInterop/Clang.jl](https://github.com/JuliaInterop/Clang.jl/tree/master/gen): LibClang.jl is generated by this package itself 
- [JuliaWeb/LibCURL.jl](https://github.com/JuliaWeb/LibCURL.jl/tree/e2e0b6fb63368ee623a24ff231ac96bdc302deb5/gen): cross-platform support
- [scipopt/SCIP.jl](https://github.com/scipopt/SCIP.jl/tree/6d86f2c65d287ef2ad50ae15804b845eef1263e2/gen): applying custom patches
- [JuliaGPU/VulkanCore.jl](https://github.com/JuliaGPU/VulkanCore.jl)
- [Gnimuc/CImGui.jl](https://github.com/Gnimuc/CImGui.jl/tree/master/gen): header-only library
- [JuliaGPU/CUDA.jl](https://github.com/JuliaGPU/CUDA.jl/tree/master/res/wrap): old generator
- [SciML/Sundials.jl](https://github.com/SciML/Sundials.jl/blob/master/scripts/wrap_sundials.jl): old generator
