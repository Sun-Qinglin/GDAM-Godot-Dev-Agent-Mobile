---
name: gdscript-style
title: GDScript 代码风格
description: 新建或修改 .gd 脚本时套用的命名、类型注解、缩进与结构约定（Godot 4.x）
---

# GDScript 代码风格

## 命名

| 对象 | 规则 | 示例 |
| --- | --- | --- |
| 文件 / 场景节点 | `snake_case` | `player_controller.gd` |
| 类（`class_name`） | `PascalCase` | `class_name PlayerController` |
| 变量 / 函数 | `snake_case` | `move_speed`、`_apply_damage()` |
| 私有成员 | 前缀下划线 | `_is_grounded` |
| 常量 / 枚举值 | `CONSTANT_CASE` | `const MAX_HEALTH := 100` |
| 信号 | 过去式 `snake_case` | `signal health_changed(value: int)` |

## 类型与安全

- 所有 `var` / `func` 参数与返回值都写类型注解，不要依赖推断。
- 只在该被外部覆盖的地方写类型前缀：`@export var speed: float = 120.0`。
- 用 `:=` 做局部变量推断，用 `: Type =` 做显式声明；不要混用成 `var x: float := 0.0`。
- 用 `is` / `as` 做类型判断，不要用字符串比较 `get_class()`。
- 常量用 `const`，运行期才确定的值用 `@onready` 或 `_ready()` 里赋值。

## 结构顺序

脚本内成员顺序固定，便于速读：

1. `class_name` / `extends`
2. `signal`
3. `enum`
4. `const`
5. `@export` 变量
6. 普通成员变量
7. `@onready` 变量
8. `_init` / `_ready` / `_process` / `_physics_process` 等生命周期
9. 公开方法
10. 私有方法（`_` 前缀）

## 缩进与格式

- 缩进用 **Tab**（Godot 官方风格），不要用空格。
- 运算符两侧留空格：`var ratio := a / b`。
- 函数之间空一行，逻辑块之间空一行。
- 单行 `if` 只在提前返回时使用；其余情况加花括号块。

## 信号与解耦

- 子节点只发信号，不直接调用父节点的业务方法。
- 连接信号统一写在 `_ready()`，并在 `_exit_tree()` 里断开需要手动断开的连接。
- 用 `Callable` 而非字符串连接：`button.pressed.connect(_on_pressed)`。

## 反例（不要写）

```gdscript
# Godot 3 写法 ❌
export var speed = 100
onready var sprite = get_node("Sprite")
yield(get_tree().create_timer(1.0), "timeout")

# Godot 4 写法 ✅
@export var speed: float = 100.0
@onready var sprite: Sprite2D = $Sprite
await get_tree().create_timer(1.0).timeout
```
