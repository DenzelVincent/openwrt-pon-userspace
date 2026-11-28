# pon-userspace 补丁与优化记录（bugfix）

起始：2026-11-15
上游：`pbs05/openwrt-pon-userspace`，对比基准 `upstream/main a903fcc`（2026-11-16，2026-11-24 拉取）。
重建基线：`main 2c979206e4`（2026-11-28）；旧 bugfix 保存在 `backup/bugfix-pre-rebuild-20261128`，本轮不恢复旧功能。
范围：本分支记录相对上游与相对主分支的优化；驱动侧见 `openwrt-pon-drivers` 台账，固件侧见 PonWrt 台账。

记录规范：

- 已合入主分支的项目归「相对上游」；bugfix 独有项目归「相对主分支」，chore/文档整理合并成一行。
- 表列：优化项、优化方案简介、关联提交、验收状态、验收说明。
- 验收状态列只记录实机验证结果（`已验收`/`待验收`，括号内可注场景或阻塞原因）；其余记 `—`。

## 一、相对上游

| 优化项 | 优化方案简介 | 关联提交 | 验收状态 | 验收说明 |
|---|---|---|---|---|
| luci-app-pon 概览页光模块状态卡片 | 概览页新增光模块状态卡片，经 `ponctl frontend` 读 rx/tx/温度，读不到时整卡隐藏 | `9667359` | 待验收（未进固件） | 不适用标准 SFP/ethtool -m 口径；进固件后核卡片读数与读不到时的隐藏 |
| 光模块卡片补线路模式/线路状态/ONU 状态 | 卡片内补线路模式、线路状态与 ONU 状态；无 WiFi 机型不加载 hg5585f 配置 | `1128d06` | 待验收（未进固件） | 进固件后核三项显示与无 WiFi 机型的配置加载 |
| 忽略 IDE 与 tmp | 忽略 IDE 与 tmp 目录 | `5cd9a46` | — | 仅仓库整理，不进固件 |

## 二、相对主分支（bugfix 新增）

| 优化项 | 优化方案简介 | 关联提交 | 验收状态 | 验收说明 |
|---|---|---|---|---|
| CTC 多 VLAN 对象实例累积 | 以 object instance 为键保留全部 OLT 下发 VLAN，重复 Set 只替换同一实例，最多 32 条；`vlan_ids` 输出去重并集。来源：[pbs05/openwrt-pon-userspace PR #14 3f892688018d1b7f5778695915326b99ebfcfba6](https://github.com/pbs05/openwrt-pon-userspace/commit/3f892688018d1b7f5778695915326b99ebfcfba6) | `6a2713a19efe02ce315d209c6c1fd0fd6b5270f0` | — | 已合并上游 PR #14 的多实例、重复 Set、Transparent 三个测试；本机无 Rust 工具链，待 `cargo test` 与实机核对完整 VLAN 列表 |
| airoha-pond 启动即保护 OOM 评分 | 每次 daemon 启动时把 `/proc/self/oom_score_adj` 写为 `-1000`，覆盖 procd respawn 重新 exec 后的评分重置；失败仅告警不阻断 PON 服务。来源：[wuziwei/ponwrt 713311e9bb87cd506d2ee9b54e0a2f3ab590e1ab](https://github.com/wuziwei/ponwrt/commit/713311e9bb87cd506d2ee9b54e0a2f3ab590e1ab) | `364f1c7d104a0503ff1afa9491df2c418d92d6bf` | — | 新增写入换行与启动路径单测；本机无 Rust 工具链，待编译并实机核对 respawn 后评分仍为 `-1000` |

## 三、待办

- 光模块卡片进固件后在 XG-040G-MD 上验收。
- 运行 `cargo test`，并在 OLT 下发多个 VLAN 实例与 procd respawn 场景验收。
