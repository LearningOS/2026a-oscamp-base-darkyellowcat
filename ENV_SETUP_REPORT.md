# LearningOS Rust 训练营环境部署报告

- 部署时间：2026-10-23（Asia/Shanghai）
- 主机：Ubuntu 24.04.3 LTS，x86_64，2 vCPU，内存 3.4 GiB，Swap 0，根分区可用约 28 GiB
- 仓库提交：`main`，`0841b51`

## 已完成配置

- 创建普通开发用户 `oscamp`（UID/GID 1001），账户无 sudo 权限，密码保持锁定；未修改 SSH 端口、root 登录方式、认证策略或已有密钥。
- 项目路径：`/home/oscamp/projects/preparing-2026a-oscamp-base-darkyellowcat`
- 已安装/确认：`build-essential`、`git 2.43.0`、`curl`、`ca-certificates`。
- 新安装 RISC-V 依赖：
  - `gcc-riscv64-linux-gnu` 4:13.2.0-7ubuntu1（实际 GCC 13.3.0）
  - `qemu-user` 1:8.2.2+ds-0ubuntu1.18（`qemu-riscv64` 8.2.2）
  - RISC-V sysroot：`/usr/riscv64-linux-gnu`
- 通过 Ubuntu 官方 `rustup` 包安装 Rustup 1.26.0；在 `oscamp` 家目录安装仓库要求的 Rust 1.98.1，包含目标 `riscv64gc-unknown-linux-gnu` 和 `rust-analyzer` 组件。
- 为降低下载失败概率，在 `/home/oscamp/.cargo/config.toml` 配置了仅对 `oscamp` 用户生效的中科大 crates.io 镜像；仓库自身的 `.cargo/config.toml` 未修改。
- 未创建 Swap；当前可用内存约 1.7 GiB，构建时使用 `CARGO_BUILD_JOBS=2`。

## 已验证成功

```text
rustc 1.98.1 (48a229cea 2026-09-01)
cargo 1.98.1 (797e8a9bc 2026-08-05)
riscv64gc-unknown-linux-gnu target installed
rust-analyzer-x86_64-unknown-linux-gnu (installed)
```

RISC-V 用户态链路已用交叉编译的 ELF 小程序验证：`riscv64-linux-gnu-gcc` 编译成功，`qemu-riscv64 -L /usr/riscv64-linux-gnu` 正常输出 `riscv-ok`。

仓库结构已检查：`README.md`、`Cargo.toml`、`rust-toolchain.toml`、`.cargo/config.toml`、`scripts/setup_riscv64.sh`、`check.sh` 均存在；`exercises.toml` 包含全部 24 项练习。

## 构建与测试结果

- `cargo build -p oscamp-cli`：成功。
- `cargo run -p oscamp-cli -- list`：成功，显示 24 项练习。
- `cargo test -p thread_spawn`：测试进程正常启动；退出码 101，10 项测试均因题目中的 `todo!()`/`not yet implemented` 失败，属于预期的未完成练习，不是工具链故障。
- `cargo test -p stack_coroutine --target riscv64gc-unknown-linux-gnu -- --nocapture`：交叉编译、QEMU runner 和动态 sysroot 均工作；退出码 101，3 项测试因题目未完成而失败。
- `oscamp watch`：在伪终端中成功启动并扫描 24 项；非交互终端会因无法启用 raw mode 而退出，这是终端条件限制。VS Code Remote-SSH 终端可直接运行。
- `bash check.sh`：首个未完成练习按预期失败后，由脚本 `set -e` 与 `((FAIL++))` 的返回状态提前退出，未进入完整汇总；这是仓库检查脚本行为，不是环境安装失败。未修改该脚本。

## 重要仓库现状

仓库的 `check.sh` 引用了 `.cargo/run_riscv64.sh`，但当前提交未包含该文件；仓库 `.cargo/config.toml` 的直接 QEMU runner 已验证可用。未添加或修改仓库脚本，以免改变题目/评分逻辑。

## 系统变更与影响

- 新增本地账号 `/home/oscamp`、Rust/Cargo 缓存、项目目录和本报告。
- 新安装上述 RISC-V/Rust 软件包；未删除文件、未修改题目答案、测试或评分逻辑，未执行 `git push`。
- 安装软件包时 Ubuntu `needrestart` 自动重启了 `ces-uniagent.service` 一次；部署过程中未主动停止其他业务服务，当前该服务状态为 `active`。未修改 1Panel、Docker、SSH 或网络配置。
- 为当前 root Remote-SSH 会话加入该项目单路径 Git `safe.directory`，以避免仓库归属为 `oscamp` 时的 dubious ownership 拒绝。
- 未配置 GitHub 推送凭据；没有复制或生成任何 SSH 私钥/高权限令牌。需要推送时，建议在 VS Code/SSH 客户端使用你自己的 GitHub SSH key，或使用最小权限 fine-grained token，并自行配置 Git credential helper。

## 后续使用

```bash
su - oscamp
cd /home/oscamp/projects/preparing-2026a-oscamp-base-darkyellowcat
cargo build -p oscamp-cli
cargo run -p oscamp-cli -- list
cargo test -p thread_spawn
# 在 VS Code Remote-SSH 的交互终端中运行：
cargo run -p oscamp-cli -- watch
```

本报告未提交到 Git；完成练习后可按需要保留或删除。
