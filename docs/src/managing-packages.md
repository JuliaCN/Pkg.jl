
# [**3.** 管理包](@id Managing-Packages)

## 添加包

有两种添加包的方法，使用 `add` 或 `dev` 命令。最常用的是 `add`，首先描述它的用法。

### 添加已注册包


在Pkg REPL中，可以使用 `add` 命令后跟包名来添加包，例如：

```julia-repl
(@v1.8) pkg> add JSON
  Installing known registries into `~/`
   Resolving package versions...
   Installed Parsers ─ v2.4.0
   Installed JSON ──── v0.21.3
    Updating `~/.julia/environments/v1.8/Project.toml`
  [682c06a0] + JSON v0.21.3
    Updating `~/environments/v1.9/Manifest.toml`
  [682c06a0] + JSON v0.21.3
  [69de0a69] + Parsers v2.4.0
  [ade2ca70] + Dates
  [a63ad114] + Mmap
  [de0858da] + Printf
  [4ec0a83e] + Unicode
Precompiling environment...
  2 dependencies successfully precompiled in 2 seconds
```

这里我们将包 Example 添加到当前环境（这是默认 `@v1.8` 环境）。在此示例中，我们使用全新的 Julia 安装，这是我们第一次使用 Pkg 添加包。默认情况下，Pkg 安装 [General](https://github.com/JuliaRegistries/General) 注册表并使用此注册表查找请求包含在当前环境中的包。状态更新在左侧显示包 UUID 的简短形式，然后是包名称和版本。最后，新安装的包被“预编译”。

可以以 `pkg> add A B C` 形式在一个命令中添加多个包。

`status` 输出您自己添加的包，在本例中为 `JSON`：

```julia-repl
(@v1.8) pkg> st
    Status `~/.julia/environments/v1.8/Project.toml`
  [682c06a0] JSON v0.21.3
```

`status -m` 显示环境中的所有包，包括递归依赖项：

```julia-repl
(@v1.8) pkg> st -m
Status `~/environments/v1.9/Manifest.toml`
  [682c06a0] JSON v0.21.3
  [69de0a69] Parsers v2.4.0
  [ade2ca70] Dates
  [a63ad114] Mmap
  [de0858da] Printf
  [4ec0a83e] Unicode
```

由于标准库（例如 `Dates`）随 Julia 一起提供，因此它们没有版本。

将包添加到项目后，可以在 Julia 中加载它：

```julia-repl
julia> using JSON

julia> JSON.json(Dict("foo" => [1, "bar"])) |> print
{"foo":[1,"bar"]}
```

!!! note
    只能加载已被 `add` 添加的包（即在 Pkg REPL 中使用 `st` 时显示的包）。仅作为依赖项引入的包（例如上面的 `Parsers` 包）无法加载。

可以通过在包名称后面附加一个 `@` 及版本号来安装包的特定版本：

```julia-repl
(@v1.8) pkg> add JSON@0.21.1
   Resolving package versions...
    Updating `~/.julia/environments/v1.8/Project.toml`
⌃ [682c06a0] + JSON v0.21.1
    Updating `~/environments/v1.9/Manifest.toml`
⌃ [682c06a0] + JSON v0.21.1
⌅ [69de0a69] + Parsers v1.1.2
  [ade2ca70] + Dates
  [a63ad114] + Mmap
  [de0858da] + Printf
  [4ec0a83e] + Unicode
        Info Packages marked with ⌃ and ⌅ have new versions available, but those with ⌅ are restricted by compatibility constraints from upgrading. To see why use `status --outdated -m`
```

如上所示，当软件包安装的不是其最新版本时，Pkg 会提供一些信息。

如果没有为版本提供所有三个数字，例如0.21，则将安装最后注册的版本0.21.x。

如果 `Example` 的某个分支（或某个提交）有一个尚未包含在注册版本中的补丁程序，我们可以通过将 `#branchname`（或 `#commitSHA1`）附加到包名称后来显式跟踪该分支（或提交）：

```julia-repl
(@v1.8) pkg> add Example#master
     Cloning git-repo `https://github.com/JuliaLang/Example.jl.git`
   Resolving package versions...
    Updating `~/.julia/environments/v1.8/Project.toml`
  [7876af07] + Example v0.5.4 `https://github.com/JuliaLang/Example.jl.git#master`
    Updating `~/environments/v1.9/Manifest.toml`
  [7876af07] + Example v0.5.4 `https://github.com/JuliaLang/Example.jl.git#master`
```

现在状态输出显示我们正在跟踪 `Example` 的 `master` 分支. 更新包时，会从该分支中​​提取更新。

!!! note
    如果我们指定一个提交 id 而不是分支名称，例如 `add Example#025cf7e`，那么我们将有效地将包“pin”到该提交。这是因为提交 id 总是指向同一个东西，而不像一个分支，它可能会被更新。

要回到跟踪 `Example` 的注册版本，请使用命令 `free`：

```julia-repl
(@v1.8) pkg> free Example
   Resolving package versions...
   Installed Example ─ v0.5.3
    Updating `~/.julia/environments/v1.8/Project.toml`
  [7876af07] ~ Example v0.5.4 `https://github.com/JuliaLang/Example.jl.git#master` ⇒ v0.5.3
    Updating `~/environments/v1.9/Manifest.toml`
  [7876af07] ~ Example v0.5.4 `https://github.com/JuliaLang/Example.jl.git#master` ⇒ v0.5.3
```

### 添加未注册包

如果一个包不在注册表上，它可以通过指定 Git 仓库的 URL 来添加：

```julia-repl
(@v1.8) pkg> add https://github.com/fredrikekre/ImportMacros.jl
     Cloning git-repo `https://github.com/fredrikekre/ImportMacros.jl`
   Resolving package versions...
    Updating `~/.julia/environments/v1.8/Project.toml`
  [92a963f6] + ImportMacros v1.0.0 `https://github.com/fredrikekre/ImportMacros.jl#master`
    Updating `~/environments/v1.9/Manifest.toml`
  [92a963f6] + ImportMacros v1.0.0 `https://github.com/fredrikekre/ImportMacros.jl#master`
```

未注册包的依赖(这里是 `MacroTools`)也会被安装。对于未注册的包，我们可以使用 `#` 给出一个分支名称（或提交的 SHA1）来跟踪 ，就像已注册包一样。

如果要使用基于 SSH 的 `git` 协议添加包，则必须使用引号，因为 URL 包含 `@`. 例如：

```julia-repl
(@v1.8) pkg> add "git@github.com:fredrikekre/ImportMacros.jl.git"
    Cloning git-repo `git@github.com:fredrikekre/ImportMacros.jl.git`
   Updating registry at `~/.julia/registries/General`
  Resolving package versions...
Updating `~/.julia/environments/v1/Project.toml`
  [92a963f6] + ImportMacros v1.0.0 `git@github.com:fredrikekre/ImportMacros.jl.git#master`
Updating `~/.julia/environments/v1/Manifest.toml`
  [92a963f6] + ImportMacros v1.0.0 `git@github.com:fredrikekre/ImportMacros.jl.git#master`
```

#### 从仓库的子目录中添加包

如果要通过 URL 添加的包不在存储库的根目录中，则需要使用 `:`。例如，要添加[SnoopCompile](https://github.com/timholy/SnoopCompile.jl)仓库中的 `SnoopCompileCore` 包 ：

```julia-repl
pkg> add https://github.com/timholy/SnoopCompile.jl.git:SnoopCompileCore
    Cloning git-repo `https://github.com/timholy/SnoopCompile.jl.git`
   Resolving package versions...
    Updating `~/.julia/environments/v1.8/Project.toml`
  [e2b509da] + SnoopCompileCore v2.9.0 `https://github.com/timholy/SnoopCompile.jl.git:SnoopCompileCore#master`
    Updating `~/.julia/environments/v1.8/Manifest.toml`
  [e2b509da] + SnoopCompileCore v2.9.0 `https://github.com/timholy/SnoopCompile.jl.git:SnoopCompileCore#master`
  [9e88b42a] + Serialization
```

### 添加本地包

通过传递给 `add` 一个git仓库的**本地路径**来代替git仓库的 URL，它能像 URL 一样工作。将跟踪本地存储库（在某个分支），并在更新包时从该本地存储库中提取更新。

!!! warning
    请注意，通过 `add` 跟踪包的方式不同于 `develop`（将在下一节中描述）。在本地 git 存储库上使用 `add` 时，在加载该包时不会立即反映对本地包存储库中文件的更改。必须提交更改并更新软件包才能引入更改。在大多数情况下，您希望在本地路径上使用 `develop`，**而非 `add`**。

### [开发中的包](@id developing)

通过仅使用 `add`，您的环境始终具有“可重现状态”，换句话说，只要使用的仓库和注册表仍然可以访问，就可以检索环境中所有依赖项的确切状态。这样做的好处是您可以将您的环境 (`Project.toml` 和 `Manifest.toml`) 发送给其他人，并且他们可以 [`Pkg.instantiate`](@ref) 此环境，来重现您本地环境的相同状态。然而，当你在开发一个包时，在某个路径上加载它们的当前状态会更方便。因此，存在 `dev` 命令。

让我们尝试 `dev` 一个已注册包：

```julia-repl
(@v1.8) pkg> dev Example
  Updating git-repo `https://github.com/JuliaLang/Example.jl.git`
   Resolving package versions...
    Updating `~/.julia/environments/v1.8/Project.toml`
  [7876af07] + Example v0.5.4 `~/.julia/dev/Example`
    Updating `~/.julia/environments/v1.8/Manifest.toml`
  [7876af07] + Example v0.5.4 `~/.julia/dev/Example`
```

`dev` 命令将包的完整克隆 fetch 到 `~/.julia/dev/`（可以通过设置环境变量 `JULIA_PKG_DEVDIR` 来更改路径，默认为 `joinpath(DEPOT_PATH[1],"dev")`）。现在，当导入 `Example` 时， julia 将从 `~/.julia/dev/Example` 中导入它，对该路径中的文件所做的任何本地更改都会反映在加载的代码中。当使用 `add` 时，我们说跟踪了包存储库；在这里我们说跟踪了路径本身。请注意，包管理器永远不会触及跟踪路径中的任何文件，因此，拉取更新、更改分支等操作都由您自己决定。如果我们尝试 `dev` 已经存在于 `~/.julia/dev/` 的包上的某个分支，包管理器将简单地重新使用现有路径。如果在本地路径上使用 `dev`，该包的路径将被记录并在加载该包时使用。路径将相对于项目文件进行记录，除非它以绝对路径的形式给出。

让我们尝试修改文件 `~/.julia/dev/Example/src/Example.jl` 并添加一个简单的函数：

```julia
plusone(x::Int) = x + 1
```

现在我们可以回到 Julia REPL 加载包并运行新函数：

```julia-repl
julia> import Example
[ Info: Precompiling Example [7876af07-990d-54b4-ab0e-23690620f79a]

julia> Example.plusone(1)
2
```

!!! warning
    每个 Julia 会话只能加载一次包。如果您已在当前 Julia 会话中运行过 `import Example`，则必须重新启动 Julia 才能看到对示例的更改。 [Revise.jl](https://github.com/timholy/Revise.jl/)可以使这个过程变得更轻松，但是设置它超出了本指南的范围。

要停止跟踪路径并再次使用已注册版本，请使用 `free`：

```julia-repl
(@v1.8) pkg> free Example
   Resolving package versions...
    Updating `~/.julia/environments/v1.8/Project.toml`
  [7876af07] ~ Example v0.5.4 `~/.julia/dev/Example` ⇒ v0.5.3
    Updating `~/.julia/environments/v1.8/Manifest.toml`
  [7876af07] ~ Example v0.5.4 `~/.julia/dev/Example` ⇒ v0.5.3
```

应该指出的是，通过使用 `dev` ,您的项目现在本质上是有状态的。它的状态取决于路径中文件的当前内容，并且在不知道跟踪路径中所有包的确切内容的情况下，manifest 不能被其他人“实例化”。

请注意，如果您向跟踪本地路径的包添加依赖项，则 Manifest（包含整个依赖关系图）将与实际依赖关系图不同步。这意味着包将无法加载该依赖项，因为它没有记录在 Manifest 中。要同步 Manifest，请使用 REPL 命令 `resolve`。

除了绝对路径外，`add` 和 `dev` 还可以接受包的相对路径。在这种情况下，将存储从活动项目到包的相对路径。当被跟踪依赖项的相对位置比它们的绝对位置更重要时，这种方法很有用。例如，跟踪的依赖项可以存储在活动项目目录中。整个目录可以移动并且 `Pkg` 仍然能够找到依赖项，因为它们对于活动项目的相对路径被保留，即使它们的绝对路径已经发生改变。

## 移除包

可以使用 `pkg> rm Package` 从当前项目中移除包。这只会移除项目中存在的包；要移除仅作为依赖项存在的包，请使用 `pkg> rm --manifest DepPackage`。请注意，这将移除（递归地）`DepPackage` 依赖的所有包。

## [更新包](@id updating)

发布新版本的软件包时，最好进行更新。只需调用 `up` 命令，尝试将项目的**所有依赖**更新到最新的兼容版本。有时这不是你想要的。您可以通过将依赖项的子集作为参数提供给 `up` 来指定要升级的包，例如：

```julia-repl
(@v1.8) pkg> up Example
```

这将只允许 `Example` 进行升级。如果您还想允许 `Example` 的依赖项升级（项目中的包除外），您可以传递 `--preserve=direct` 参数。

```julia-repl
(@v1.8) pkg> up --preserve=direct Example
```

如果您还想允许项目中的 `Example` 的依赖项进行升级，您可以使用 `--preserve=none`：

```julia-repl
(@v1.8) pkg> up --preserve=none Example
```
## 固定包

固定包永远不会更新。可以使用 `pin` 来固定一个包，例如：

```julia-repl
(@v1.8) pkg> pin Example
 Resolving package versions...
  Updating `~/.julia/environments/v1.8/Project.toml`
  [7876af07] ~ Example v0.5.3 ⇒ v0.5.3 ⚲
  Updating `~/.julia/environments/v1.8/Manifest.toml`
  [7876af07] ~ Example v0.5.3 ⇒ v0.5.3 ⚲
```

请注意 `⚲` 符号显示包已经被固定。要移除固定使用 `free` 命令：

```julia-repl
(@v1.8) pkg> free Example
  Updating `~/.julia/environments/v1.8/Project.toml`
  [7876af07] ~ Example v0.5.3 ⚲ ⇒ v0.5.3
  Updating `~/.julia/environments/v1.8/Manifest.toml`
  [7876af07] ~ Example v0.5.3 ⚲ ⇒ v0.5.3
```

## 测试包

可以使用 `test` 命令运行包的测试：
The tests for a package can be run using `test` command:

```julia-repl
(@v1.8) pkg> test Example
...
   Testing Example
   Testing Example tests passed
```

## 构建包

首次安装包时，包的构建步骤会自动运行。构建过程的输出被定向到一个文件。要显式运行包的构建步骤，请使用 `build` 命令：

```julia-repl
(@v1.8) pkg> build IJulia
    Building Conda ─→ `~/.julia/scratchspaces/44cfe95a-1eb2-52ea-b672-e2afdf69b78f/6e47d11ea2776bc5627421d59cdcc1296c058071/build.log`
    Building IJulia → `~/.julia/scratchspaces/44cfe95a-1eb2-52ea-b672-e2afdf69b78f/98ab633acb0fe071b671f6c1785c46cd70bb86bd/build.log`

julia> print(read(joinpath(homedir(), ".julia/scratchspaces/44cfe95a-1eb2-52ea-b672-e2afdf69b78f/98ab633acb0fe071b671f6c1785c46cd70bb86bd/build.log"), String))
[ Info: Installing Julia kernelspec in /home/kc/.local/share/jupyter/kernels/julia-1.8
```

## [解释和解决版本冲突](@id conflicts)

一个环境由一组相互兼容的包组成。有时，您会发现自己要同时使用的两个软件包对环境的要求互相不兼容。在这种情况下，您会收到“Unsatisfiable requirements”错误：

```@setup conflict
using Pkg
include(joinpath(pkgdir(Pkg), "test", "resolve_utils.jl"))
using .ResolveUtils
deps_data = Any[["A", v"1.0.0", "C", v"0.2"],
                ["B", v"1.0.0", "D", v"0.1"],
                ["C", v"0.1.0", "D", v"0.1"],
                ["C", v"0.1.1", "D", v"0.1"],
                ["C", v"0.2.0", "D", v"0.2"],
                ["D", v"0.1.0"],
                ["D", v"0.2.0"],
                ["D", v"0.2.1"]]
reqs_data = Any[["A", "*"],
                ["B", "*"]]
```

```@example conflict
print("pkg> add A\n", try resolve_tst(deps_data, reqs_data) catch e sprint(showerror, e) end)   # hide
```

此消息意味着一个名为 `D` 的包有版本冲突，即使您从未直接 `add` 过 `D`。如果您尝试使用的其他软件包也需要 `D`，同样会出现这种错误。

!!! note
    在处理这些冲突时，首先要考虑项目越大，发生这种情况的可能性就越大。强烈建议为给定任务使用专用项目，并且在遇到这些问题时删除未使用的依赖项。例如，一个常见的陷阱是，在您的默认环境（即 `(@1.8)`）中包含有多个包，并将其用作您使用 julia 执行的所有任务的环境。最好为您正在处理的任务创建一个专用项目，并将依赖关系保持在最低限度。要了解更多信息，请参阅 [使用“环境”](@ref Working-with-Environments)

错误消息包含很多关键信息。分段解释可能最容易：

```
Unsatisfiable requirements detected for package D [756980fe]:
 D [756980fe] log:
 ├─possible versions are: [0.1.0, 0.2.0-0.2.1] or uninstalled
```

表示 `D` 具有三个发布版本，`v0.1.0`、`v0.2.0`和`v0.2.1`。您还可以选择根本不安装它。这些选项中的每一个都可能对可以安装的其他软件包集具有不同的含义。

更重要的是，请注意笔画字符（垂直和水平线）及其缩进。这些字符组合在一起将*消息*连接到特定的*包*。例如，右边的笔划 `├─` 表示其右边的消息 (`possible versions...`) 连接到其垂直笔划指向的包(`D`) 。同样的规则适用于下一行：

```
 ├─restricted by compatibility requirements with B [f4259836] to versions: 0.1.0
```

这里的垂直笔划也对齐在 `D` 下方，因此该消息是引用到 `D` 的。具体来说，还有另外的包 `B` 依赖于 `D` 的 `v0.1.0`版本. 请注意，这不是 `D` 的最新版本。

接下来是一些关于 `B` 的信息：

```
 │ └─B [f4259836] log:
 │   ├─possible versions are: 1.0.0 or uninstalled
 │   └─restricted to versions * by an explicit requirement, leaving only versions 1.0.0
```

第一行下面的两行有一个与 `B` 对齐的垂直笔划，因此它们提供了关于 `B` 的信息。他们告诉你 `B` 只有一个版本 `v1.0.0`。您没有指定过 ` B` 的特定版本(`restricted to versions *` 表示任何版本都可以)，但是 `explicit requirement` 表明您要求B成为环境的一部分，例如曾执行过 `pkg> add B`。您之前可能已经添加过 `B` 了，但这里对特定版本的依赖仍然有效。

冲突变得清晰了：

```
└─restricted by compatibility requirements with C [c99a7cb2] to versions: 0.2.0 — no versions left
```

同样，垂直笔划与 `D` 对齐：这意味着 `D` *也*被 `C` 包依赖，`C` 依赖 `D` 的 `v0.2.0` 版本。这和 `B` 依赖 `D` 的 `v0.1.0` 版本是冲突的。

但是等等，你可能会问，`C` 是什么，我为什么需要它？接下来的几行介绍了这个问题：

```
   └─C [c99a7cb2] log:
     ├─possible versions are: [0.1.0-0.1.1, 0.2.0] or uninstalled
     └─restricted by compatibility requirements with A [29c70717] to versions: 0.2.0
```

这些提供了有关 `C` 的更多信息，表明它有 3 个已发布版本：`v0.1.0`、`v0.1.1`和`v0.2.0`。此外，`C` 是另一个包 `A` 所必需的。确实，`A` 要求 `v0.2.0`版本的 `C`。 `A` 的起源在接下来的几行中揭示：

```
       └─A [29c70717] log:
         ├─possible versions are: 1.0.0 or uninstalled
         └─restricted to versions * by an explicit requirement, leaving only versions 1.0.0
```

所以我们可以看到 `A` 是 `explicitly required` 的, 并且在此示例中，我们尝试将它 `add` 到我们的环境中。

总之，我们明确要求使用 `A` 和 `B`，但这里显示了一个 `D` 的冲突，原因是 `B` 和 `C` 依赖的 `D` 的版本冲突。 尽管 `C` 不是我们明确要求的，但 `A` 是。

要修复此类错误，您有多种选择：

- 尝试[更新包](@ref updating)。这些软件包的开发人员可能最近发布了相互兼容的新版本。
- 从您的环境中删除 `A` 或删除 `B`。也许 `B` 是你以前工作的时候遗留下来的，你已经不再需要它了。如果您同时不需要 `A`和`B`，这是解决问题的最简单方法。
- 尝试报告您的冲突。在这种情况下，我们能够推断出 `B` 依赖 `D` 的过时版本。 因此，您可以在 `B.jl` 的开发存储库中报告问题，要求更新 `D` 版本。
- 尝试自己解决问题。一旦您了解 `Project.toml` 文件以及它们如何声明其兼容性要求，这将变得更容易。我们将在[修复冲突](@ref)中回到这个例子。

## 垃圾收集旧的、未使用的包

随着包的更新和项目的删除，曾经使用过的已安装包的版本和组件将不可避免地变旧，并且不会再被任何现有项目使用。`Pkg` 保留所有项目的使用日志，以便它可以查看日志并准确查看哪些项目仍然存在，以及这些项目使用了哪些包/[artifacts](@ref Artifacts)。如果包或 artifacts 未被任何项目标记为正在使用，则会将其添加到孤立包列表中。处于孤立列表中 30 天未再次使用的包和 artifacts 会在下一次垃圾回收时从系统中删除。这个时间可以通过 `Pkg.gc()` 的 `collect_delay` 关键字参数来配置。值 `0` 将导致立即收集当前未使用的任何内容，完全跳过孤儿列表；如果您的磁盘空间不足，并且想清理尽可能多的未使用的包和 artifacts，您可能想尝试此参数，但如果您再次需要这些版本，则必须重新下载它们。要使用默认参数运行典型的垃圾收集，只需使用 `pkg>` REPL中的 `gc` 命令：

```julia-repl
(@v1.8) pkg> gc
    Active manifests at:
        `~/BinaryProvider/Manifest.toml`
        ...
        `~/Compat.jl/Manifest.toml`
    Active artifacts:
        `~/src/MyProject/Artifacts.toml`

    Deleted ~/.julia/packages/BenchmarkTools/1cAj: 146.302 KiB
    Deleted ~/.julia/packages/Cassette/BXVB: 795.557 KiB
   ...
   Deleted `~/.julia/artifacts/e44cdf2579a92ad5cbacd1cddb7414c8b9d2e24e` (152.253 KiB)
   Deleted `~/.julia/artifacts/f2df5266567842bbb8a06acca56bcabf813cd73f` (21.536 MiB)

   Deleted 36 package installations (113.205 MiB)
   Deleted 15 artifact installations (20.759 GiB)
```

请注意，只有 `~/.julia/packages` 中的包被删除。

## 离线模式

在离线模式下，Pkg 尝试在不连接互联网的情况下尽可能做的更多。例如，当添加一个包时，Pkg 只考虑版本解析中已经下载的版本。

要在离线模式下工作，请使用 `import Pkg; Pkg.offline(true)` 或将环境变量 `JULIA_PKG_OFFLINE` 设置为 `"true"`。

## Pkg 客户端/服务器

当你添加一个新的已注册包时，通常会发生三件事：

1. 更新注册表，
2. 下载包的源代码，
3. 如果不可用，请下载软件包所需的 [artifacts](@ref Artifacts)。

[General](https://github.com/JuliaRegistries/General) 注册表和其中的大多数包都是在 GitHub 上开发的，而 artifacts 数据则托管在各种平台上。当与 GitHub 和 AWS S3 的网络连接不稳定时，安装或更新包的体验通常不是很好。幸运的是，pkg 客户端/服务器功能在以下方面改善了体验：

1. 如果设置，pkg 客户端将首先尝试从 pkg 服务器下载数据，
2. 如果失败，则返回到从原始来源（例如，GitHub）下载。

从 Julia 1.5 开始，`https://pkg.julialang.org` 由 JuliaLang 组织提供并作为默认 pkg 服务器。在大多数情况下，这应该是透明的，但用户仍然可以通过环境变量 `JULIA_PKG_SERVER` 设置/取消设置上游的 pkg 服务器。

```julia
# manually set it to some pkg server
julia> ENV["JULIA_PKG_SERVER"] = "pkg.julialang.org"
"pkg.julialang.org"

# unset to always download data from original sources
julia> ENV["JULIA_PKG_SERVER"] = ""
""
```

澄清一下，Pkg 服务器未提供某些来源。

* 通过 `git` 获取包/注册表
  * `]add https://github.com/JuliaLang/Example.jl.git`
  * `]add Example#v0.5.3`（请注意，这与 `]add Example@0.5.3` 不同）
  * `]registry add https://github.com/JuliaRegistries/General.git`，包括 Julia 在 1.4 之前安装的注册表。
* 没有下载信息的artifacts
  * [TestImages](https://github.com/JuliaImages/TestImages.jl/blob/eaa94348df619c65956e8cfb0032ecddb7a29d3a/Artifacts.toml#L1-L2)

!!! note
    如果您通过 pkg 服务器安装了新的注册表，那么旧 Julia 版本不可能更新注册表，因为 1.4 之前的 Julia 不知道如何获取新数据。因此，对于经常在多个 Julia 版本之间切换的用户，建议仍然使用 git 控制注册表。

pkg server 的部署请参考 [PkgServer.jl](https://github.com/JuliaPackaging/PkgServer.jl)。
