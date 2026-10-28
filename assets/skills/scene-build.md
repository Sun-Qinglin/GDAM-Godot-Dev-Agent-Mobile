---
name: scene-build
title: 手写 .tscn 场景
description: 需要新建或重写 .tscn / .tres 文本场景时保证结构正确、能被 Godot 4 正常打开
---

# 手写 .tscn 场景

`.tscn` 是纯文本，手写时最常见的错误是「结构不合法」，Godot 打开后整棵树错位或脚本丢失。

## 必备骨架

```ini
[gd_scene load_steps=3 format=3 uid="uid://b1234567890ab"]

[ext_resource type="Script" path="res://src/player.gd" id="1_player"]
[ext_resource type="Texture2D" path="res://art/player.png" id="2_tex"]

[sub_resource type="CircleShape2D" id="CircleShape2D_1"]
radius = 12.0

[node name="Player" type="CharacterBody2D"]
script = ExtResource("1_player")

[node name="Sprite" type="Sprite2D" parent="."]
texture = ExtResource("2_tex")
position = Vector2(0, -8)

[node name="Collision" type="CollisionShape2D" parent="."]
shape = SubResource("CircleShape2D_1")
```

## 硬性规则

- 第一行必须是 `[gd_scene ... format=3]`；`.tres` 则是 `[gd_resource type="..." format=3]`。
- `load_steps` = 外部资源数 + 子资源数 + 1（写错通常仍能打开，但会报警告）。
- 每个 `[ext_resource]` 都要有**唯一**的 `id`，命名建议 `"1_xxx"`、`"2_xxx"`。
- 引用写法是 `ExtResource("1_player")` / `SubResource("CircleShape2D_1")`，**不要**直接写 `res://` 路径。
- 节点 path 用相对父节点的写法：根节点无 `parent`，直接子节点为 `parent="."`，孙节点为 `parent="Sprite"`。
- 属性名必须与 Godot 4 的属性一致（`texture`、`shape`、`position`、`scale`）。
- 改 uid 时要同步 `[gd_scene uid=...]` 与 `.godot/uid_cache.bin` 的引用场景；不确定就不要手写 `uid`，留空也合法。
- 中文字符串可以直接写，文件保持 UTF-8 无 BOM。
- 缩进不影响解析，但请统一用 Tab。

## 节点顺序

父节点必须出现在子节点之前。`parent="."` 的行若出现在父节点定义之前，Godot 会解析失败。

## 常见坑

- **脚本没挂上**：忘记在节点段写 `script = ExtResource("1_xxx")`。
- **碰撞体不生效**：`CollisionShape2D` 没有 `shape`，或形状资源的 `id` 和引用不一致。
- **UI 不显示**：`Control` 节点缺少 `anchors_preset` / `offset_*`，或父节点不是 `Control`/`Container`。
- **信号没连上**：`[connection signal="pressed" from="Button" to="." method="_on_button_pressed"]` 必须放在文件末尾，且 `from` 是节点路径。
