# Celer Conf

**Example configuration repo** for [Celer](https://github.com/celer-pkg/celer). It defines the build tools, platforms, and projects a workspace can use; `celer init` clones it as `conf/`.

> This is the sample used by `celer init` and Celer's tests. Fork it and replace the contents with your own platforms and projects.

## Quick start

```bash
celer init --url=https://github.com/celer-pkg/celer-conf.git
celer configure --platform=x86_64-linux-ubuntu-22.04-gcc-11.5.0
celer configure --project=my_first_project
celer install sqlite3@3.49.0      # or: celer deploy
```

`celer update --conf-repo` pulls the latest conf. To use your own repo, pass its URL to `celer init` or set `conf_repo` in `celer.toml`.

## Layout

```
conf
├── buildtools/<arch>-<os>.toml                  # extra build tools for the host
├── platforms/<platform-name>.toml               # toolchain + rootfs per target
└── projects/<project-name>.toml                 # dependencies + build environment
    └── <project-name>/<lib>/<version>/port.toml # optional project-local ports
```

The file name without `.toml` is the name used on the command line. `platform` and `project` are independent: `--project` applies the project's `target_platform`, and a later `--platform` overrides it. `native` is a reserved platform meaning the local toolchain. Scaffold entries with `celer create --platform/--project`.

## Build tools

Files here extend Celer's built-in registry — the same `name` + `version` replaces the built-in entry.

```toml
# buildtools/x86_64-linux.toml
[[build_tools]]
  name    = "doxygen"
  version = "1.14.0"
  default = true
  url     = "https://www.doxygen.nl/files/doxygen-1.14.0.linux.bin.tar.gz"
  paths   = ["doxygen-1.14.0/bin"]
```

Required: `name`, `version`, `url`. Optional: `default`, `sha256`, `archive`, `paths`, `vars`, `envs`. Python build dependencies use a `python3:` prefix, e.g. `"python3:mako@1.3.10"`. See [Build Tools](https://github.com/celer-pkg/celer/blob/master/docs/en-US/article_build_tools.md).

## Platforms

```toml
# platforms/x86_64-linux-ubuntu-22.04-gcc-11.5.0.toml
[toolchain]
  url              = "https://...gcc-ubuntu-11.5.0-x86_64-x86_64-linux-gnu.tar.xz"
  sha256           = "3a7ef1c4..."
  path             = "gcc-ubuntu-11.5.0-x86_64-x86_64-linux-gnu/bin"
  name             = "gcc"
  version          = "11.5.0"
  system_name      = "linux"
  system_processor = "x86_64"
  host             = "x86_64-linux-gnu"
  crosstool_prefix = "x86_64-linux-gnu-"
  cc               = "x86_64-linux-gnu-gcc"
  cxx              = "x86_64-linux-gnu-g++"

[rootfs]
  url             = "https://...ubuntu-base-22.04.5-base-amd64.tar.xz"
  sha256          = "08442eca..."
  path            = "ubuntu-base-22.04.5-base-amd64"
  pkg_config_path = ["usr/lib/x86_64-linux-gnu/pkgconfig"]
```

- **Mandatory Fields**: `url`, `sha256` (skipped for `file:///`), `name` (`gcc`/`clang`/`clang-cl`/`msvc`/`qcc`), `version`, `system_name`, `system_processor`, `host`, `crosstool_prefix`, `cc`, `cxx`.   
- **Optional Fields**: `cflags`/`cxxflags`/`ldflags`, `cmake_vars`, `cmake_policy_version_minimum`, `envs`, auxiliary tools. Rootfs requires `url`, `sha256`, `path`, `pkg_config_path`.

Included platforms:

- **x86_64-linux:** `x86_64-linux-ubuntu-20.04-gcc-9.5.0`, `x86_64-linux-ubuntu-22.04-gcc-11.5.0`, `x86_64-linux-ubuntu-22.04-clang-21.1.4`
- **aarch64-linux:** `aarch64-linux-ubuntu-18.04-gcc-7.5.0`, `aarch64-linux-ubuntu-22.04-gcc-11.5.0`
- **x86_64-windows:** `x86_64-windows-clang-21.1.4`, `x86_64-windows-{msvc,clang-cl}-{community,enterprise}-14` (the MSVC entries point at a locally installed Visual Studio via `file:///`)

See [Platform Configuration](https://github.com/celer-pkg/celer/blob/master/docs/en-US/article_platform.md).

## Projects

```toml
# projects/my_first_project.toml
ports_baseline = "master"        # required: ports repo commit hash or branch

ports  = ["sqlite3@3.49.0"]
vars   = ["VAR1=value1"]         # CMake variables
envs   = ["ENV_VAR1=hello"]      # environment variables
macros = ["MICRO_VAR1=value1"]   # C/C++ macros

[python]                         # optional, must stay last in the file
  version = "3.11.5"
```

`ports_baseline` is required, everything else optional. Ports can be overridden per project under `projects/<project>/<lib>/<version>/port.toml` (or `projects/<project>/ports/...` for imported third-party ports); those win over the global `ports/` repo. Without a project, Celer uses the built-in `default` project. See [Project Configuration](https://github.com/celer-pkg/celer/blob/master/docs/en-US/article_project.md).

## Notes

- Toolchains and rootfs archives live in the [`resource` release](https://github.com/celer-pkg/celer-conf/releases/tag/resource) and are pinned by `sha256`. If you fork, update `url` and `sha256` together.
- Conf is optional: without it Celer uses the local toolchain (Visual Studio on Windows, installed gcc on Linux).
- Unknown keys are silently ignored, so a typo such as `micros` does nothing. Copy field names from the docs.
