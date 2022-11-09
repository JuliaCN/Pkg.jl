
# **5.** 创建包

## 为包生成文件

!!! note
    [PkgTemplates](https://github.com/invenia/PkgTemplates.jl)包提供了一种简单、可重复和可定制的方式来为新包生成文件。它还可以生成文档、CI 等所需的文件。我们建议您使用 PkgTemplates 来创建新包，而不是使用下面描述的最小 `pkg> generate` 功能。

要为新包生成最少的文件，请使用 `pkg> generate`.

```julia-repl
(@v1.8) pkg> generate HelloWorld
```

这将创建一个包含以下文件的新项目 `HelloWorld`（使用外部 [`tree` command](https://linux.die.net/man/1/tree) 命令可视化）：

```julia-repl
julia> cd("HelloWorld")

shell> tree .
.
├── Project.toml
└── src
    └── HelloWorld.jl

1 directory, 2 files
```

`Project.toml` 文件包含包名、唯一的 UUID、版本、作者和潜在的依赖项：

```toml
name = "HelloWorld"
uuid = "b4cd1eb8-1e24-11e8-3319-93036a3eb9f3"
version = "0.1.0"
authors = ["Some One <someone@email.com>"]

[deps]
```

`src/HelloWorld.jl` 的内容是:

```julia
module HelloWorld

greet() = print("Hello World!")

end # module
```

我们现在可以激活项目并加载包：

```julia-repl
pkg> activate .

julia> import HelloWorld

julia> HelloWorld.greet()
Hello World!
```

## 将依赖项添加到项目中

假设我们要在我们的项目中使用标准库包 `Random` 及已注册包 `JSON`。我们只 `add` 这些包（注意提示符现在如何显示新生成的项目的名称，因为我们已经 `activated` 了它）：

```julia-repl
(HelloWorld) pkg> add Random JSON
   Resolving package versions...
    Updating `~/HelloWorld/Project.toml`
  [682c06a0] + JSON v0.21.3
  [9a3f8284] + Random
    Updating `~/HelloWorld/Manifest.toml`
  [682c06a0] + JSON v0.21.3
  [69de0a69] + Parsers v2.4.0
  [ade2ca70] + Dates
 ...
```

`Random` 和 `JSON`都被添加到项目的 `Project.toml` 文件中，并且生成的依赖项被添加到 `Manifest.toml` 文件中。解析器已经尽可能安装了每个包的高版本，同时仍然尊重每个包对其依赖项强制执行的兼容性。

现在可以在我们的项目中使用 `Random` 和 `JSON`。修改 `src/HelloWorld.jl` 为：

```julia
module HelloWorld

import Random
import JSON

greet() = print("Hello World!")
greet_alien() = print("Hello ", Random.randstring(8))

end # module
```

重新加载包，使用 `Random` 的新函数 `greet_alien` 可以被调用：

```julia-repl
julia> HelloWorld.greet_alien()
Hello aT157rHV
```

## 向包中添加构建步骤

构建步骤在首次安装包或使用 `build` 时执行，通过执行文件 `deps/build.jl` 构建一个包。

```julia-repl
julia> print(read("deps/build.jl", String))
println("I am being built...")

(HelloWorld) pkg> build
  Building HelloWorld → `deps/build.log`
 Resolving package versions...

julia> print(read("deps/build.log", String))
I am being built...
```

如果构建步骤失败，则将构建步骤的输出打印到控制台

```julia-repl
julia> print(read("deps/build.jl", String))
error("Ooops")

(HelloWorld) pkg> build
    Building HelloWorld → `~/HelloWorld/deps/build.log`
ERROR: Error building `HelloWorld`:
ERROR: LoadError: Ooops
Stacktrace:
 [1] error(s::String)
   @ Base ./error.jl:35
 [2] top-level scope
   @ ~/HelloWorld/deps/build.jl:1
 [3] include(fname::String)
   @ Base.MainInclude ./client.jl:476
 [4] top-level scope
   @ none:5
in expression starting at /home/kc/HelloWorld/deps/build.jl:1
```

!!! warning
    构建步骤通常不应在包目录中创建或修改任何文件。如果您需要存储构建步骤中的一些文件，请使用 [Scratch.jl](https://github.com/JuliaPackaging/Scratch.jl) 包。

## 向包中添加测试

当一个包被测试时，文件 `test/runtests.jl` 被执行：

```julia-repl
julia> print(read("test/runtests.jl", String))
println("Testing...")

(HelloWorld) pkg> test
   Testing HelloWorld
 Resolving package versions...
Testing...
   Testing HelloWorld tests passed
```

测试在一个新的 Julia 进程中运行，其中包本身以及任何特定于测试的依赖项都可用，见下文。

!!! warning
    测试通常不应在包目录中创建或修改任何文件。如果您需要存储构建步骤中的一些文件，请使用[Scratch.jl](https://github.com/JuliaPackaging/Scratch.jl)包。

### 特定于测试的依赖项

有两种方法可以添加特定于测试的依赖项（依赖项不是包的依赖项，但在测试包时仍然可以加载）。

#### 基于 `target` 测试特定依赖项

使用此方法添加特定于测试的依赖项，将包添加到 `[extras]` 节下并添加到测试目标，例如要添加 `Markdown` 和 `Test` 作为测试依赖项，添加以下内容到Project.toml文件。

```toml
[extras]
Markdown = "d6f4376e-aef5-505a-96c1-9c027394607a"
Test = "8dfed614-e22c-5e08-85e1-65c5234f0b40"

[targets]
test = ["Markdown", "Test"]
```

此例中除了 `test` 之外，没有其他“targets” 。

#### `test/Project.toml` 文件测试特定依赖项

!!! note
    `Project.toml`，`test/Project.toml` 以及它们对应的 `Manifest.toml` 之间的确切交互尚未完全确定，并且在未来的版本中可能会发生变化。因此，所有 Julia 1.X 版本都支持添加特定于测试的依赖项的旧方法，将在下一节中介绍。

由于是由 `test/Project.toml` 给出的，因此，在运行测试时，这将会成为活动项目，并且只能使用依赖于 `test/Project.toml` 的项目。请注意，Pkg 将隐式添加测试包本身。

!!! note
    如果 `test/Project.toml` 不存在, Pkg 将使用基于`target`测试特定依赖项。

要添加特定于测试的依赖项（即仅在测试时可用的依赖项），只需将此依赖项添加到 `test/Project.toml` 项目中就足够了。可以通过从 Pkg REPL 中激活此环境，然后像平时一样使用 `add` 添加依赖。让我们添加 `Test` 标准库作为测试依赖项：

```julia-repl
(HelloWorld) pkg> activate ./test
[ Info: activating environment at `~/HelloWorld/test/Project.toml`.

(test) pkg> add Test
 Resolving package versions...
  Updating `~/HelloWorld/test/Project.toml`
  [8dfed614] + Test
  Updating `~/HelloWorld/test/Manifest.toml`
  [...]
```

现在可以在测试脚本中使用 `Test` ，可以看到它在测试时被安装：

```julia-repl
julia> print(read("test/runtests.jl", String))
using Test
@test 1 == 1

(HelloWorld) pkg> test
   Testing HelloWorld
 Resolving package versions...
  Updating `/var/folders/64/76tk_g152sg6c6t0b4nkn1vw0000gn/T/tmpPzUPPw/Project.toml`
  [d8327f2a] + HelloWorld v0.1.0 [`~/.julia/dev/Pkg/HelloWorld`]
  [8dfed614] + Test
  Updating `/var/folders/64/76tk_g152sg6c6t0b4nkn1vw0000gn/T/tmpPzUPPw/Manifest.toml`
  [d8327f2a] + HelloWorld v0.1.0 [`~/.julia/dev/Pkg/HelloWorld`]
   Testing HelloWorld tests passed```
```

## 依赖的兼容性

一般来说，每个依赖项都应该有一个兼容性约束。这是一个重要的话题，所以有一个单独的章节：[兼容性](@ref Compatibility)。

## [包命名准则](@id Package-naming-guidelines)

对于大多数 Julia 用户来说，包名应该是切合实际的，*即使对于那些不是领域专家的人来说也是如此*。以下指南适用于 `General` 注册表，但也可能对其他包注册表有用。

由于 `General` 注册表属于整个社区，所以当你发布包时，人们可能会对你的包名有意见，特别是如果它不明确或可能与其他东西混淆时。通常，您会得到一个可能更适合您的包的新名称的建议。

1. 避免行话。特别是要避免使用首字母缩略词，除非混淆的可能性很小。

  * 如果您在谈论美国，可以使用 `USA`。
  * 即使您在谈论积极的心态，也不能使用 `PMA`。

2. 避免在你的包名中使用 `Julia` 或在它前面加上 `Ju`。

  * 您的用户通常可以从上下文中清楚地知道该包是 Julia 包。
  * 包名称已经有一个 `.jl` 扩展，它向用户表明了 `Package.jl` 是个Julia 包。
  * 名称中包含 Julia 可能意味着该包与 Julia 语言本身的贡献者相关或由其认可。
  
3. 提供与新类型相关的大部分功能的包应该具有英文单词复数形式的名称。

  * `DataFrames` 提供 `DataFrame` 类型。
  * `BloomFilters` 提供 `BloomFilter` 类型。
  * 相比之下，`JuliaParser` 不提供新类型，而是在函数 `JuliaParser.parse()` 中提供新功能。

4. 在清晰的表达方面犯错，因为清晰的表达在你看来可能更冗长。

  * `RandomMatrices` 是一个比 `RndMator` 或 `RMT`更明确的名称 ，即使后者更短。

5. 不太系统的名称可能适合实现其领域内的几种可能方法之一的包。

  * Julia 没有单一的综合绘图包。相反，`Gadfly`、`PyPlot`、`Winston` 和其他包都实现了基于特定设计理念的独特方法。
  * 相反，`SortingAlgorithms` 提供了一个一致的接口来使用许多成熟的排序算法。

6. 包装外部库或程序的包应以这些库或程序命名。

  * `CPLEX.jl` 包装了 `CPLEX` 库, 它可以在 web 搜索中轻松识别。
  * `MATLAB.jl` 提供从 Julia 内部调用 MATLAB 引擎的接口。

7. 避免与现有包相似的名称
     * `Websocket` 和 `WebSockets` 太相似，容易被用户混淆。 应该使用新名称，例如 `SimpleWebsockets`。

## 注册包

一旦一个包准备好了，它就可以在[General Registry](https://github.com/JuliaRegistries/General#registering-a-package-in-general)注册表中注册（另请参阅[FAQ](https://github.com/JuliaRegistries/General#faq)）。目前，包是通过[`Registrator`](https://juliaregistrator.github.io/)提交的。 此外 `Registrator`，[`TagBot`](https://github.com/marketplace/actions/julia-tagbot)帮助管理标记发布的过程。

## 最佳实践

包应该避免改变自己的状态（写入包目录中的文件）。一般来说，包不应该假设它们位于可写位置（例如，作为系统级 depot 的一部分安装）甚至是稳定的位置（例如，它们通过[PackageCompiler.jl](https://github.com/JuliaLang/PackageCompiler.jl)捆绑到系统映像中）。为了支持 Julia 包生态系统中的各种用例，Pkg 开发人员创建了许多辅助包和技术来帮助包作者创建自包含、不可变和可重定位的包：

* [`Artifacts`](https://pkgdocs.julialang.org/v1/artifacts/)可用于将数据块与您的包捆绑在一起，甚至允许它们按需下载。相比尝试通过诸如 `joinpath(@__DIR__, "data", "my_dataset.csv")` 不可重定位的路径打开文件，artifacts 更优雅。一旦你的包被预编译，`@__DIR__`的结果将被自动加入到你的预编译包数据中，如果你尝试分发这个包，它会尝试在错误的位置加载文件。可以使用 `artifact"name"` 字符串宏轻松绑定和访问 Artifacts。

* [`Scratch.jl`](https://github.com/JuliaPackaging/Scratch.jl)提供“暂存空间”的概念，即包的可变数据容器。暂存空间专为完全由包管理的数据缓存而设计，在卸载包时应将其删除。对于重要的用户生成的数据，包应继续写入到用户指定的路径，该路径不受 Julia 或 Pkg 管理。

* [`Preferences.jl`](https://github.com/JuliaPackaging/Preferences.jl)允许包读写选项偏好(preferences)到顶级 `Project.toml`. 这些选项偏好可以在运行时或编译时读取，以启用或禁用包行为的不同方面。以前，包会将文件写入自己的包目录以记录用户或环境设置的选项，但现在非常不鼓励这样做，使用 `Preferences` 可以做的更好。

