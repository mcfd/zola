+++
title =  "Nvidia显卡如果最低时钟频率非0 则RTD3无法关闭独显"
date = "2024-12-24"

[taxonomies]
tags = ["Nvidia","nvidia","显卡"]
+++

~~*在今天折腾了很久的一个很神奇的事。*~~

<!--more-->

<!--more-->

<!--more-->

<!--more-->

### nvidia_oc

```bash
./nvidia_oc set --index 0  --mem-offset 400 --min-clock 787  --max-clock 2100
```

其中的 <u>*--min-clock 787*</u> 部分设置了显卡最小时钟频率。

在我使用独显直连使用时，如果开启一个新的桌面应用，这时显卡时钟频率会从一个很低的值突然一下跳到 **1200** 以上，这就造成了突发的卡顿感。



### Hybrid

当然在 **Hybrid** 上已经使用了 **Intel** 核显，并且体验非常好。这会 **Nvidia** ~~就可以滚蛋了！~~ 当然在需要的时候还是要用的，只是在不用时显卡空转的十几W功耗十分感人<cite>[^1]</cite>。

这下我就需要让它在不干活的时候一边凉快去。

因为之前抄 [Cachyos](https://wiki.cachyos.org/features/cachyos_settings/) 的配置东西都已经配好了，在正常情况下的结果应该是

```bash
cat /proc/driver/nvidia/gpus/0000:01:00.0/power

Runtime D3 status:          Enabled (fine-grained)
Video Memory:               Off

GPU Hardware Support:
 Video Memory Self Refresh: Supported
 Video Memory Off:          Supported

S0ix Power Management:
 Platform Support:          Supported
 Status:                    Disabled
```

使用 `cat /sys/class/drm/card*/device/power_state` 应该能在多个 GPU 中看到 **D3cold**

```bash
cat /sys/class/drm/card*/device/power_state

D3cold
D0
```

然而在跑了 **nvidia_oc** 的情况下独显一直在空转，在 `nvidia-smi` 中一直可以看到一个Xorg 进程常驻（当然这是正常情况，另外运行 `nvidia-smi` 也会造成独显被激活）

在我倒腾了很久后，思考了是不是 **nvidia-persistenced.service** 造成的时候，才发现**nvidia_oc.service** 我的开机自动超频服务，顿时灵光一闪！

```bash
./nvidia_oc set --index 0 --min-clock 0
```

然后就成功进入了 **Suspended** 当然有些应用可能会跑在独显上导致唤醒，比如 *截图还有Steam* 什么的

`LIBVA_DRIVER_NAME=iris or iHD` 可能会改善一些?



[^1]:（<u>主要是因为楼上家庭服务器老E5太吃电被吃怕了</u>）












