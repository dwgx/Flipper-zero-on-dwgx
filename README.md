# Flipper-zero-on-dwgx

> 本仓库是 [@dwgx](https://github.com/dwgx) 的 Flipper Zero **官方固件 (Official Firmware / OFW)** 个人整理包。核心是自己改的 `x_ble_spam`（"闪光两点" 轻量版，原版固件开箱即用），附带一份按分类整理的 SD 卡资源快照，以及经验证的社区资源导航。
>
> 固件版本：OFW 1.4+ 兼容 · 设备：Flipper Zero · 日期：2026-04

![firmware](https://img.shields.io/badge/firmware-OFW_1.4%2B-orange) ![platform](https://img.shields.io/badge/platform-Flipper_Zero-black) ![license](https://img.shields.io/badge/license-Personal_Archive-blue)

---

## 目录

- [亮点：`x_ble_spam` 原版固件版](#亮点x_ble_spam-原版固件版)
- [快速安装](#快速安装)
- [仓库结构](#仓库结构)
- [SD 卡资源分类](#sd-卡资源分类)
- [社区资源导航](#社区资源导航)
  - [中文社区](#中文社区国内)
  - [国际 Awesome / 合集](#国际-awesome--合集)
  - [红外数据库](#红外数据库-ir-databases)
  - [SubGHz 数据库](#subghz-数据库)
  - [NFC / RFID 工具](#nfc--rfid-工具)
  - [BadUSB Payload 集合](#badusb-payload-集合)
  - [GPIO / ESP 扩展](#gpio--esp-扩展)
  - [跨固件兼容 fap](#跨固件兼容-fap)
  - [游戏合集](#游戏合集)
- [许可与免责](#许可与免责)

---

## 亮点：`x_ble_spam` 原版固件版

原始的 BLE Spam 类工具（例如 `Xtreme-Apps` / `Momentum` 里的大块头）都依赖 **RAW HCI 接口**，只能在第三方固件上跑。本仓库的 `x_ble_spam` 是 **重写的精简版**，走的是 OFW 公开的 `furi_hal_bt_extra_beacon_*` API，**所以可以直接装在原版固件上**。

| 特性 | 值 |
|---|---|
| 二进制体积 | **8156 B** (`x_ble_spam.fap`) |
| 使用 API | 官方 `furi_hal_bt_extra_beacon_*`（非 RAW HCI，不会宕机） |
| 支持设备 | 39 种（Apple / Samsung / Android / Windows / LovePlay 等） |
| 广播模式 | 单设备循环 / `TypeRotate` 轮换全部 |
| 蓝灯 | 正常广播 |
| 红灯 | BLE 初始化失败 |

### 设备列表（节选）

| 类型 | 备注 |
|---|---|
| `TypeApplePair` | Apple Proximity Pair，触发配对弹窗（近距离 ~1m） |
| `TypeAppleAction` | Apple Nearby Action，系统模态（远距离 >10m） |
| `TypeSamsung` / `TypeSamsungWatch` | Samsung EasySetup Buds / Watch |
| `TypeAndroid` | Google Fast Pair |
| `TypeWindows` | Windows Swift Pair |
| `TypeLovePlay` / `TypeStop` | LoveSpouse 成人用品遥控 |
| `TypeRotate` | 轮播所有设备 |

### 操控

- **L / R / U / D**：切换设备。**长按**翻页。
- **OK**：START / STOP 切换。广播中切设备会自动重启以应用新协议。
- **BACK**：退出，自动恢复正常 BT 广播。

### 已知协议层限制（**非代码 Bug**）

- `Setup New iPhone` / `Apple Watch` / `AutoFill` / `iOS17 Crash` 已被 iOS 16.6+ / 17.2+ 修复。
- Beats Flex / Solo / Studio 有自身冷却机制（厂商固件层面）。
- Apple Watch 弹窗显示 "Not Your Device" 属预期行为。

### 数据包细节（踩过的坑）

- **Apple Proximity Pair**：电池字节用 `rand() % 16`（不是 `% 10`，否则 Beats 不弹）。
- **Apple Nearby Action**：Continuity type `0x0F`，flags `0xC0`（Join AppleTV 用 `0xBF`）。
- **BLE 地址类型**：必须 `GapAddressTypeRandom`，MAC 高 2 位强制 `0xC0`。
- **启动前**：停止正常 BT 广播，退出时恢复。

详细源码：[`custom_apps/x_ble_spam/x_ble_spam.c`](custom_apps/x_ble_spam/x_ble_spam.c)

---

## 快速安装

### 方式 1：复制 `.fap` 到 SD 卡

```
SD:/apps/Bluetooth/x_ble_spam.fap
```

插回 Flipper → 菜单 **Apps → Bluetooth → X BLE Spam** 即可运行。

### 方式 2：从源码用 uFBT 构建

```bash
python -m pip install --user ufbt
cd custom_apps/x_ble_spam
ufbt               # 构建
ufbt launch        # 构建并推送到设备启动
```

> 前提：Flipper 接 USB，`ufbt` 在 PATH 上。Windows 用户请通过 `py -m pip install --user ufbt` 安装。

---

## 仓库结构

```
Flipper-zero-on-dwgx/
├── README.md                       # 本文件
├── LICENSE                         # 个人存档许可
├── custom_apps/
│   └── x_ble_spam/                 # ★ 主打：原版固件 BLE Spam
│       ├── x_ble_spam.c            # 主源文件 (15KB)
│       ├── application.fam         # uFBT manifest
│       ├── esp_boost.{c,h}         # ESP32 辅助模块（可选）
│       ├── icons/                  # 图标资源
│       └── *.fap                   # 预构建产物
├── sd_pack/                        # SD 卡整理快照（去敏后）
│   ├── apps/                       # 按分类的 .fap
│   │   ├── Bluetooth/
│   │   ├── NFC/
│   │   ├── Sub-GHz/
│   │   ├── GPIO/
│   │   ├── Tools/
│   │   ├── Media/
│   │   ├── Games/
│   │   ├── Infrared/
│   │   ├── RFID/
│   │   ├── USB/
│   │   ├── iButton/
│   │   └── Scripts/                # JS 示例
│   ├── apps_assets/                # 应用资产
│   ├── apps_manifests/             # 应用 manifest
│   ├── badusb/
│   │   ├── UberGuidoZ_BadUSB/      # UberGuidoZ 社区包
│   │   ├── hak5_Ducky/             # Hak5 官方 DuckyScript
│   │   └── demo_*.txt              # 官方 demo
│   ├── infrared/
│   │   ├── assets/                 # 官方内置 IR（AC/TV/音响/投影）
│   │   └── assets_user/Lucaslhm_IRDB/  # Lucaslhm 红外大库（含国内家电）
│   ├── nfc/
│   │   ├── assets/                 # 官方字典 + AID/国家/货币编码
│   │   ├── assets_user/UberGuidoZ_NFC/  # UberGuidoZ NFC 集
│   │   └── pack_*.nfc              # NDEF demo 包
│   └── subghz/
│       ├── assets/                 # 官方协议 (Alutech / Came / KeeLoq / Nice)
│       └── assets_user/UberGuidoZ_SubGHz/  # UberGuidoZ SubGHz 大库
└── docs/                           # 详细文档（见各分类）
```

**已剔除内容（出于隐私/版权）：**
- ❌ `wav_player/` (189MB，含个人音乐收藏)
- ❌ `apps_data/` (设备运行时日志与状态)
- ❌ `update/` (固件镜像，应从 Flipper 官方获取)
- ❌ `nfc/Icoca.nfc`（日本交通卡 dump）
- ❌ `nfc/mf_classic_dict_user.*`（个人收集的门禁密钥）
- ❌ `infrared/*.ir`（家里空调等个人捕获）
- ❌ `*/REJECTED.log`（个人失败捕获日志）

---

## SD 卡资源分类

### 📱 `apps/Bluetooth/`
- **`x_ble_spam.fap`** ★ 本仓库主打，OFW 原生
- `x_findmy.fap` — Apple Find My 信标
- `hid_ble.fap` — BLE HID 键盘/鼠标/媒体控制
- `bt_trigger.fap` — BLE 触发器
- `pc_monitor.fap` — PC 资源监控

### 📶 `apps/NFC/`
- `nfc_magic.fap` — Magic 卡（Gen1/2/4）读写
- `mifare_fuzzer.fap` — MIFARE 模糊测试
- `nfc_apdu_runner.fap` — APDU 脚本执行
- `nfc_rfid_detector.fap` — 场检测（NFC/125K 都探）
- `picopass.fap` — PicoPass / HID iClass 读写
- `iso15693_nfc_writer.fap` — ISO15693 标签写入
- `x_nfc_maker.fap` — NFC 卡快速制作工具

### 📡 `apps/Sub-GHz/`
- `subghz.fap` — 扩展 SubGHz（CLI、分析增强）
- `subghz_playlist_creator.fap` — 播放列表生成器
- `tpms.fap` — 胎压监测解码
- `weather_station.fap` — 气象站接收

### 🔌 `apps/GPIO/`
- `esp32_wifi_marauder.fap` / `ghost_esp.fap` — ESP32 WiFi 安全工具伴侣
- `nrf24_mouse_jacker.fap` / `nrf24tool.fap` — NRF24 键鼠劫持
- `flipboard_keyboard.fap` — FlipBoard 键盘
- `uart_terminal.fap` — UART 终端
- `logic_analyzer.fap` — 逻辑分析仪
- `24cxxprog.fap` — EEPROM 编程器
- `i2ctools.fap` — I2C 工具
- `signal_generator.fap` / `servotester.fap` / `lightmeter.fap` / ...

### 🛠️ `apps/Tools/`
- `qrcode.fap` — QR 码生成/显示
- `hex_editor.fap` / `hex_viewer.fap` — HEX 编辑与查看
- `text_viewer.fap` / `x_textreader.fap` — 文本阅读（后者支持长文/小说）
- `multi_converter.fap` — 进制 / 编码转换
- `passgen.fap` — 密码生成器
- `programmercalc.fap` — 程序员计算器
- `segment_clock.fap` — 段式时钟
- `flipp_pomodoro.fap` / `cntdown_tim.fap` — 番茄钟 / 倒计时
- `x_barcode.fap` / `x_counter.fap` — 条码显示 / 计数器
- `theme_manager.fap` — 主题管理
- `resistors.fap` — 电阻色环查询

### 🎮 `apps/Games/`
- `doom.fap` — 经典 DOOM 移植
- `tetris.fap` / `snake20.fap` / `snake_game.fap` / `flappy_bird.fap` / `game_2048.fap`
- `jetpack_joyride.fap` / `heap_defence.fap` / `t_rex_runner.fap`
- `minesweeper_redux.fap` / `solitaire.fap`

### 🎵 `apps/Media/`
- `wav_player_plus.fap` — 增强版 WAV 播放器
- `sleep_saver.fap` — 动画待机壁纸
- `metronome.fap` / `bpm_tapper.fap` — 节拍器 / BPM 拍测
- `morse_code.fap` — 摩斯电码
- `ocarina.fap` — 陶笛（塞尔达风）
- `tuning_fork.fap` — 音叉
- `text2sam.fap` — 文字转 SAM 语音

### 🔴 `apps/Infrared/` & `apps/RFID/` & `apps/USB/` & `apps/iButton/`
- `infrared.fap` / `htw_ac_remote.fap` — 扩展红外 + HTW 空调专用
- `lfrfid.fap` — 125kHz LF RFID 扩展
- `bad_usb.fap` / `hid_usb.fap` / `u2f.fap` — BadUSB / USB HID / U2F
- `ibutton.fap` — iButton (1-Wire) 扩展

---

## 社区资源导航

> 所有链接经过 `gh api` 实时验证存在（2026-04）。

### 中文社区（国内）

| 项目 | 说明 |
|---|---|
| [ZhaiRenGaiZaoJia/FlipperZero-CN-Firmware](https://github.com/ZhaiRenGaiZaoJia/FlipperZero-CN-Firmware) | 国内主流深度汉化优化固件 |
| [kalicyh/Momentum-Firmware-CN](https://github.com/kalicyh/Momentum-Firmware-CN) | Momentum 固件中文版 |
| [kalicyh/awesome-flipperzero-i18n](https://github.com/kalicyh/awesome-flipperzero-i18n) | Awesome 列表中文翻译 |
| [Lucaslhm/Flipper-IRDB](https://github.com/Lucaslhm/Flipper-IRDB) | 红外库（含格力美的小米华为 TCL 等国内品牌） |
| [RustySchackelford000/Flipper-Zero-Mfkey32-dictionary](https://github.com/RustySchackelford000/Flipper-Zero-Mfkey32-dictionary) | 超大 M1 门禁字典，破小区/校园卡 |

### 国际 Awesome / 合集

| 项目 | 说明 |
|---|---|
| [djsime1/awesome-flipperzero](https://github.com/djsime1/awesome-flipperzero) | 资源总目录（必收藏） |
| [UberGuidoZ/Flipper](https://github.com/UberGuidoZ/Flipper) | 全球最大文件库（IR/NFC/SubGHz/BadUSB 全分类） |
| [xMasterX/all-the-plugins](https://github.com/xMasterX/all-the-plugins) | 第三方 fap 插件大合集 |
| [playmean/fap-list](https://github.com/playmean/fap-list) | 预编译 fap 清单 |
| [flipperdevices/flipperzero-ufbt](https://github.com/flipperdevices/flipperzero-ufbt) | 官方 uFBT 构建工具 |

### 红外数据库 (IR Databases)

| 项目 | 说明 |
|---|---|
| [Lucaslhm/Flipper-IRDB](https://github.com/Lucaslhm/Flipper-IRDB) | 综合库（本仓库已集成在 `sd_pack/infrared/assets_user/`） |
| [UberGuidoZ/Flipper-IRDB](https://github.com/UberGuidoZ/Flipper-IRDB) | UberGuidoZ 维护的 IR 大库 |
| [logickworkshop/Flipper-IRDB](https://github.com/logickworkshop/Flipper-IRDB) | 原始分类详尽的红外遥控数据库 |

### SubGHz 数据库

| 项目 | 说明 |
|---|---|
| [Zero-Sploit/FlipperZero-Subghz-DB](https://github.com/Zero-Sploit/FlipperZero-Subghz-DB) | 13,000+ 信号，门禁/车库最全合集 |
| [UberGuidoZ/Flipper-SGDB](https://github.com/UberGuidoZ/Flipper-SGDB) | UberGuidoZ 车库信号库 |
| [DefinetlyNotAI/Full_Flipper_Database](https://github.com/DefinetlyNotAI/Full_Flipper_Database) | 多源整合综合库 |
| [DarkFlippers/flipperzero-subbrute](https://github.com/DarkFlippers/flipperzero-subbrute) | SubGHz 协议暴力破解验证 |
| [shalebridge/flipper-subghz-scheduler](https://github.com/shalebridge/flipper-subghz-scheduler) | `.sub` 文件定时发送工具 |

### NFC / RFID 工具

| 项目 | 说明 |
|---|---|
| [zacharyweiss/magspoof_flipper](https://github.com/zacharyweiss/magspoof_flipper) | 磁条卡模拟（MagSpoof 移植） |

### BadUSB Payload 集合

| 项目 | 说明 |
|---|---|
| [hak5/usbrubberducky-payloads](https://github.com/hak5/usbrubberducky-payloads) | Hak5 官方 DuckyScript 经典集 |
| [I-Am-Jakoby/Flipper-Zero-BadUSB](https://github.com/I-Am-Jakoby/Flipper-Zero-BadUSB) | 知名研究员高级 Payload |
| [FalsePhilosopher/badusb](https://github.com/FalsePhilosopher/badusb) | 社区活跃维护 |
| [Kavitate/FlipperZeroBadUSB](https://github.com/Kavitate/FlipperZeroBadUSB) | Win/Mac 实践脚本集 |

### GPIO / ESP 扩展

| 项目 | 说明 |
|---|---|
| [justcallmekoko/ESP32Marauder](https://github.com/justcallmekoko/ESP32Marauder) | ESP32 WiFi/BLE 安全工具（固件端） |
| [0xchocolate/flipperzero-wifi-marauder](https://github.com/0xchocolate/flipperzero-wifi-marauder) | Marauder 的 Flipper 端伴侣 |
| [SkeletonMan03/FZEasyMarauderFlash](https://github.com/SkeletonMan03/FZEasyMarauderFlash) | Marauder 一键烧录工具 |
| [0xchocolate/flipperzero-esp-flasher](https://github.com/0xchocolate/flipperzero-esp-flasher) | Flipper 直接烧 ESP，无需电脑 |

### 跨固件兼容 fap

> 下列项目用 uFBT 可编译成 OFW 兼容的 `.fap`：

| 项目 | 说明 |
|---|---|
| [LTVA1/flipper-zero-video-player](https://github.com/LTVA1/flipper-zero-video-player) | 30fps 带声高清视频播放器 |
| [John4E656F/fl-BLE_SPAM](https://github.com/John4E656F/fl-BLE_SPAM) | 另一个 BLE Spam 实现参考 |

### 游戏合集

| 项目 | 说明 |
|---|---|
| [doofy-dev/flipper_games](https://github.com/doofy-dev/flipper_games) | 扑克/21 点等游戏包 |

### 官方商店

- **[Flipper App Catalog](https://lab.flipp.dev)** — OFW 的浏览器直装商店，最安全的一手来源。

---

## 许可与免责

- **我自己写的代码**（主要是 `custom_apps/x_ble_spam/`）：遵循根目录 `LICENSE`（**个人使用 · 禁止二次分发 · 禁止商用**）。
- **第三方 `.fap` 与资源包**（`sd_pack/` 下大部分内容）：版权归各原作者，**本仓库仅为个人学习/整理的方便快照**。商业再分发请查看对应上游仓库的 LICENSE。
- `sd_pack/badusb/UberGuidoZ_BadUSB`、`sd_pack/badusb/hak5_Ducky`、`sd_pack/subghz/assets_user/UberGuidoZ_SubGHz`、`sd_pack/nfc/assets_user/UberGuidoZ_NFC`、`sd_pack/infrared/assets_user/Lucaslhm_IRDB` 均为上游社区包的**镜像**，原始仓库见上文 "社区资源导航"。
- **使用本仓库内容的风险自负**。BLE Spam / BadUSB / RFID Fuzzer 等工具仅限**本人设备安全测试 / CTF / 教学**使用。严禁用于未授权的攻击、骚扰、商业用途或任何违法行为。在中国大陆境内，部分工具的公开使用可能触犯《网络安全法》《治安管理处罚法》《刑法》等法规，请自行判断。
- 如果本仓库无意中包含了你的原创内容但未正确署名，或你是上游作者希望本仓库移除相关镜像，请提 issue，会立刻处理。

---

## 欢迎

- ⭐ 如果这份整理对你有帮助，点个 Star。
- 🐛 发现问题 / 想补充资源 → issue or PR（仅文档/导航 PR，代码 PR 暂不合并）。
- 作者：[@dwgx](https://github.com/dwgx)
