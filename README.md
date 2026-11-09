# NumberPAD — ZMK 固件配置

17 键数字小键盘，ProMicro nRF52840（Nologo SuperMini / 兼容 nice!nano）主控。
基于 ZMK v0.3 + ZMK Studio。

## 使用方法

1. 把这个目录作为仓库推到 GitHub（公开或私有均可）。
2. Actions 里跑 `Build ZMK firmware`，下载 artifact `numberpad`。
3. 双击模块上的 RST 焊盘进 bootloader（出现 `NICENANO` 盘符），把 `numberpad.uf2` 拖进去。

本地构建：

```bash
west init -l config
west update
west build -s zmk/app -b nice_nano_v2 -S studio-rpc-usb-uart \
  -- -DSHIELD=numberpad -DZMK_CONFIG="$PWD/config"
```

## 改键

固件已开 ZMK Studio，用 https://zmk.studio/ 或桌面版在线改键。改过一次之后
`.keymap` 里的内容就不再生效，除非在 Studio 里做 "Restore Stock Settings"。

## 硬件映射（来自 PCB 网表）

| 矩阵 | 模块引脚 | nRF52840 |
| --- | --- | --- |
| COL0–COL3 | 21 / 20 / 19 / 18 | P0.31 / P0.29 / P0.02 / P1.15 |
| ROW0–ROW4 | 1 / 0 / 2 / 3 / 4 | P0.06 / P0.08 / P0.17 / P0.20 / P0.22 |

二极管 `col2row`：阳极在列线，阴极经开关回行线。

注意：

- 模块焊在**PCB 背面**——就是热插拔轴座和二极管所在的那一面，**元件面朝外**（翻到背面
  能看见芯片和 USB 口）。USB 口朝远离电池座的那一端（板子上边缘）。脚序由网表唯一确定：
  模块 B+/RAW 那一列插进靠板中心的那一列孔（pad1 列），B- 那一列插进靠板边缘的那一列。
  焊之前先用万用表通断档核一遍：pad3 和 pad26 对电池负极（P1 的 GND 脚）导通，pad2 对
  电源开关输出导通。pad22/pad23 在板上是悬空的（模块内部 GND 同网，不用管）。
- 板上没接 RST 按键，进 bootloader 只能短接模块 RST 焊盘到 GND 两次。第二层给了
  一个 `&bootloader` 键作为替代入口。
- 电池经电源开关接到模块 RAW，B+ 焊盘（pad1）是空的，所以模块的板载充电器可能不给电池
  充电——第一次用 5V 充完电后实测确认一下。电量读取走 VDDH，插 USB 时读数偏高属正常。

## 默认布局

```
NumLock  /    *    -
7        8    9    +      (2u)
4        5    6
1        2    3    Enter  (2u)
0 (2u)         .
```

长按 `NumLock` 进第二层：`BT_SEL 0/1/2`、`BT_CLR`、`OUT_USB`、`OUT_BLE`、
`Studio Unlock`、`Sys Reset`、`Bootloader`。
