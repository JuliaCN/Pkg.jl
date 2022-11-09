
# [**4.** 使用“环境”](@id Working-with-Environments)

下面讨论 Pkg 与环境的交互。有关环境在代码加载中扮演的角色的更多信息，包括可以从中加载代码的环境的“栈”，请参阅[Julia 手册中的此部分](https://docs.julialang.org/en/v1/manual/code-loading/#Environments-1)。

## 创建自己的环境

到目前为止，我们已经将包添加到默认环境中 `~/.julia/environments/v1.8`。然而，创建另外的独立项目很容易。这种方法的好处是允许您在代码旁边创建 `Project.toml` 及 `Manifest.toml` 文件，并添加进版本控制（例如 git）中。需要指出的是，当两个项目使用相同版本的同一个包时，这个包的内容只保存一份。为新项目创建一个目录，然后激活该目录以使其成为“活动项目”，操作如下：

```julia-repl
(@v1.8) pkg> activate MyProject
Activating new environment at `~/MyProject/Project.toml`

(MyProject) pkg> st
    Status `~/MyProject/Project.toml` (empty project)
```

请注意，激活新项目时，REPL 提示会发生变化。在添加包之前，此环境中没有文件，甚至可能不会创建环境目录：

```julia-repl
julia> isdir("MyProject")
false

(MyProject) pkg> add Example
   Resolving package versions...
   Installed Example ─ v0.5.3
    Updating `~/MyProject/Project.toml`
  [7876af07] + Example v0.5.3
    Updating `~~/MyProject/Manifest.toml`
  [7876af07] + Example v0.5.3
Precompiling environment...
  1 dependency successfully precompiled in 2 seconds

julia> readdir("MyProject")
2-element Vector{String}:
 "Manifest.toml"
 "Project.toml"

julia> print(read(joinpath("MyProject", "Project.toml"), String))
[deps]
Example = "7876af07-990d-54b4-ab0e-23690620f79a"

julia> print(read(joinpath("MyProject", "Manifest.toml"), String))
# This file is machine-generated - editing it directly is not advised

julia_version = "1.8.2"
manifest_format = "2.0"
project_hash = "2ca1c6c58cb30e79e021fb54e5626c96d05d5fdc"

[[deps.Example]]
git-tree-sha1 = "46e44e869b4d90b96bd8ed1fdcf32244fddfb6cc"
uuid = "7876af07-990d-54b4-ab0e-23690620f79a"
version = "0.5.3"
```

新环境与我们之前使用的环境完全不同。有关更详细的说明，请参阅 [`Project.toml` 和 `Manifest.toml`](@ref Project-and-Manifest)。

## 使用别人的项目

只需使用 `git clone` 克隆别人的项目， 然后 `cd` 到项目目录并调用：

```julia-repl
shell> git clone https://github.com/JuliaLang/Example.jl.git
Cloning into 'Example.jl'...
...

(@v1.8) pkg> activate Example.jl
Activating project at `~/Example.jl`

(Example) pkg> instantiate
  No Changes to `~/Example.jl/Project.toml`
  No Changes to `~/Example.jl/Manifest.toml`
```

如果项目包含 manifest，这将以该 manifest 给出的相同状态安装包。否则，它将解析与项目兼容的最新版本的依赖项。

请注意，`activate` 本身不会安装缺少的依赖项。如果您只有一个 `Project.toml` 文件， 则 `Manifest.toml` 必须通过“解析”环境来生成，任何丢失的包都必须安装并预编译。`instantiate` 命令为你做了这些工作。

如果您已经有一个已解析的 `Manifest.toml`，那您仍然需要确保所有软件包及其正确版本都已安装。 `instantiate` 再次为您执行这些操作。

简而言之，`instantiate` 确保环境已经准备好可以使用。如果什么也不用做，那么 `instantiate` 什么也不做。

!!! note
    您可以使用 `--project=<path>` 参数在启动时指定项目，替代在 Julia 中使用 `activate`。例如，要使用当前目录中的环境从命令行运行脚本，您可以运行 

    ```bash 
    $ julia --project=. myscript.jl
    ```

## 临时环境

临时环境可以轻松地从空白环境开始以测试一个包或一组包，并让 Pkg 在您测试完后自动删除环境。例如，在编写错误报告时，您可能希望在一个“干净”环境中测试您的最小复现示例，以保证它确实是可复现的。您可能还需要一个临时空间来试用一个新包，或者一个沙箱来解决几个不兼容的包之间的版本冲突。

```julia-repl
(@v1.8) pkg> activate --temp # requires Julia 1.5 or later
  Activating new environment at `/var/folders/34/km3mmt5930gc4pzq1d08jvjw0000gn/T/jl_a31egx/Project.toml`

(jl_a31egx) pkg> add Example
    Updating registry at `~/.julia/registries/General`
   Resolving package versions...
    Updating `/private/var/folders/34/km3mmt5930gc4pzq1d08jvjw0000gn/T/jl_a31egx/Project.toml`
  [7876af07] + Example v0.5.3
    Updating `/private/var/folders/34/km3mmt5930gc4pzq1d08jvjw0000gn/T/jl_a31egx/Manifest.toml`
  [7876af07] + Example v0.5.3
```

## 共享环境

“共享”环境是存在于 `~/.julia/environments` 的简单环境，因此，默认的 `v1.8` 环境是共享环境：

```julia-repl
(@v1.8) pkg> st
Status `~/.julia/environments/v1.8/Project.toml`
```

可以使用 `activate` 命令的 `--shared` 参数激活共享环境 ：

```julia-repl
(@v1.8) pkg> activate --shared mysharedenv
  Activating project at `~/.julia/environments/mysharedenv`

(@mysharedenv) pkg>
```

共享环境在 Pkg REPL 提示符中的名称前有一个 `@`。

## 环境预编译

在导入包之前，Julia 会将源代码“预编译”到磁盘上更高效的中间缓存中。如果未导入的包是新的或自上次缓存以来已发生更改，则可以通过“代码加载”触发此预编译。

```julia-repl
julia> using Example
[ Info: Precompiling Example [7876af07-990d-54b4-ab0e-23690620f79a]
```

或者使用 Pkg 的预编译选项，它可以并行预编译整个环境或给定的依赖项，这比上面的代码加载方式快得多。

```julia-repl
(@v1.8) pkg> precompile
Precompiling environment...
  23 dependencies successfully precompiled in 36 seconds
```

然而，由于 Pkg 的自动预编译机制，以上两种方式通常是不需要的。

### 自动预编译

默认情况下，任何添加到项目或在 Pkg 操作中更新的包以及它的依赖项都将自动预编译。

```julia-repl
(@v1.8) pkg> add Images
   Resolving package versions...
    Updating `~/.julia/environments/v1.9/Project.toml`
  [916415d5] + Images v0.25.2
    Updating `~/.julia/environments/v1.9/Manifest.toml`
    ...
Precompiling environment...
  Progress [===================>                     ]  45/97
  ✓ NaNMath
  ✓ IntervalSets
  ◐ CoordinateTransformations
  ◑ ArnoldiMethod
  ◑ IntegralArrays
  ◒ RegionTrees
  ◐ ChangesOfVariables
  ◓ PaddedViews
```

`develop` 命令是个例外，它既不构建也不预编译包。何时预编译由用户决定。

如果在自动预编译期间给定的包版本出错，Pkg 会记住它，下面的自动尝试都会跳过该包并发出简短的警告。手动预编译可用于强制重试这些包，因为 `pkg> precompile` 将始终重试所有包。

要禁用自动预编译，请设置 `ENV["JULIA_PKG_PRECOMPILE_AUTO"]=0`.

### 预编译已加载包的新版本

如果会话中已经加载了已更新的包，则预编译过程将继续并预编译新版本以及任何依赖它的包，但需注意该包在会话重新启动之前无法使用。
