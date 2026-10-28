# 幻趣盒 · 触控游戏终端（RK3568 + LVGL 9）

> 课程项目 / 个人项目 · 2025.10 – 2025.12
> 平台：RK3568（aarch64 Linux）· GUI：LVGL 9 · 语言：C · 构建：CMake 交叉编译

## 一句话

在一块触屏板上做了一台小游戏机：登录、2048、切水果、网络排行榜、系统设置，全部跑在 framebuffer 上。

## 功能

- **登录页**：`lv_keyboard` 输入用户名/密码
- **2048**：滑动方向识别、合并计分、胜负判定、数字渐变配色
- **切水果**：线段-圆碰撞检测、重力运动、裂两半 + 粒子
- **网络排行榜**：客户端 TCP 上报成绩，服务端一连接一线程，排行落文件
- **系统设置**：亮度（sysfs）、音量/音乐（mplayer）

## 底层怎么接的

| 能力 | 做法                                                         |
| ---- | ------------------------------------------------------------ |
| 显示 | `/dev/fb0` + `FBIOGET_VSCREENINFO` + `mmap`，把 framebuffer 交给 LVGL 的 flush 回调 |
| 触摸 | `/dev/input/eventX` 读 `input_event`                         |
| 背光 | sysfs 的 `brightness` 节点                                   |
| 音频 | `mplayer -slave` + 命名管道                                  |
| 网络 | `pthread` + `mutex`；客户端设 `SO_RCVTIMEO` 防阻塞           |

## 界面管理（关键设计）

所有界面对象都由容器统一管理：切页时只销毁/隐藏容器，不逐个删控件，避免野指针。
登录页的键盘挂在容器上，切走后由容器统一回收。

## 踩过的坑

1. LVGL 对象在切页后野指针崩溃 → 改成容器托管 + 统一销毁。
2. socket 阻塞把 UI 卡住 → 客户端加接收超时，服务端每连接一线程。
3. framebuffer 双缓冲没开时画面撕裂 → 只刷脏区域。

## 编译

```bash
cmake -B build -DCMAKE_TOOLCHAIN_FILE=toolchain-aarch64.cmake
cmake --build build
```

产物拷到板上运行；需先停掉 Weston（直接写 framebuffer）。
