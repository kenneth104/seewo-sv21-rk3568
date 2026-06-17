# Seewo SV21 KWRT 6.12.92 v3-oldusb DTB

本目录保存一份已在实机验证通过的 Seewo SV21 RK3568 KWRT DTB。

## 文件

- `rk3568-seewo-sv21-kwrt-6.12.92-v3-oldusb.dtb`
  - 可直接替换启动分区 `rockchip.dtb` 的二进制 DTB。
- `rk3568-seewo-sv21-kwrt-6.12.92-v3-oldusb.dts`
  - 上述 DTB 对应的反编译 DTS，便于日后审计和继续修改。
- `SHA256SUMS`
  - DTB 和 DTS 的校验值。

## 背景

第三方 KWRT/OpenWrt.ai 镜像在 Seewo SV21 上刷入后，LAN 口 `eth0` 正常，
但内置 USB Realtek RTL8153 WAN 口没有枚举，系统只有 `eth0`，没有 `eth1`。

前两版 DTB 尝试过：

- 增加疑似 USB hub 供电 GPIO0_D5/GPIO 29；
- 将 `usb@fd000000` 改为 USB2-only/high-speed；

实机都能启动，但 RTL8153 仍然不出现。因此第三版不再继续猜单个 GPIO 或
USB2/USB3 参数，而是改用旧系统中已确认 WAN 可工作的 DTB 作为 USB 拓扑底稿，
再补回 KWRT 所需的板名和 CPU 调压修正。

## 主要改动

第三版基于旧可用 DTB 生成，并做了以下最小修正：

- 根节点身份改为 KWRT 当前识别需要的：
  - `compatible = "seewo,sv21", "rockchip,rk3568";`
  - `model = "seewo SV21";`
- CPU 供电调压节点改为可被内核驱动识别的芯片：
  - 从 `tcs,tcs4525` 改为 `silergy,syr827`
  - I2C 地址保持 `reg = <0x40>`
  - CPU supply phandle 保持原引用关系，避免 CPU OPP 断链
- 保留旧 DTB 中已知可用的 `usb@fd000000` USB3/xHCI 拓扑，使 RTL8153
  继续挂在 SuperSpeed USB bus 上。
- 为 `usb@fd000000/device@2` 补充子节点地址描述：
  - `#address-cells = <0x01>;`
  - `#size-cells = <0x00>;`

## 实机验证

验证时间：2026-06-18

设备：Seewo SV21 RK3568

系统：

- `Kwrt 25.12-SNAPSHOT 06.10.2026`
- kernel `6.12.92`
- target `rockchip/armv8`
- arch `aarch64_generic`

启动 DTB：

```text
47c9bd6792bf3b72379db1e81661181844b885d05fa7c934f59dbf2404184e5f  /mnt/mmcblk1p1/rockchip.dtb
```

板名识别：

```text
compatible=seewo,sv21 rockchip,rk3568
model=seewo SV21
```

CPU 调频：

```text
scaling_driver=cpufreq-dt
scaling_governor=schedutil
scaling_available_frequencies=408000 600000 816000 1104000 1416000 1608000 1800000 1992000
cpuinfo_max_freq=1992000
scaling_max_freq=1992000
```

实机采样中 CPU 已到达 `1992000`，不再卡在 `816000`。

调压芯片识别：

```text
/proc/device-tree/i2c@fdd40000/regulator@40/compatible=silergy,syr827
fan53555-regulator 0-0040: FAN53555 Option[8] Rev[1] Detected!
```

WAN/RTL8153：

```text
8-1 0bda:8153 USB 10/100/1000 LAN
eth1 -> ../../devices/platform/fd000000.usb/xhci-hcd.1.auto/usb8/8-1/8-1:1.0/net/eth1
```

`eth1` 状态：

```text
operstate=up
carrier=1
speed=1000
duplex=full
```

WAN DHCP：

```text
ifstatus wan:
  up=true
  l3_device=eth1
  proto=dhcp
  device=eth1
```

## 可复现校验

使用 `dtc` 从本目录 DTS 重新编译出的 DTB，与本目录二进制 DTB 逐字节一致：

```text
47c9bd6792bf3b72379db1e81661181844b885d05fa7c934f59dbf2404184e5f
```

重新编译时会出现一些由反编译 DTB 带来的 phandle 检查警告，例如
`clocks_property`、`resets_property`、`gpios_property`。这些警告在当前 DTS
往返校验中未改变最终 DTB 二进制，实机验证也已通过。

## 使用注意

替换前必须备份现有启动分区中的 `rockchip.dtb`。推荐只在已确认有恢复手段
的情况下替换，例如能拆机离线改启动分区，或有串口/其他救援方式。

实机启动分区在当前 KWRT 上观察为：

```text
/mnt/mmcblk1p1/rockchip.dtb
```

替换后需要重启才会生效。不要在业务运行中随意重启。

## 风险和后续维护

- 这版是基于旧可用 DTB 的整体验证版，改动面比“只补一个 GPIO”的小补丁大。
- 已在一台 SV21 + KWRT kernel 6.12.92 上验证 CPU 调频、LAN、WAN 正常。
- 如果未来 KWRT 更新内核、板级 DTS 或启动分区布局，需要重新反编译并比对，
  不能无条件假定本 DTB 永远适配新版。
- iStore 和 ZeroTier 属于 rootfs/软件层，DTB 本身不直接修改它们；但未来整包
  更新如果覆盖启动分区 DTB，仍可能重新引入 WAN 或 CPU 调频问题。
