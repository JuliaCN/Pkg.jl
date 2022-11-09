
# **7.** 注册表

注册表包含有关软件包的信息，例如可用的版本和依赖项，以及可以下载它们的位置。如果没有安装其他注册表，则 [`General`注册表](https://github.com/JuliaRegistries/General) 默认被自动安装。

## 管理注册表

可以从 Pkg REPL 或使用函数式 API 添加、移除和更新注册表。在本节中，我们将描述 REPL 接口。注册表 API 记录在[Registry API Reference](@ref) 部分。

### 添加注册表

可以在 Pkg REPL 中使用 `registry add` 命令添加自定义注册表。这通常通过一个指向注册表的 URL 来完成。

如果安装自定义注册表导致 `General` 注册表没有自动安装，手动指定它也很容易：它会被自动添加。在这种情况下，可以很容易地添加 `General`：

```julia-repl
pkg> registry add General
```

现在所有 `General` 中注册的包都是可用的，例如从 General 中添加包。要查看当前安装了哪些注册表，您可以使用 `registry status` （或 `registry st`）命令：

```julia-repl
pkg> registry st
Registry Status
 [23338594] General (https://github.com/JuliaRegistries/General.git)
```

注册表总是添加到 user depot，位于 `DEPOT_PATH` 的第一个条目（参见[词汇表](@ref Glossary)章节）。

!!! note "来自包服务器的注册表"
    包服务器可能会宣传额外可用的包注册表。当 Pkg 运行在一个使用 `JULIA_PKG_SERVER` 环境变量配置自定义包服务器的，干净的 Julia depot 中时（例如在一个新安装之后），所有可用的注册表都会被自动添加。如果 depot 已经安装了一些注册表（例如 General），额外的注册表可以使用无参数 `registry add` 命令轻松安装。

### 移除注册表

可以使用 `registry remove`（或 `registry rm`）命令移除注册表。这里我们移除 `General` 注册表：

```julia-repl
pkg> registry rm General
  Removing registry `General` from ~/.julia/registries/General

pkg> registry st
Registry Status
  (no registries found)
```

如果有多个名为 `General` 的已安装注册表，您必须使用 `uuid` 消除歧义，就像在操作包时一样，例如：

```julia-repl
pkg> registry rm General=23338594-aafe-5451-b93e-139f81909106
  Removing registry `General` from ~/.julia/registries/General
```

### 更新注册表

`registry update`（或`registry up`）命令可用于更新注册表。这里我们更新 `General` 注册表：

```julia-repl
pkg> registry up General
  Updating registry at `~/.julia/registries/General`
  Updating git-repo `https://github.com/JuliaRegistries/General`
```

要更新所有已安装的注册表，只需执行以下操作：

```julia-repl
pkg> registry up
  Updating registry at `~/.julia/registries/General`
  Updating git-repo `https://github.com/JuliaRegistries/General`
```

执行包操作时，注册表会在每个会话中自动更新一次，因此很少需要手动执行。

### 创建和维护注册表

Pkg 只为注册表提供客户端工具，而不是创建或维护它们的功能。但是，[Registrator.jl](https://github.com/JuliaRegistries/Registrator.jl) 和[LocalRegistry.jl](https://github.com/GunnarFarneback/LocalRegistry.jl) 提供了创建和更新注册表的方法，而 [RegistryCI.jl](https://github.com/JuliaRegistries/RegistryCI.jl) 提供了用于维护注册表的自动化测试和合并功能。

