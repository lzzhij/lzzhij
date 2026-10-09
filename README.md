# 鸿蒙端侧运行时 · 内嵌 Node.js · 端侧 Agent

> 在 **HarmonyOS / OpenHarmony** App 的**进程内**跑真正的 Node.js。
> 所有结论来自**真机实测** —— **包括失败的结论**。

---

## 📖 先看这个：实战笔记

**[harmonyos-node-notes](https://github.com/lzzhij/harmonyos-node-notes)** —— 5 篇，842 行，全部真机验证

| 篇 | 内容 |
|---|---|
| 01 | **6 条硬约束与 9 个必崩坑** —— 含 13 行「症状 → 真因 → 修法」速查表 |
| 02 | **常驻与子进程的实测死法** —— `801` / `9800005` / `9900002` 完整取证 |
| 03 | **包体瘦身 151.6 MB → 71.8 MB** —— `compressNativeLibs` 该写在哪一层 |
| 04 | **运行期限制与等价替代** —— `flock` / `link` / `spawn` / `homedir` |
| 05 | **排障通道搭建** —— 四条通道 + 三个「假信号」 |

## 🩺 再跑这个：本机体检

**[ohos-node-doctor](https://github.com/lzzhij/ohos-node-doctor)** —— 零依赖，自己解析 ELF，一条命令查出违规项

```bash
node ohos-node-doctor.mjs /path/to/your/harmony
```

| 检查 | 为什么重要 |
|---|---|
| 是否真共享库（`ET_DYN` 且**无 `PT_INTERP`**） | 带 `PT_INTERP` 的 PIE 被 `dlopen` 会崩在 **V8 第一次堆分配** |
| `DT_NEEDED` 依赖是否齐备 | 漏 `libc++_shared.so` 时 ArkTS 侧**只拿到 `undefined`，不报错** |
| 是否装 SIGSYS shim | 不装则 libuv 在 loop init 就崩（`io_uring_setup` 被 seccomp trap） |
| `argc` 是否排除 argv 末尾哨兵 | 多算一个 ⇒ Node 拿到 NULL ⇒ **5 毫秒内段错误** |
| 打包配置 / 权限 / 包名版本 | 体积差一倍、上架被驳回的常见原因 |

**自检 18 条断言全通过；CI 在 Node 18/20/22 × Ubuntu/Windows 上 6/6 绿。**

---

## 🔧 这些经验覆盖的典型症状

如果你在搜下面任何一条，笔记里都有答案：

```
dlopen 成功但 dlsym 全 null
node::Start 里 SIGTRAP：Check failed: 12 == (*__errno_location())
启动即 ANR：APP_INPUT_BLOCK
dlopen 失败但 ArkTS 侧只拿到 undefined
internal/modules/* 全部 MODULE_NOT_FOUND
flock is not supported on openharmony-arm64
EACCES: permission denied, link '...tmp' -> '...final'
Capability not support. (code=801)
9800005  /  9900002
arkts-no-any-unknown
HAP 里没有 libs/
```

---

## 📬 联系

- **免费自助**：上面两个仓库覆盖 12 类已解问题，绝大多数情况够用
- **公开提问**：开 issue（附**症状原文**与 `llvm-readelf` 输出 —— 原始输出能省三轮来回）
- **付费支持**（远程诊断 / 集成交付 / 上架支持）：`lzj031216@163.com`

**我的边界（先说清）**：不承诺"上架一定通过"；不承诺设备不支持的能力
（`801` / `9800005` / `9900002` 都有真机证据）；**不索要你的密钥、证书或凭据**。

---

<sub>这里的东西我会持续更新 —— 每解决一个真机问题就记一条。</sub>
