
# **2.** 入门

以下是对 Pkg 基本功能的快速概述。它应该可以帮助新用户熟悉基本的 Pkg 功能，例如添加和删除包以及使用环境。  

!!! note
    本节省略了一些 Pkg 输出，以保持基本指南的重点。这将有助于保持良好的节奏，避免陷入细节的困境中。如果您需要更多详细信息，请参阅 Pkg 手册的后续部分。

!!! note
    本指南使用 Pkg REPL 执行 Pkg 命令。对于非交互式使用，我们推荐使用 Pkg API。Pkg API 完整记录在 Pkg 文档的[API Reference](@ref)部分。

## 基本用法

Pkg 带有一个 REPL。在 Julia REPL 中按 `]` 进入 Pkg REPL。要返回 Julia REPL，请按 `Ctrl+C` 或 `backspace`（当 REPL 光标位于输入的开头时）。

进入 Pkg REPL 后，您应该会看到以下提示：

```julia-repl
(@v1.8) pkg>
```

要添加包，使用 `add`:

```julia-repl
(@v1.8) pkg> add Example
   Resolving package versions...
   Installed Example ─ v0.5.3
    Updating `~/.julia/environments/v1.8/Project.toml`
  [7876af07] + Example v0.5.3
    Updating `~/.julia/environments/v1.8/Manifest.toml`
  [7876af07] + Example v0.5.3
```

安装包后，可以将其加载到 Julia 会话中：

```julia-repl
julia> import Example

julia> Example.hello("friend")
"Hello, friend"
```

还可以一次指定多个包进行安装：

```julia-repl
(@v1.8) pkg> add JSON StaticArrays
```

`status` 命令 (或更短的 `st` 命令) 可以用于查看已安装的包.

```julia-repl
(@v1.8) pkg> st
Status `~/.julia/environments/v1.6/Project.toml`
  [7876af07] Example v0.5.3
  [682c06a0] JSON v0.21.3
  [90137ffa] StaticArrays v1.5.9
```

!!! note
    某些 Pkg REPL 命令具有长版本和短版本，例如`status` and `st`。

要移除包，使用 `rm` (或 `remove`):

```julia-repl
(@v1.8) pkg> rm JSON StaticArrays
```

使用 `up` (或 `update`) 来更新已安装的包：

```julia-repl
(@v1.8) pkg> up
```

如果您一直遵循本指南，则安装的软件包可能是最新版本，因此 `up` 不会执行任何操作。下面展示在故意安装旧版本的示例包，然后升级它的情况下的状态输出：

```julia-repl
(@v1.8) pkg> st
Status `~/.julia/environments/v1.8/Project.toml`
⌃ [7876af07] Example v0.5.1
Info Packages marked with ⌃ have new versions available and may be upgradable.

(@v1.8) pkg> up
    Updating `~/.julia/environments/v1.8/Project.toml`
  [7876af07] ↑ Example v0.5.1 ⇒ v0.5.3
```

我们可以看到状态输出告诉我们有更新的版本可用并且 `up` 升级了包。

有关管理包的更多信息，请参阅文档的 [管理包](@ref Managing-Packages) 部分。


## 环境入门

目前为止，我们已经介绍了基本的包管理：添加、更新和删除包。

您可能已经注意到 REPL 提示符中的 `(@v1.8)`。这让我们知道 `v1.8`是**活动环境**。不同的环境可以安装完全不同的包和版本。活动环境是将被 Pkg 命令修改的环境，例如 `add`,`rm` 和 `update`。

让我们建立一个新环境，以便进行实验。要设置活动环境，请使用 `activate`：

```julia-repl
(@v1.8) pkg> activate tutorial
[ Info: activating new environment at `~/tutorial/Project.toml`.
```

Pkg 让我们知道我们正在创建一个新环境，并且该环境将存储在~/tutorial目录中。环境的路径是相对于 REPL 的当前工作目录创建的。

Pkg 还更新了 REPL 提示符以反映新的活动环境：

```julia-repl
(tutorial) pkg>
```

我们可以使用 `status` 命令查询有关活动环境的信息:

```julia-repl
(tutorial) pkg> status
    Status `~/tutorial/Project.toml`
   (empty environment)
```

`~/tutorial/Project.toml` 是活动环境的**项目文件**。项目文件是一个[TOML](https://toml.io/en/)文件，Pkg 在这里存储已显式安装的包。请注意，这个新环境是空的。让我们添加一些包并观察：

```julia-repl
(tutorial) pkg> add Example JSON
...

(tutorial) pkg> status
    Status `~/tutorial/Project.toml`
  [7876af07] Example v0.5.3
  [682c06a0] JSON v0.21.3
```

 我们可以看到 `tutorial` 环境现在包含 `Example` 和 `JSON`。

!!! note
    如果您在多个环境中安装了相同的软件包（相同版本），则该软件包只会被下载并存储在硬盘上一次。这使得环境非常轻量级并且可以有效地自由创建。仅使用包含大量包的默认环境是 Julia 初学者的常见错误。学习如何有效地使用环境将改善您使用 Julia 包的体验。

有关环境的更多信息，请参阅文档的 [使用“环境”](@ref Working-with-Environments) 部分。

## 寻求帮助

如果您遇到困难，可以寻求 `Pkg` 帮助：

```julia-repl
(@v1.8) pkg> ?
```

您应该会看到可用命令列表以及简短说明。您可以通过指定命令来寻求更详细的帮助：

```julia-repl
(@v1.8) pkg> ?develop
```

本指南应该可以帮助您开始使用 `Pkg`。 `Pkg` 在强大的包管理方面提供更多功能，请阅读完整手册以了解更多信息！
