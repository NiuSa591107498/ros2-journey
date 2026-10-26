# ros2-journey

大一这一年，从零走到「仿真里跑通 SLAM + Nav2」，再迁到真机。
这份仓库记判断依据，不只记结论。写错的判断也留着，因为错在哪比对了什么更有用。

开始日期：2026-10-26

---

## 机器档案

全部实测，来源逐条注明。不写「大概」「应该」。

| 项 | 值 | 来源 |
|---|---|---|
| 机型 | ASUS TX Gaming FX608LPG（天选） | SMBIOS Type 1 |
| 主板 | ASUSTeK COMPUTER INC. FX608LPG | SMBIOS Type 2 |
| BIOS | AMI FX608LPG.302（2026-05-08） | SMBIOS Type 0 |
| CPU | Intel Core Ultra 9 290HX Plus，24 核 24 线程 | SMBIOS Type 4 + 注册表 |
| 集显 | Intel Graphics（Arrow Lake-HX 核显） | 设备管理器 |
| 独显 | NVIDIA GeForce RTX 5070 Laptop GPU，8 GB | 设备管理器 |
| 内存 | 16 GB DDR5-5600（Micron MTC8C1084S1SC56BH1K） | SMBIOS Type 17 |
| **内存插槽** | **2 个，已用 1 个**。ChannelB-DIMM1 有 16 GB，ChannelA-DIMM1 空 | SMBIOS Type 16/17（2026-10-26 实测） |
| 内存容量上限 | 固件报 64 GB（保守值，实际以 ASUS 规格页为准） | SMBIOS Type 16 |
| 磁盘 | C: 240.4 GB（剩 82.8）／D: 683 GB（剩 167） | .NET DriveInfo |
| 磁盘布局 | 推断为**单块约 1 TB NVMe** 分 C/D（两分区合计 923.4 GB + EFI/MSR/恢复 ≈ 930 GiB） | 推断，待磁盘管理确认 |
| 固件与安全启动 | UEFI，Secure Boot 已开启 | `$env:firmware_type` + 注册表 `UEFISecureBootEnabled=1` |
| 无线网卡 | Realtek 8852CE WiFi 6E | 设备管理器 |
| 系统 | Windows 11 家庭中文版 25H2，Build 26200.9711（EditionID: CoreCountrySpecific） | 注册表 |
| BitLocker | **已确认关闭**：C: 与 D: 均为「完全解密 / 保护关闭 / 已加密 0.0% / 密钥保护器找不到」 | `manage-bde -status`，2026-10-26 |
| 已装工具 | VS Code、OpenSSH 客户端 | PATH |
| 未装 | Git、GitHub CLI、WSL 及任何 Linux 发行版 | PATH 检查 |

**一句话结论**：CPU/GPU 富余，内存是唯一短板。空着一个插槽，加一条 16 GB DDR5-5600 SO-DIMM 就能到 32 GB，不需要整组换。

---

## 路线选择

| 时间 | 环境 | 用来做什么 |
|---|---|---|
| 第 1–3 周 | Windows + WSL2 + Ubuntu 24.04 + ROS 2 Jazzy | 写节点、话题 / 服务 / 动作通信，零分区风险 |
| 第 4 周起 | 双系统 Ubuntu 24.04 + ROS 2 Jazzy | Gazebo 的 3D 渲染、真机 USB 与串口 |

### 为什么是 Ubuntu 24.04 + Jazzy，不是 22.04 + Humble

本机三样东西太新：Arrow Lake-HX 平台的 CPU 与核显、RTX 5070 Laptop（Blackwell）、Realtek 8852CE 无线网卡。
22.04 的 5.15 内核认不出这套硬件，24.04 的 HWE 内核才有 Arrow Lake 图形栈和 rtw89 的 8852CE 支持。
选 22.04 的结果是把第一个月花在驱动上，而不是花在 ROS2 上。

### 为什么先 WSL2 再双系统

第一个周末应该用来让乌龟动起来，不是和显卡驱动搏斗。WSL2 阶段验证的是「我想不想学这个」，双系统阶段才解决「仿真跑不跑得动」。

### 为什么不用 Docker / 云主机代替

Gazebo 和 RViz2 要真实的 GPU 图形栈。容器里的 GUI 加速和远程桌面都替代不了，装完能跑但帧率会让你误判成代码慢。

### 这条路的代价（选了就必须接受）

- 分区操作有数据丢失风险。从 D: 缩 120 GB 之前，重要文件必须备份到移动硬盘。
- RTX 50 系在 Linux 上需要 ≥570 的驱动版本，天选还有 MUX 与混合输出，第一次装黑屏的概率不低。
- Realtek 8852CE 的 rtw89 驱动在 Linux 下有稳定性问题报告，可能需要走有线网或换网卡。
- 保留 Secure Boot 就要给 NVIDIA 内核模块做 MOK 签名，否则装完进不去图形界面。关掉 Secure Boot 更省事，代价是 Windows 侧少一层启动防护。

---

## 阶段

- [x] 阶段 0：两张门票 —— 插槽数、BitLocker、`ros2-journey` 仓库、本 README
- [ ] 阶段 1：WSL2 + ROS 2 Jazzy 装起来，键盘能让乌龟动（第 1–3 周）
- [ ] 阶段 2：兴趣验证（40 分钟，别跳过）
- [ ] 阶段 3：切双系统（第 4 周）
- [ ] 阶段 4：进 Gazebo（第 5–6 周）
- [ ] 阶段 5：slam_toolbox 建图 + Nav2 导航（第 7–10 周）

---

## 日志

| 日期 | 做了什么 | 卡在哪 | 根因 |
|---|---|---|---|
| 2026-10-26 | 读 SMBIOS 拿到完整机器档案；确认内存 2 槽用 1 | — | — |

---

## 环境重建记录

> 每次换机器或重装，照这一节做，别照教程做。教程会过期，这一节会更新。

（待阶段 1 完成后补）
