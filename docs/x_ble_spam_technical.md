# `x_ble_spam` 技术细节

深入文档，主 README 见 [../README.md](../README.md)。

## 为什么能装在原版固件

绝大多数 Flipper BLE Spam 实现（Momentum / Xtreme / RogueMaster 内嵌的那些）都依赖两类接口：

1. **RAW HCI 接口** — 直接往 BT 协处理器喂命令。Flipper 的官方固件不开放这个接口给外部 app（`furi_hal_bt` 只暴露一部分受控的 API）。
2. **`furi_hal_bt_set_profile` / `furi_hal_bt_stop_advertising`** 等受限 API — 官方 fap SDK 对外部 app 不开放。

本实现只用这一条链路：

```
furi_hal_bt_extra_beacon_set_config()   // 配置 adv 参数（间隔、功率、地址类型等）
furi_hal_bt_extra_beacon_set_data()     // 塞 manufacturer data
furi_hal_bt_extra_beacon_start()        // 启动 extra beacon（与主 BT 服务并存）
furi_hal_bt_extra_beacon_stop()         // 停止
```

`extra_beacon` 是官方暴露给外部 app 的辅助广播通道，**与主 BT 栈并行**，所以：

- 不需要 patch 固件
- 不会和普通 BT 服务冲突（只会自动 stop/restart 一次主 adv）
- 不会触发 BT 协处理器 crash（老 BLE Spam 的典型问题）

## 体积对比

| 实现 | 来源 | `.fap` 大小 | 固件需求 |
|---|---|---|---|
| 本仓库 `x_ble_spam` | 自写 | **8 KB** | OFW 1.4+ |
| BLE Spam (Xtreme-Apps) | Xtreme | ~60 KB | Xtreme / RM |
| BLE Spam (Momentum) | Momentum | ~80 KB | Momentum 独占 |

## 数据包格式详解

### Apple Proximity Pair (`TypeApplePair`)

```
AD Structure:
  len: 0x1E (30)
  type: 0xFF (Manufacturer Specific)
  company: 0x004C (Apple)
  continuity: 0x07 (Proximity Pair)
  sublen: 0x19 (25)
  flags: 0x01
  device_id[2]          // 见下表
  color: 0x00..0xFF     // 根据设备型号
  battery: rand()%16    // ★ 不是 %10，否则 Beats 不弹
  ...
```

设备 ID 部分摘要：
| 设备 | ID |
|---|---|
| AirPods (1st) | `0x01 0x02` |
| AirPods Pro | `0x0E 0x20` |
| AirPods Max | `0x0A 0x20` |
| Beats Studio3 | `0x09 0x20` |
| Powerbeats Pro | `0x0B 0x20` |
| ... | |

### Apple Nearby Action (`TypeAppleAction`)

```
continuity: 0x0F (Nearby Action)
flags: 0xC0         // ★ 注意：Join AppleTV 用 0xBF，不是 0xC0
action_type:
  0x13 — AppleTV AutoFill
  0x27 — AppleTV Connect
  0x20 — Join AppleTV
  ...
auth_tag: rand()
```

### BLE 地址类型

**必须** `GapAddressTypeRandom`，并把 MAC 地址**高 2 位强制置为 `0xC0`**（否则被当成 public 地址，iOS 不处理）：

```c
addr[5] |= 0xC0;
addr[5] &= 0xCF;
```

## 已知协议层限制（不是代码 Bug）

| 问题 | 原因 |
|---|---|
| iOS 16.6+ 不再弹 "Setup New iPhone" | Apple 在 iOS 16.6 修复了协议 |
| iOS 17.2+ 不再弹 "AutoFill" | Apple 在 iOS 17.2 修复了协议 |
| Beats Flex / Solo / Studio 冷却 ~10 min | 耳机固件层面机制 |
| Apple Watch 弹窗显示 "Not Your Device" | 预期行为：未配对的手表视图 |

这些都是上游协议被修复了，**本工具代码层面无法绕过**。

## 故障排查

### 红灯常亮 → BLE 初始化失败

- `furi_hal_bt_extra_beacon_start()` 返回失败
- 最常见原因：另一个 BT app 没退干净。退回主菜单再试

### 蓝灯亮但 iPhone 不弹

- 检查 iOS 版本：某些设备类型在新系统已失效（见上表）
- 距离太远：Proximity Pair 类要求 <1m；Nearby Action 类可 >10m
- 地址未随机化：确认 `GapAddressTypeRandom` 而不是 `Public`

### 菜单切设备后没效果

- **切设备时会自动 stop → 替换 data → start**
- 如果没重启：检查 OK 键是否真的处在 START 状态
