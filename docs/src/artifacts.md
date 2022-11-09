
# [**8.** Artifacts](@id Artifacts)

`Pkg` 可以安装和管理数据容器，它不是 Julia 包。这些容器可以包含特定于平台的二进制文件、数据集、文本或任何其他类型的数据，这些数据可以方便地放置在不可变的、具有生命周期的数据存储中。这些容器（称为“Artifacts”）可以在本地创建、托管在任何地方，并在安装 Julia 包时自动下载和解包。此机制还为使用[`BinaryBuilder.jl`](https://github.com/JuliaPackaging/BinaryBuilder.jl) 进行包构建提供二进制依赖。

## 基本用法

`Pkg` artifacts 在Artifacts.toml文件中声明，该文件可以放在当前目录或包的根目录中。目前，`Pkg` 支持从 URL 下载 tar 文件（可以压缩）。以下是一个最小 `Artifacts.toml` 文件，它允许从 `github.com` 下载 `socrates.tar.gz` 文件。在此示例中，定义了一个名为 `socrates` 的 artifact。

```TOML
# a simple Artifacts.toml file
[socrates]
git-tree-sha1 = "43563e7631a7eafae1f9f8d9d332e3de44ad7239"

    [[socrates.download]]
    url = "https://github.com/staticfloat/small_bin/raw/master/socrates.tar.gz"
    sha256 = "e65d2f13f2085f2c279830e863292312a72930fee5ba3c792b14c33ce5c5cc58"
```

如果 `Artifacts.toml` 文件位于您的当前目录中，则 `socrates.tar.gz` 可以下载、解压缩，并以 `artifact"socrates"` 方式使用。由于这个 tar 压缩包包含一个 `bin` 文件夹和一个名为 `socrates` 的文本文件，我们可以按如下方式访问该文件的内容。

```julia
using Pkg.Artifacts

rootpath = artifact"socrates"
open(joinpath(rootpath, "bin", "socrates")) do file
    println(read(file, String))
end
```

如果您有一个可通过 `url` 访问的现有 tar 压缩包，也可以以这种方式访问​​它。要创建 `Artifacts.toml` 您必须计算两个哈希值：下载文件的 `sha256` 哈希值和解压内容的 `git-tree-sha1` 哈希值。这些可以计算如下：

```julia
using Tar, Inflate, SHA

filename = "socrates.tar.gz"
println("sha256: ", bytes2hex(open(sha256, filename)))
println("git-tree-sha1: ", Tar.tree_hash(IOBuffer(inflate_gzip(filename))))
```

要从您创建的包中访问此 artifact，请将 `Artifacts.toml` 放在包的根目录中，和 `Project.toml` 相邻。然后，确保在您的 `deps` 节中添加了 `Pkg`，以及在 `compat` 节中设置 `julia = "1.3"` 或更高版本。

## `Artifacts.toml` 文件

`Pkg` 提供了用于使用 artifacts 的 API，以及用于记录软件包中 artifacts 使用情况的 TOML 文件格式，并在软件包安装时自动下载 artifacts。Artifacts 始终可以通过内容哈希来引用，但通常通过绑定到项目源树中 `Artifacts.toml` 文件的内容哈希的名称来访问。

!!! note
    可以使用替代名称 `JuliaArtifacts.toml`，类似于分别使用 `JuliaProject.toml` 和 `JuliaManifest.toml` 代替 `Project.toml` 和 `Manifest.toml`。
    It is possible to use the alternate name `JuliaArtifacts.toml`, similar
    to how it is possible to use `JuliaProject.toml` and `JuliaManifest.toml`
    instead of `Project.toml` and `Manifest.toml`, respectively.

此处显示了一个 `Artifacts.toml` 文件示例：

```TOML
# Example Artifacts.toml file
[socrates]
git-tree-sha1 = "43563e7631a7eafae1f9f8d9d332e3de44ad7239"
lazy = true

    [[socrates.download]]
    url = "https://github.com/staticfloat/small_bin/raw/master/socrates.tar.gz"
    sha256 = "e65d2f13f2085f2c279830e863292312a72930fee5ba3c792b14c33ce5c5cc58"

    [[socrates.download]]
    url = "https://github.com/staticfloat/small_bin/raw/master/socrates.tar.bz2"
    sha256 = "13fc17b97be41763b02cbb80e9d048302cec3bd3d446c2ed6e8210bddcd3ac76"

[[c_simple]]
arch = "x86_64"
git-tree-sha1 = "4bdf4556050cb55b67b211d4e78009aaec378cbc"
libc = "musl"
os = "linux"

    [[c_simple.download]]
    sha256 = "411d6befd49942826ea1e59041bddf7dbb72fb871bb03165bf4e164b13ab5130"
    url = "https://github.com/JuliaBinaryWrappers/c_simple_jll.jl/releases/download/c_simple+v1.2.3+0/c_simple.v1.2.3.x86_64-linux-musl.tar.gz"

[[c_simple]]
arch = "x86_64"
git-tree-sha1 = "51264dbc770cd38aeb15f93536c29dc38c727e4c"
os = "macos"

    [[c_simple.download]]
    sha256 = "6c17d9e1dc95ba86ec7462637824afe7a25b8509cc51453f0eb86eda03ed4dc3"
    url = "https://github.com/JuliaBinaryWrappers/c_simple_jll.jl/releases/download/c_simple+v1.2.3+0/c_simple.v1.2.3.x86_64-apple-darwin14.tar.gz"

[processed_output]
git-tree-sha1 = "1c223e66f1a8e0fae1f9fcb9d3f2e3ce48a82200"
```

此 `Artifacts.toml` 绑定了三个 artifacts：一个命名为 `socrates`，一个命名为 `c_simple`，一个命名为 `processed_output`。artifacts 唯一需要的信息是它的 `git-tree-sha1`。因为 artifacts 仅通过其内容哈希寻址，所以 `Artifacts.toml` 文件的目的是提供有关这些 artifacts 的元数据，例如绑定人类可读的名称到内容哈希，提供有关可以从何处下载 artifacts 的信息，甚至绑定单个名称到多个哈希，由特定于平台的约束（例如操作系统或 `libgfortran` 的版本）键控(keyed)。

## Artifact 类型和属性

在上面的示例中，`socrates` artifact 展示了具有多个下载位置的独立于平台的 artifact。下载和安装 `socrates` artifact 时，将按顺序尝试 URL 直到成功。该`socrates` artifact 被标记为 `lazy`，这意味着它不会在安装包含的包时自动下载，而是在包第一次尝试使用它时按需下载。

`c_simple` artifact 展示了一个依赖于平台的 artifact，其中 `c_simple` 数组中的每个条目都包含键，键帮助调用包根据主机的详细信息选择适当的下载地址。请注意，每个 artifact 都包含对应下载条目的 `git-tree-sha1` 和 `sha256`。这是为了确保下载的 tar 压缩包在尝试解压缩之前是安全的，并强制所有 tar 压缩包必须扩展为同样的整体 git 树哈希。

`processed_output` artifact不包含 `download` 节，因此无法被安装。像这样的 artifact 是先前运行的代码的结果，生成一个新的 artifact 并将生成的哈希绑定到该项目中的名称上。

## 使用 Artifacts

可以使用从 `Pkg.Artifacts` 命名空间公开的便捷 API 来操作 artifact 。作为一个激励性的例子，让我们想象我们正在编写一个需要加载 [Iris 机器学习数据集](https://archive.ics.uci.edu/ml/datasets/iris)的包。虽然我们可以在构建步骤中将数据集下载到包目录中，并且目前许多包都这样做，但这有一些明显的缺点：

* 首先，它修改了包目录，使包安装变得具有状态了，这是我们要避免的。将来，我们希望能够以完全只读的方式安装包，而不是在安装后能够自行修改。

* 其次，下载的数据不会在包的不同版本之间共享。如果我们安装了三个不同版本的包以供不同项目使用，那么我们需要三个不同的数据副本，即使这些版本之间没有更改。此外，每次我们升级或降级软件包时，除非我们做了一些聪明的事情（而且可能很脆弱），否则我们必须再次下载数据。

对于 artifacts，我们将检查 `iris` artifacts 是否已经存在于磁盘上，只有不存在时我们才会下载并安装它，之后我们可以将结果绑定到我们的 `Artifacts.toml` 文件中：

```julia
using Pkg.Artifacts

# This is the path to the Artifacts.toml we will manipulate
artifact_toml = joinpath(@__DIR__, "Artifacts.toml")

# Query the `Artifacts.toml` file for the hash bound to the name "iris"
# (returns `nothing` if no such binding exists)
iris_hash = artifact_hash("iris", artifact_toml)

# If the name was not bound, or the hash it was bound to does not exist, create it!
if iris_hash == nothing || !artifact_exists(iris_hash)
    # create_artifact() returns the content-hash of the artifact directory once we're finished creating it
    iris_hash = create_artifact() do artifact_dir
        # We create the artifact by simply downloading a few files into the new artifact directory
        iris_url_base = "https://archive.ics.uci.edu/ml/machine-learning-databases/iris"
        download("$(iris_url_base)/iris.data", joinpath(artifact_dir, "iris.csv"))
        download("$(iris_url_base)/bezdekIris.data", joinpath(artifact_dir, "bezdekIris.csv"))
        download("$(iris_url_base)/iris.names", joinpath(artifact_dir, "iris.names"))
    end

    # Now bind that hash within our `Artifacts.toml`.  `force = true` means that if it already exists,
    # just overwrite with the new content-hash.  Unless the source files change, we do not expect
    # the content hash to change, so this should not cause unnecessary version control churn.
    bind_artifact!(artifact_toml, "iris", iris_hash)
end

# Get the path of the iris dataset, either newly created or previously generated.
# this should be something like `~/.julia/artifacts/dbd04e28be047a54fbe9bf67e934be5b5e0d357a`
iris_dataset_path = artifact_path(iris_hash)
```

对于使用先前绑定的 artifacts 的特定用例，我们有一个速记符号 `artifact"name"`，它将自动搜索当前包中的 `Artifacts.toml` 文件，按名称查找给定的 artifacts，如果尚未安装，则安装它，然后返回该给定 artifacts 的路径。下面给出了这个速记符号的一个例子：

```julia
using Pkg.Artifacts

# For this to work, an `Artifacts.toml` file must be in the current working directory
# (or in the root of the current package) and must define a mapping for the "iris"
# artifact.  If it does not exist on-disk, it will be downloaded.
iris_dataset_path = artifact"iris"
```

## `Pkg.Artifacts` API

`Artifacts` API 分为三个级别：哈希感知函数、名称感知函数和实用函数。
The `Artifacts` API is broken up into three levels: hash-aware functions, name-aware functions and utility functions.

* **哈希感知**函数只处理内容哈希。这些方法允许您查询 artifact 是否存在、其路径是什么、验证 artifact 是否满足其在磁盘上的内容哈希等。哈希感知函数包括：`artifact_exists()`、`artifact_path()`、`remove_artifact()`、`verify_artifact()`和 `archive_artifact()`。请注意，通常您不应该使用 `remove_artifact()`，而应该使用 `Pkg.gc()` 来清理安装的 artifact。

* **名称感知**函数处理 `Artifacts.toml` 文件中的绑定名称，因此，通常需要 `Artifacts.toml` 文件路径和 artifact 名称。名称感知函数包括：`artifact_meta()`、`artifact_hash()`、`bind_artifact!()`、`unbind_artifact!()`、`download_artifact()` 和 `ensure_artifact_installed()`。

* **实用**函数处理 artifact 生命周期的各种方面，例如 `create_artifact()`、`ensure_all_artifacts_installed()`，甚至是 `@artifact_str` 字符串宏。

有关文档字符串和方法的完整列表，请参阅 [Artifacts Reference](@ref)章节。

## 覆盖 artifact 位置

有时需要能够覆盖 artifact 的位置和内容。一个常见的例子是计算环境，其中必须使用某些版本的二进制依赖项，而不管包是使用哪个版本的依赖项发布的。虽然典型的 Julia 配置会下载、解包并链接到通用库，但系统管理员可能希望禁用此功能，而使用已安装在本地计算机上的库。`Pkg` 通过在 `artifacts` depot 目录中放置一个 `Overrides.toml` 文件启用对此功能的支持（例如对于默认用户仓库是 `~/.julia/artifacts/Overrides.toml` 文件），它可以通过内容哈希，或包的 `UUID` 和绑定到 artifact 的名称覆盖 artifact 的位置。此外，目标位置可以是绝对路径，也可以是 artifact 内容哈希。这允许系统管理员创建他们自己的 artifacts，然后他们可以通过覆盖其他包来使用新 artifact。

```TOML
# Override single hash to an absolute path
78f35e74ff113f02274ce60dab6e92b4546ef806 = "/path/to/replacement"

# Override single hash to new artifact content-hash
683942669b4639019be7631caa28c38f3e1924fe = "d826e316b6c0d29d9ad0875af6ca63bf67ed38c3"

# Override package bindings by specifying the package UUID and bound artifact name
# For demonstration purposes we assume this package is called `Foo`
[d57dbccd-ca19-4d82-b9b8-9d660942965b]
libfoo = "/path/to/libfoo"
libbar = "683942669b4639019be7631caa28c38f3e1924fe"
```

由于`Pkg` depot 的分层特性，多个 `Overrides.toml` 文件可能同时生效。这允许“内部” `Overrides.toml` 文件覆盖放在“外部” `Overrides.toml` 文件中的覆盖规则。要删除覆盖并重新启用 artifact 的默认位置逻辑，请将条目映射插入到空字符串中：

```TOML
78f35e74ff113f02274ce60dab6e92b4546ef806 = "/path/to/new/replacement"
683942669b4639019be7631caa28c38f3e1924fe = ""

[d57dbccd-ca19-4d82-b9b8-9d660942965b]
libfoo = ""
```

如果上面给出的两个 `Overrides.toml` 片段相互叠加，最终结果将映射内容哈希 `78f35e74ff113f02274ce60dab6e92b4546ef806` 到 `"/path/to/new/replacement"`，并映射 `Foo.libbar` 到由内容哈希标识的 artifact `683942669b4639019be7631caa28c38f3e1924fe`。请注意，虽然该哈希之前已被覆盖，但它不再被覆盖，因此 `Foo.libbar` 将直接查看 `~/.julia/artifacts/683942669b4639019be7631caa28c38f3e1924fe` 位置。

大多数受覆盖影响的方法可以通过设置它们的 `honor_overrides=false` 关键字参数来忽略覆盖。要使基于 UUID/名称 的覆盖起作用，`Artifacts.toml` 文件必须在知道加载包的 UUID 的情况下被加载。这是由 `artifacts""` 字符串宏自动推导出来的，但是，如果您出于某种原因在包中手动使用 `Pkg.Artifacts` API 并且希望尊重覆盖，则必须通过关键字参数将包的 UUID 提供给 API 调用，类似于 `artifact_meta()` 和 `ensure_artifact_installed()` 使用 `pkg_uuid` 关键字参数。

## 扩展平台选择

!!! compat "Julia 1.7"
    Pkg 的扩展平台选择至少需要 Julia 1.7，并且被认为是实验性的。

Julia 1.6 中的新增功能，`Platform` 对象可以应用扩展属性，允许使用诸如 CUDA 驱动程序版本兼容性、微架构兼容性、julia 版本兼容性等扩展属性标记 artifacts！请注意，此功能被认为是实验性的，将来可能会更改。如果您作为包开发人员发现自己需要此功能，请与我们联系，以便它可以为整个生态系统的利益而发展。为了在 `Pkg.add()` 时支持 artifact 选择，`Pkg` 将运行特殊命名的文件 `<project_root>/.pkg/select_artifacts.jl`，传递当前平台三元组作为第一个参数。这个 artifact 选择脚本应该打印一个 `TOML` - 表示此包根据给定平台需要的 artifacts 的序列化字典，如果给定平台三元组未明确提供平台功能，则根据需要对系统进行任何检查，以自动检测平台功能。字典的格式应该与从 `Artifacts.select_downloadable_artifacts()` 返回的匹配，实际上大多数包应该简单地用一个增强的 `Platform` 对象调用此函数。artifact 选择 hook 的定义的示例，可能如下所示，分为两个文件：

```julia
# .pkg/platform_augmentation.jl
using Libdl, Base.BinaryPlatforms
function augment_platform!(p::Platform)
    # If this platform object already has a `cuda` tag set, don't augment
    if haskey(p, "cuda")
        return p
    end

    # Open libcuda explicitly, so it gets `dlclose()`'ed after we're done
    dlopen("libcuda") do lib
        # find symbol to ask for driver version; if we can't find it, just silently continue
        cuDriverGetVersion = dlsym(lib, "cuDriverGetVersion"; throw_error=false)
        if cuDriverGetVersion !== nothing
            # Interrogate CUDA driver for driver version:
            driverVersion = Ref{Cint}()
            ccall(cuDriverGetVersion, UInt32, (Ptr{Cint},), driverVersion)

            # Store only the major version
            p["cuda"] = div(driverVersion, 1000)
        end
    end

    # Return possibly-altered `Platform` object
    return p
end
```

```julia
using TOML, Artifacts, Base.BinaryPlatforms
include("./platform_augmentation.jl")
artifacts_toml = joinpath(dirname(@__DIR__), "Artifacts.toml")

# Get "target triplet" from ARGS, if given (defaulting to the host triplet otherwise)
target_triplet = get(ARGS, 1, Base.BinaryPlatforms.host_triplet())

# Augment this platform object with any special tags we require
platform = augment_platform!(HostPlatform(parse(Platform, target_triplet)))

# Select all downloadable artifacts that match that platform
artifacts = select_downloadable_artifacts(artifacts_toml; platform)

# Output the result to `stdout` as a TOML dictionary
TOML.print(stdout, artifacts)
```

在此 hook 定义中，我们的 platform_augmentation 例程打开一个系统库(`libcuda`)，搜索 CUDA 驱动的版本符号，然后将版本号中的主版本号嵌入到我们要增强的 `Platform` 对象的 `cuda` 属性中。虽然实际尝试关闭已加载的库对于此代码而言并不重要（因为它很可能会在包操作完成后，立即被 CUDA 包再次打开），但最佳实践是使 hook 尽可能轻量级和透明，因为它们将来可能会被其他 Pkg 实用程序使用。在您自己的包中，您也应该在使用 `@artifact_str` 宏时使用增强的平台对象，如下所示：

```julia
include("../.pkg/platform_augmentation.jl")

function __init__()
    p = augment_platform!(HostPlatform())
    global my_artifact_dir = @artifact_str("MyArtifact", p)
end
```

这可确保您的代码使用与 Pkg 尝试安装的 artifact 相同的 artifact。

Artifact 选择 hook 仅允许使用`Base`、`Artifacts`、`Libdl` 和 `TOML` 包。不允许使用任何其他标准库，也不允许使用任何包（包括他们所属的包）。

