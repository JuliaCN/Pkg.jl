
# [**10.** `Project.toml` and `Manifest.toml`](@id Project-and-Manifest)

Pkg 的两个核心文件是 `Project.toml` 和 `Manifest.toml`。`Project.toml` 和 `Manifest.toml` 都是用 [TOML](https://github.com/toml-lang/toml)（因此是 `.toml` 扩展名）编写的，包括有关依赖项、版本、包名称、UUID 等的信息。

!!! note
    `Project.toml` 和 `Manifest.toml` 文件不仅由包管理器使用；Julia 的代码加载也使用它们，并确定例如 `using Example` 应该做什么。有关更多详细信息，请参阅 Julia 手册中有关 [代码加载](https://docs.julialang.org/en/v1/manual/code-loading/) 的部分。

## `Project.toml`

项目文件从高层次上描述了项目，例如，项目文件中列出了包/项目的依赖关系和兼容性约束。文件条目如下所述。

### `authors` 字段

对于包，`authors` 可选字段是描述包作者的字符串列表，格式为 `NAME <EMAIL>`。例如：

```toml
authors = ["Some One <someone@email.com>",
           "Foo Bar <foo@bar.com>"]
```

### `name` 字段

包/项目的名称由 `name` 字段决定，例如：

```toml
name = "Example"
```

名称必须是有效[标识符](https://docs.julialang.org/en/v1/base/base/#Base.isidentifier)（不以数字开头且既不是 `true` 也不是的 `false`  Unicode 字符序列）。对于包，建议遵循 [包命名准则](@ref Package-naming-guidelines)。`name` 字段对于包是必需的。

### `uuid` 字段

`uuid` 是包/项目 [通用唯一标识符]( https://en.wikipedia.org/wiki/Universally_unique_identifier ) 字符串，例如：

```toml
uuid = "7876af07-990d-54b4-ab0e-23690620f79a"
```

`uuid` 字段对于包是必需的。

!!! note
    建议使用 `UUIDs.uuid4()` 生成随机 UUID。

### `version` 字段

`version` 是包/项目版本号的字符串。它应该由三个数字组成：主要版本号、次要版本号和补丁号，用 `.` 分隔，例如：

```toml
version = "1.2.5"
```

Julia 使用[语义版本控制](https://semver.org/)(SemVer)，该 `version` 字段应遵循 SemVer。基本规则是：

* 在 1.0.0 之前，一切正常，但是当您进行重大更改时，次要版本应该增加。
* 在 1.0.0 之后，仅在增加主要版本时进行重大更改。
* 在 1.0.0 之后，如果不增加次要版本，则不应添加新的公共 API。这尤其包括来自 `Base` 或其他包的新类型、函数、方法和方法重载 。

另请参阅[兼容性](@ref Compatibility)部分。

请注意，当涉及到 1.0.0 之前的版本时，`Pkg.jl` 偏离了 SemVer 规范。有关详细信息，请参阅 [pre-1.0 行为](@ref compat-pre-1.0) 部分。

### `[deps]` 节

该节列出了包/项目的所有依赖项[deps]。每个依赖项都以“名称-uuid”对的形式列出，例如：

```toml
[deps]
Example = "7876af07-990d-54b4-ab0e-23690620f79a"
Test = "8dfed614-e22c-5e08-85e1-65c5234f0b40"
```

通常不需要手动向 `[deps]` 节添加条目；这由 Pkg 操作来处理，例如 `add`。

### `[compat]` 节

`[deps]` 下面列出的依赖项的兼容性约束可以在该 `[compat]` 节中列出。例子：

```toml
[deps]
Example = "7876af07-990d-54b4-ab0e-23690620f79a"

[compat]
Example = "1.2"
```

兼容性章节详细描述了不同的可能兼容性约束。也可以列出 `julia` 自身的约束，尽管 `julia` 未在 `[deps]` 节中列为依赖项：

```toml
[compat]
julia = "1.1"
```


## `Manifest.toml`

manifest 文件是环境中包状态的绝对记录。它包括有关项目（直接和间接）依赖项的确切信息。给定一对 `Project.toml` 和 `Manifest.toml`，可以实例化完全相同的包环境，这对于可复现性非常有用。有关详细信息，请参阅 [`Pkg.instantiate`](@ref)。

!!! note
    `Manifest.toml` 文件由 Pkg 生成和维护，一般情况下，不应手动修改此文件。

### `Manifest.toml` 条目

manifest 中有三个顶级条目，如下所示：

```toml
julia_version = "1.8.2"
manifest_format = "2.0"
project_hash = "4d9d5b552a1236d3c1171abf88d59da3aaac328a"
```

这显示了创建 manifest 的 Julia 版本、manifest 的“格式”和项目文件的哈希值，这样就可以看到 manifest 与项目文件相比什么时候过时了。

每个依赖项在 manifest 文件中都有自己的节，其内容根据依赖项添加到环境的方式而有所不同。每个依赖项的节都包含以下条目的组合：

* `uuid`: 依赖项的 [UUID](https://en.wikipedia.org/wiki/Universally_unique_identifier)，例如 `uuid = "7876af07-990d-54b4-ab0e-23690620f79a"`。
* `deps`: 一个向量，列出依赖项的依赖项，例如 `deps = ["Example", "JSON"]`。
* `version`: 版本号，例如 `version = "1.2.6"`。
* `path`: 源代码的文件路径，例如 `path = /home/user/Example`。
* `repo-url`: 源代码存储库的 URL ，例如 `repo-url = "https://github.com/JuliaLang/Example.jl.git"`。
* `repo-rev`: git 存储库的修订版, 例如一个分支 `repo-rev = "master"` 或一个提交 `repo-rev = "66607a62a83cb07ab18c0b35c038fcd62987c9b1"`。
* `git-tree-sha1`: git 源码树的哈希， 例如 `git-tree-sha1 = "ca3820cc4e66f473467d912c4b2b3ae5dc968444"`。

#### 已添加的包

当从包注册表添加包时，例如通过调用 `pkg> add Example` 或使用特定版本 `pkg> add Example@1.2`，生成的 `Manifest.toml` 条目如下所示：

```toml
[[deps.Example]]
deps = ["DependencyA", "DependencyB"]
git-tree-sha1 = "8eb7b4d4ca487caade9ba3e85932e28ce6d6e1f8"
uuid = "7876af07-990d-54b4-ab0e-23690620f79a"
version = "1.2.3"
```

请特别注意，没有 `repo-url` 存在，因为该信息包含在能够找到此包的注册表中。

#### 按分支添加的包

添加由分支指定的包时生成的 `[deps]` 节，例如 `pkg> add Example#masteror` 或 `pkg> add https://github.com/JuliaLang/Example.jl.git`，如下所示：

```toml
[[deps.Example]]
deps = ["DependencyA", "DependencyB"]
git-tree-sha1 = "54c7a512469a38312a058ec9f429e1db1f074474"
repo-rev = "master"
repo-url = "https://github.com/JuliaLang/Example.jl.git"
uuid = "7876af07-990d-54b4-ab0e-23690620f79a"
version = "1.2.4"
```

请注意，我们正在跟踪的分支 (`master`) 和远程存储库 url (`"https://github.com/JuliaLang/Example.jl.git"`) 都存储在 manifest 中。

#### 按提交添加的包

添加由提交指定的包时生成的 `[deps]` 节，例如 `pkg> add Example#cf6ba6cc0be0bb5f56840188563579d67048be34`，如下所示：

```toml
[[deps.Example]]
deps = ["DependencyA", "DependencyB"]
git-tree-sha1 = "54c7a512469a38312a058ec9f429e1db1f074474"
repo-rev = "cf6ba6cc0be0bb5f56840188563579d67048be34"
repo-url = "https://github.com/JuliaLang/Example.jl.git"
uuid = "7876af07-990d-54b4-ab0e-23690620f79a"
version = "1.2.4"
```

与跟踪分支的唯一区别是 `repo-rev` 的内容。

#### 开发包

使用 `develop` 命令添加包时生成的 `[deps]` 节，例如 `pkg> develop Example` 或 `pkg> develop /path/to/local/folder/Example`，如下所示：

```toml
[[deps.Example]]
deps = ["DependencyA", "DependencyB"]
path = "/home/user/.julia/dev/Example/"
uuid = "7876af07-990d-54b4-ab0e-23690620f79a"
version = "1.2.4"
```

请注意，包含源代码的路径，并且直接反映对该源代码树所做的更改。

#### 已固定的包

已固定的包也记录在 manifest 文件中，例如由 `pkg> add Example; pin Example` 生成的 `[deps]` 节如下所示：

```toml
[[deps.Example]]
deps = ["DependencyA", "DependencyB"]
git-tree-sha1 = "54c7a512469a38312a058ec9f429e1db1f074474"
pinned = true
uuid = "7876af07-990d-54b4-ab0e-23690620f79a"
version = "1.2.4"
```
唯一的不同是附加了 `pinned = true` 条目。

#### 多个同名的包

Julia 根据 UUID 区分包，这意味着仅靠名称不足以识别包。在同一个环境中可以有多个包具有相同的名称，但具有不同的 UUID。在这种情况下，`Manifest.toml` 文件看起来有点不同。例如，考虑您已将包 `A` 和 `B` 添加到您的环境的情况，文件 `Project.toml` 如下所示：

```toml
[deps]
A = "ead4f63c-334e-11e9-00e6-e7f0a5f21b60"
B = "edca9bc6-334e-11e9-3554-9595dbb4349c"
```

如果 `A` 现在依赖于 `B = "f41f7b98-334e-11e9-1257-49272045fb24"`，即*另一个*名为 `B` 的包，在 `Manifest.toml` 文件中将有两个不同的 `B` 包。在这种情况下，为清楚起见删除了 `git-tree-sha1` 和 `version` 字段的完整 `Manifest.toml` 文件如下所示：

```toml
[[deps.A]]
uuid = "ead4f63c-334e-11e9-00e6-e7f0a5f21b60"

    [deps.A.deps]
    B = "f41f7b98-334e-11e9-1257-49272045fb24"

[[deps.B]]
uuid = "f41f7b98-334e-11e9-1257-49272045fb24"
[[deps.B]]
uuid = "edca9bc6-334e-11e9-3554-9595dbb4349c"
```

现在，有一个包含两个 `B` 包的数组，并且 `A` 包的 `[deps]` 节已扩展为明确说明 `A` 依赖于哪个 `B` 包。

