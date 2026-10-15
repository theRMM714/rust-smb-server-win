# rust-smb-server-win

本仓是 [paltaio/rust-smb-server](https://github.com/paltaio/rust-smb-server) 的 **fork**（上游 MIT，版权归 Palta.io）。
上游是纯 Rust 的 SMB2/3 文件共享服务库（crate `smb-server`）；本 fork 只做一件事：
**产出 Windows 上可用的 SMB 共享 helper**。

## 它现在用来做什么

[Solomni](https://github.com/theRMM714/Solomni) 的**虚拟机档**在 Windows 宿主上用它做实时共享后端：
宿主把要共享的目录交给 `smbd.exe`，它以**匿名**共享暴露在 `127.0.0.1:<port>`；guest 经 QEMU `guestfwd` 隧道
把 `//10.0.2.100/<share>` 挂成 cifs（`mount -t cifs -o guest`）。

本仓 CI 构建并发布这个 helper；Solomni **只按 URL 下载 Release 资产**，不把本仓的提交史带进产品仓。

## 与上游的差异

全部改动集中在一次提交（Windows port + smbd helper）：

- 新增 binary `smbd`（`src/bin/smbd.rs`）：`smbd --listen <ip:port> --share <名>=<目录>[:ro] …`
  - 只支持**匿名**共享（guest 侧 `-o guest`），没有用户 / 密码；
  - 绑定成功后向 stdout 打一行 `LISTENING <ip:port>` 并 flush，随后服务到终止信号；
  - NetBIOS 名固定 `SOLOMNI`。
- Windows 上的定位 IO 修正；`unlink` 对目录的识别；delete-on-close 的错误传播。

上游库的公开 API 与行为不变——`smb-server` crate 照旧可用。

## 构建与发布

CI 在 `windows-latest` 上跑（`.github/workflows/release-windows.yml`）：

```sh
cargo build --release --bin smbd --target x86_64-pc-windows-msvc
```

产物复制成 `dist/smbd-windows-x86_64.exe`，`v*` tag 时挂到本仓 Release；也可用 `workflow_dispatch` 手动跑一次。

## 上游库怎么用

库部分与上游完全一致：安装、嵌入与运行时配置见上游 README 的
[Install / Embed / Runtime config](https://github.com/paltaio/rust-smb-server)。

## License

MIT（与上游同一许可证，版权归 Palta.io；见 [LICENSE](LICENSE)）。
