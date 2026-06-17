基于seewo-sv21-rk3568开发板的openwrt自动构建项目[P3TERX](https://github.com/P3TERX/P3TERX)


感谢lede大佬提供
[设备支持](https://github.com/coolsnowwolf/lede/commit/36278047cf24cb58640b3bdddefb9e58a0e6ee05)和[显卡驱动](https://github.com/coolsnowwolf/lede/commit/faf9cfd9906730f0696f753bcd3a028cfd396c0d)

我添加了一些常用的软件，但未解决GPU/VPU驱动，凑合用吧

依赖文件详见depends-ubuntu-2004，刷机方法详见RK3568刷机教程.docx

## 已验证 DTB

- `dtb/kwrt-25.12-snapshot-6.12.92-v3-oldusb/`
  - 用于 KWRT 25.12-SNAPSHOT / kernel 6.12.92 的 SV21 第三版 DTB。
  - 已在实机验证：CPU 调频恢复到最高 1992MHz，RTL8153 WAN 口恢复为 `eth1`。

![seewo-sv21-rk3568](https://github.com/blacksamuraiiii/seewo-sv21-rk3568/blob/main/rk3568-seewo-sv21.jpg)
