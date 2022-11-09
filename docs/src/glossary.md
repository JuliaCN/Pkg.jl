
# [**9.** 词汇表](@id Glossary)

**Project（项目）:** 具有标准布局的源代码树，包括存放主体 Julia 代码的 `src` 目录、测试项目的 `test` 目录、存放文档文件的 `docs` 目录，以及可选的存放构建脚本及其输出的`deps` 目录。一个项目通常也会有一个项目文件，并且可以选择有一个 manifest 文件：

- **Project file（项目文件）:** 项目根目录中的文件，名为 `Project.toml`（或 `JuliaProject.toml`），描述有关项目的元数据，包括其名称、UUID（用于包）、作者、许可证以及它所依赖的包和库的名称和 UUID上。

- **Manifest file（manifest 文件）:** 项目根目录中的一个文件，名为 `Manifest.toml`（或 `JuliaManifest.toml`），描述了项目使用的每个包和库的完整依赖关系图和确切版本。

**Package（包）:** 提供可重用功能的项目，其他 Julia 项目可以通过 `import X` 或 `using X` 使用。一个包应该有一个项目文件，其中包含一个 `uuid` 条目提供此包的 UUID 。此 UUID 用于在依赖它的项目中的标识此包。

!!! note
    由于遗留原因，可以从 REPL 或脚本的顶层加载没有项目文件或 UUID 的包。但是，无法从带有项目文件或 UUID 的项目中加载没有项目文件的包。一旦你从项目文件加载后，一切都需要项目文件和 UUID。

**Application（应用程序）:** 提供具有独立功能的项目，此项目不打算被其他 Julia 项目重用。例如，Web 应用程序或命令行实用程序，或科学论文随附的模拟/分析代码。应用程序也可能有一个 UUID ，但是并不需要它。应用程序还可以为其所依赖的包提供全局配置选项。另一方面，包可能不提供全局配置，因为这可能与主应用程序的配置冲突。

!!! note
    **Projects _vs._ Packages _vs._ Applications:**

    1. **Project** 是一个概括行的术语：packages 和 applications 是 projects 的种类。
    2. **Packages** 应该有 UUID, applications 也可以有 UUID，但是并不需要它。
    3. **Applications** 可以提供全局配置, 而 packages 不能。

**Environment（环境）:** 项目文件提供的顶级名称映射与依赖关系图的组合，以及从包到 manifest 文件提供的入口点的映射。有关更多详细信息，请参阅有关代码加载的手册部分。

- **Explicit environment（显式环境）:** 形式上的环境指，显式项目文件和可选的对应 manifest 文件同在一个目录中。如果 manifest 文件不存在，则隐含的依赖图和位置映射都为空。

- **Implicit environment（隐式环境）:** 作为目录提供的环境（没有项目文件或 menifest 文件），其中包含入口点形式为 `X.jl`、`X.jl/src/X.jl` 或 `X/src/X.jl` 的包。这些入口点隐含了顶级名称映射。这些包目录中存在的项目文件暗示了依赖关系图，例如 `X.jl/Project.toml` 或 `X/Project.toml`。如果有项目文件的话，`X`包的依赖项是对应项目文件中的依赖项。位置映射由入口点本身隐含。

**Registry（注册表）:** 具有标准布局的源码树，记录一组已注册包的元数据、可用的版本标签、以及哪些版本的包相互兼容或不兼容。注册表按包名称和 UUID 进行索引，并且每个注册包都有一个目录，提供它的以下元数据：

- name – e.g. `DataFrames`
- UUID – e.g. `a93c6f00-e57d-5684-b7b6-d8193f3e46c0`
- repository – e.g. `https://github.com/JuliaData/DataFrames.jl.git`
- versions – 所有已注册版本标签的列表

对于包的每个注册版本，都提供以下信息：

- 它的语义版本号 – e.g. `v1.2.3`
- 它的 git tree SHA-1 哈希 – e.g. `7ffb18ea3245ef98e368b02b81e8a86543a11103`
- 从名称到依赖项的 UUID 的映射
- 它与其他软件包的哪些版本兼容/不兼容

依赖项和兼容性使用已压缩但人类可读的包版本区间格式存储。

**Depot:** 系统上的一个目录，其中包含各种与包相关的资源，包括：

- `environments`: 共享命名环境 (e.g. `v1.0`, `devtools`)
- `clones`: 包存储库的裸克隆
- `compiled`: 缓存的已编译包镜像 (`.ji` files)
- `config`: 全局配置文件 (e.g. `startup.jl`)
- `dev`: 包开发的默认目录
- `logs`: 日志文件 (e.g. `manifest_usage.toml`, `repl_history.jl`)
- `packages`: 已安装的包版本
- `registries`: 注册表的克隆 (e.g. `General`)

**Load path:** 一个 environments 栈，在其中搜索包标识符、包依赖项和入口点。加载路径在 Julia 中由 `LOAD_PATH` 全局变量控制，该变量在启动时根据环境变量 `JULIA_LOAD_PATH` 的值填充。第一个条目是您的主要环境，通常是当前项目，而后面的条目提供了可能希望从 REPL 或顶级脚本中使用的附加包。

**Depot path:** 一个 depot 位置栈，包含包管理器以及 Julia 的代码加载机制，在其中查找注册表、已安装包、命名环境、存储库克隆、缓存的编译包镜像和配置文件。depot path 由 Julia 的 `DEPOT_PATH` 全局变量控制，该变量在启动时根据 `JULIA_DEPOT_PATH` 环境变量的值设置。第一个条目是“user depot”，当前用户应当具有写入权和所有权。user depot 位于：克隆的注册表，安装的新版本包，创建和更新的命名环境，克隆的包存储库，保存的新编译的包镜像文件，写入的日志文件，默认检出（checked out）的开发包，以及保存的全局配置数据。depot path 中的后续条目被视为只读，适用于由系统管理员安装和管理的注册表、包等。

