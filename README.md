# EventBinder

事件绑定模块：按构造参数把一个模块的事件映射到另一个模块的事件 / Event binding Module that maps the events of one Module to the events of another according to its constructor parameters

本仓库派生自 [QDU-Robomaster/EventBinder](https://github.com/QDU-Robomaster/EventBinder)（Apache-2.0），改动见 `NOTICE`。

This repository is derived from [QDU-Robomaster/EventBinder](https://github.com/QDU-Robomaster/EventBinder) (Apache-2.0); see `NOTICE` for the changes.

## 1. 模块作用 / Purpose

EventBinder 是纯粹的连接件：构造时它把每条绑定翻译成一次 `target_event->Bind(*source_event, source_event, target_event)`，源事件激活时目标事件跟着激活。之后它不再参与运行，没有线程、没有 Topic。

绑定由构造参数给出：

- `modules`：可参与绑定的模块列表，每项是 `{名称, 模块实例}`。名称是配置里的 `id`，模块实例必须提供 `GetEvent()`。
- `event_binding_groups`：绑定的分组，每项是一条 `{源名称, 源事件 ID, 目标名称, 目标事件 ID}`。事件 ID 可以是任意枚举，构造时统一转成 `uint32_t`，所以可以直接写 `Fsi6x::SwitchPos::...` 和 `CMD::Mode::...`。

源或目标名称在 `modules` 里找不到时，该条绑定被**静默跳过**。名字写错不会编译报错、也不会运行时报错，只是不生效 —— 配置里改了模块 `id` 记得同步这里。

EventBinder is a pure connector: at construction it translates every binding into one `target_event->Bind(*source_event, source_event, target_event)`, so activating the source event also activates the target. It takes no further part at runtime — no thread, no Topic.

Bindings come from the constructor parameters:

- `modules`: the Modules that can take part, each `{name, module}`. The name is the `id` from the configuration, and the Module must provide `GetEvent()`.
- `event_binding_groups`: groups of bindings, each `{source name, source event ID, target name, target event ID}`. An event ID may be any enumerator and is converted to `uint32_t`, so `Fsi6x::SwitchPos::...` and `CMD::Mode::...` can be written directly.

A binding whose source or target name is not found in `modules` is **silently skipped**. A wrong name is neither a compile error nor a runtime error, it just does nothing — remember to update these when a Module `id` changes.

## 2. 构造接口 / Constructor

```cpp
EventBinder(std::initializer_list<ModuleInfo> modules,
            std::initializer_list<BindingGroup> event_binding_groups = {});
```

嵌套结构（配置里写成 YAML 字符串时，括号层数必须对）：

Nested structures (the brace depth must match when written as a YAML string):

```
{ { {"源", 源事件, "目标", 目标事件}, ... } }   // 一条绑定 / one binding
{ { ... } }                                     // 一组 / one group
{ ... }                                         // 全部组 / all groups
```

## 3. Topic

无 / None

EventBinder 只调用 `LibXR::Event::Bind()`，本身不创建任何 Topic。

EventBinder only calls `LibXR::Event::Bind()` and creates no Topic of its own.

## 4. 配置示例 / Configuration Example

把 Fsi6x 的左拨杆接到 CMD 的控制模式上：拨到低位切自动控制。

Wire the left switch of Fsi6x to the control mode of CMD: flipping it low selects automatic control.

```yaml
modules:
  - module: starpang888/CMD
    id: cmd
  - module: starpang888/Fsi6x
    id: fsi6x
    args:
      - uart: usart3
      - cmd: cmd
  - module: starpang888/EventBinder
    id: event_binder
    args:
      - modules: '{{"fsi6x", fsi6x}, {"cmd", cmd}}'
      - event_binding_groups: '{{{{"fsi6x", Fsi6x::SwitchPos::FSI6X_SW_L_POS_BOT, "cmd", CMD::Mode::CMD_AUTO_CTRL}}}}'
```

EventBinder 必须排在它引用的所有模块**之后**。

EventBinder must be listed **after** every Module it references.

## 5. 依赖与硬件 / Dependencies and Hardware

模块依赖 / Module dependencies：

无 / None

EventBinder 是模板化的，只用到 `LibXR::Event`，不依赖任何具体模块类型。上游版本 `#include` 了 `CMD.hpp` 和 `DR16.hpp` 并据此声明依赖，但两个类型在类里从未被使用；本派生版本已删除。

EventBinder is generic and only uses `LibXR::Event`, depending on no concrete Module type. The upstream version included `CMD.hpp` and `DR16.hpp` and declared dependencies on them, yet neither type was used anywhere in the class; this derivation drops both.

硬件 / Hardware：

无 / None
