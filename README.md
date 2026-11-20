# llama.cpp-sp-build

自动从 [ggml-org/llama.cpp](https://github.com/ggml-org/llama.cpp) 拉取源码，在 **GitHub 免费标准 Windows runner** 上以 `GGML_BACKEND_DL=ON` 编译四个后端，并把产物发布为 GitHub Release：

| 后端 | Runner | 工具链 / 依赖（自动安装） |
|---|---|---|
| CPU（多指令集变体） | `windows-2025` | LLVM 22.1.8（choco）+ Ninja + MSVC 链接 |
| Vulkan | `windows-2025` | LunarG Vulkan SDK 1.4.357.0 |
| OpenVINO | `windows-2022` | OpenVINO 2026.4.1 归档 + vcpkg(OpenCL) |
| SYCL | `windows-2022` | Intel oneAPI（DPC++ 2026.2 + oneDNN + MKL/TBB）+ Level Zero SDK |

构建配方逐条对齐 llama.cpp 官方 `release.yml`（master 分支），仅做以下改动：产物以 GGML_BACKEND_DL 独立后端 DLL 发布、增加 ALL-in-one 合并包、Release 内容与命名自定义。

## 使用

1. **一次性设置**（否则最后发布 Release 一步会 403）：
   `Settings → Actions → General → Workflow permissions` 选择 **Read and write permissions** 并保存。
2. 打开 `Actions → Build llama.cpp Windows backends (GGML_BACKEND_DL) → Run workflow`：
   - `llama_ref`：要构建的 llama.cpp 分支 / tag / 40 位 SHA（默认 `master` 最新）
   - `release_tag`：自定义 tag（留空自动生成 `b<run_number>-<短SHA>`）
   - `prerelease`：是否标记为预发布
3. 运行结束后在 Releases 页面获得：
   - `llama-<tag>-bin-win-cpu-x64.zip`（全套工具 + `ggml-cpu-*.dll` 变体）
   - `llama-<tag>-bin-win-vulkan-x64.zip` / `-openvino-` / `-sycl-`（各后端自包含包）
   - `llama-<tag>-bin-win-all-backends-x64.zip`（全部后端合一）
   - `SHA256SUMS.txt`

## 使用产物

`ggml-*.dll` 与 `llama-server.exe` 等放在同一目录即可，后端由 GGML_BACKEND_DL 在运行时自动发现加载。Vulkan/SYCL 需要相应显卡驱动；OpenVINO/SYCL 运行时 DLL 已随包附带。

## 维护说明

- 依赖版本集中在 workflow 顶部 `env`（Vulkan/OpenVINO/oneAPI/LLVM 版本与 URL）。Intel 的离线安装直链可能随版本过期，更新时请对照 llama.cpp 官方 `release.yml` 的 `windows-sycl` job。
- 全程使用免费标准 runner（公开仓库 Windows runner 不计费），无 larger-runner。
