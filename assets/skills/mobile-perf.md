---
name: mobile-perf
title: 安卓端性能检查
description: 目标平台是安卓手机时，按清单检查帧率、内存、发热与包体，避免桌面级写法
---

# 安卓端性能检查

手机 GPU / 内存都比桌面紧张，同一个场景在 PC 上 120 帧、在中端机上可能 25 帧。
改动涉及渲染、粒子、物理、贴图时按下面清单自查。

## 渲染

- 优先 `Mobile` 渲染器，不要给移动项目配 `Forward+` 专属特性（体积雾、SSAO、SDFGI）。
- 2D 用 `CanvasItem` 批处理：同类贴图放同一 `TextureAtlas`，避免每帧打断合批。
- 少用 `CanvasLayer` 嵌套 + 全屏 `ColorRect` 叠加；透明层越多，overdraw 越重。
- 需要全屏后处理时改用单 Pass `CanvasLayer` shader，而不是多个 `BackBufferCopy`。
- 3D 场景贴图默认开 mipmap，压缩格式选 ETC2/ASTC。

## 脚本与节点

- `_process` / `_physics_process` 里不要 `get_node()`、不要字符串查找、不要 `instantiate()`。
- 高频查找在 `_ready()` 里缓存成 `@onready` 成员。
- 大量同类对象（子弹、特效）用对象池复用，不要频繁 `queue_free()` + `instantiate()`。
- 每帧只做必要计算：能按帧间隔的（AI 决策、检测）用 `Timer` 或计数器降频。
- 避免在 `_process` 中修改 UI 文本；数值没变化时不要赋值（`Label.text` 赋值会触发布局）。

## 物理

- 2D 碰撞形状优先 `CircleShape2D` / `RectangleShape2D`，避免 `ConvexPolygonShape2D` 高顶点数。
- 大量静态物体设为 `StaticBody2D` + 静态碰撞，不要用 `RigidBody2D`。
- `Area2D` 监测数量多时改用 `PhysicsServer2D` 直接查询或降频检测。
- 物理帧率（`physics/common/physics_ticks_per_second`）默认 60，不要随手调高。

## 内存与资源

- 大图按屏宽加载，不要直接把 4K 贴图塞进手机项目。
- 场景切换用 `ResourceLoader.load_threaded_request()` 异步预加载，避免主线程卡顿。
- 退出场景时断开信号、停止 `Tween` / `AnimationPlayer`，防止对象泄漏。
- 音频用 `AudioStreamOggVorbis` / `AudioStreamMP3`，不要用未压缩 WAV 存长音频。

## 包体

- 导出时开启「导出为 APK/AAB + 按架构拆分」，剔除未使用的插件。
- 只保留实际用到的 `export_presets.cfg` 平台。
- 资源重复（同一贴图多份拷贝）先用 `search_code` 查引用再删。

## 自查输出格式

发现性能问题时，按「问题 → 位置（文件:行号）→ 影响 → 改法」四段给结论，
不要只丢一段优化后的代码。
