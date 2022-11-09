
```@meta
EditURL = "https://github.com/ioxera/Pkg.jl/blob/master/docs/src/basedocs.md"
```

# Pkg

Pkg 是Julia内建的包管理器，用于处理安装，更新，移除包等操作。

!!! note
    下面是Pkg的简短介绍。更详细的信息位于 `Project.toml` 和 `Manifest.toml` 文件，
    包括包版本兼容性(`[compat]`)，环境，注册表等信息。高度建议阅读完整手册，可以访问：
    [https://pkgdocs.julialang.org](https://pkgdocs.julialang.org).

```@eval
import Markdown
file = joinpath(Sys.STDLIB, "Pkg", "docs", "src", "getting-started.md")
str = read(file, String)
str = replace(str, r"^#.*$"m => "")
str = replace(str, "[API Reference](@ref)" => "[API Reference](https://pkgdocs.julialang.org/v1/api/)")
str = replace(str, "(@ref Working-with-Environments)" => "(https://pkgdocs.julialang.org/v1/environments/)")
str = replace(str, "(@ref Managing-Packages)" => "(https://pkgdocs.julialang.org/v1/managing-packages/)")
Markdown.parse(str)
```
