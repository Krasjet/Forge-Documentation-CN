动画状态机文件
===========

动画状态机 (ASM) 文件是动画API的核心. 它们定义了动画的执行方式以及如何使用骨架文件中定义的剪辑。

概念
----------

ASM包含_参数_，_剪辑_，_状态_和_转换_(parameters, clips, states 和transitions)。

### 状态(States )

动画_状态_ 机可以在许多不同的_状态_中。 您可以在状态部分中定义哪些状态。

### 转换(Transitions)

转换定义允许哪些状态进入其他状态，例如允许“关闭”状态进入“打开”状态。

!!! note "提示"
	
	但是，转换 _不会_ 定义状态之间播放的动画。 如果要这样做，则必须创建一个播放动画的附加状态，然后使用事件转到下一个状态。

### 参数(Parameters)

!!! note "提示"
    
    参数在代码中称为`TimeValues`，因此这是SomethingValue的命名约定。

所有参数都采用输入，通常以秒为单位的当前游戏时间作为浮点数（考虑特定tick）并输出另一个时间。 此输出用作剪辑的输入，告诉它当前动画的进展。

每个参数都可以在ASM中定义，也可以在代码中加载ASM时定义。 加载时参数通常是`VariableValue`类型，它返回一个代码内可变的值，忽略其输入。
其他类型允许你对输入进行数学运算（`SimpleExprValue`），返回一个常量（`ConstValue`），引用其他参数（`ParameterValue`），返回
输入unmodified（`IdentityValue`）并执行两个参数的组合（`CompositionValue`）。

### 剪辑(Clips)

!!! note "提示"
    
    剪辑可以是ASM剪辑，在ASM中定义的剪辑，也可以是电枢剪辑，在骨架文件中定义的剪辑。
    对于本页的其余部分，除非另有说明，否则“剪辑”将引用ASM剪辑。

剪辑接收输入，通常是时间，并使用它对模型执行某些操作。 不同类型的剪辑做不同的事情，最简单的是动画电影剪辑（`ModelClip`）。 您还可以覆盖另一个ASM剪辑的输入（`TimeClip`）。如果输入为正，则在动画另一个剪辑时触发事件（`TriggerClip`）。在两个剪辑之间平滑混合（`SlerpClip`）。 在ASM参考另一个剪辑（`ClipReference`）或什么也不做（`IdentityClip`）。

### 事件(Events)

Various things can trigger events in the ASM. Events in the ASM are represented using only text.
Some events are special, with text that is formatted like this: `!event_type:event_value`. Right now there is only one kind of `event_type`, namely `transition`. This tries to transition to whatever state is defined in the `event_value`. Anything else is a normal event and can be used from the `pastEvents` callback, but more information about that is on the [implementing][] page.

各种东西可以触发ASM中的事件。 ASM中的事件仅使用文本表示。
有些事件很特殊，文本格式如下：`!event_type:event_value`。 现在只有一种`event_type`，即`transition`。 这会尝试转换为`event_value`中定义的任何状态。 其他任何东西都是正常事件，可以从`pastEvents`回调中使用，但有关它的更多信息在[使用API][implementation]页面上。


编程API
----------

!!! warning "警告"

    ASM代码API只能用于 _客户端_ 。 在代码中存储ASM时，请使用一端不可知的“IAnimationStateMachine”接口。

可以通过调用`ModelLoaderRegistry.loadASM`来加载ASM。 它需要两个参数，第一个是`ResourceLocation`表示
存储ASM的位置，以及第二个加载时定义参数的`ImmutableMap`。

一个例子:
```java
@Nullable
private final IAnimationStateMachine asm;
private final VariableValue cycle = new VariableValue(4);

public Spin() {
     asm = proxy.loadASM(new ResourceLocation(MODID, "asms/block/rotatest.json"), ImmutableMap.of("cycle_length", cycle));
}
```

在这里，使用一个名为`cycle_length`的额外参数加载ASM（用端代理SidedProxy以避免在服务器上崩溃）。 这个参数的类型是`VariableValue`，所以我们可以从我们的代码中设置它。

使用ASM实例，您可以使用`.currentState()`获取当前状态，并使用`.transition(nextState)`转换到另一个状态。

`VariableValue`参数可以通过调用`.setValue`来设置它们的值，但是你不能读回这个值。 无需通知ASM此更改，它会自动更改。

文件格式
-------------

ASM存储在json文件中。 位置无关紧要，但它们通常放在`asms`文件夹中。

首先是一个例子:
```json
{
  "parameters": {
    "anim_cycle": ["/", "#cycle_length"]
  },
  "clips": {
    "default": ["apply", "forgedebugmodelanimation:block/rotatest@default", "#anim_cycle" ]
  },
  "states": [
    "default"
  ],
  "transitions": {},
  "start_state": "default"
}
```

如上所述，文件具有参数，剪辑，状态和转换，以及ASM的起始状态。

所有这些标签都是必需的，即使它们是空的。

### 参数(Parameters)

```javascript
{
    "name": <parameter_definition>
}
```

`<parameter_definition>`不同类型的参数有不同的格式，简单的参数是：

- `IdentityValue`:字符串`#identity`,
- `ParameterValue`: 要引用的参数，前缀为`#`，例如`＃my_awesome_parameter`
- `ConstValue`: 一个数字用作返回的常量

#### 数学表达式 (`SimpleExprValue`)

格式: `[ regex("[+\\-*/mMrRfF]+"), <parameter_definition>, ... ]`

##### 示例:
```json
[ "+", 4 ]
[ "/+", 5, 1]
[ "++", 2, "#other" ]
[ "++", "#other", [ "compose", "#cycle", 3] ]
```

##### 说明

`SimpleExprValue`获取其输入并对其应用操作。
第一个参数是要应用的操作序列，其余参数表示这些操作的操作数。每个操作的输入是整个参数的输入（对于第一个操作）或前一个操作的结果。

##### 操作(区分大小写):

| 操作 | 含义 |
| --- | --- |
| `+` | `输出 = 输入 + 参数` |
| `-` | `输出 = 输入 - 参数` |
| `*` | `输出 = 输入 * 参数` |
| `/` | `输出 = 输入 / 参数` |
| `m` | `输出 = min(输入, 参数)` |
| `M` | `输出 = max(输入, 参数)` |
| `r` | `输出 = floor(输入 / 参数) * 参数` (向下取整) |
| `R` | `输出 = ceil(输入 / 参数) * 参数`  (向上取整) |
| `f` | `输出 = 输入 - floor(输入 / 参数) * 参数`  (取余) |
| `F` | `输出 = ceil(输入 / 参数) * 参数 - 输入`  (参数减余数) |

##### 示例说明:
- 输入 + 4
- (输入 / 5) + 1
- 输入 + 2 +参数 `other`的值
- 输入 + 参数`other` 的值+ 参数 `cycle`的值 赋值为 3

#### 功能组件 (`CompositionValue`)

格式: `[ "compose", <parameter_definition>, <parameter_definition> ]`

##### 示例:
```json
[ "compose", "#cycle", 3]
[ "compose", "#test", "#other"]
[ "compose", [ "+", 3], "#other"]
[ "compose", [ "compose", "#other2", "#other3"], "#other"]
```
##### 说明

`CompositionValue`将两个 参数定义 作为输入，并执行`value1(value2(input))`。 换句话说，它连接两个输入，用给定的输入调用第二个函数，用第二个输出调用第一个函数。

##### 示例说明:
- `cycle(3)`
- `test(other(输入))`
- `3 + other(输入) `
- `other2(other3(other(输入)))` 因为 `value1` = `other2(other3(输入))` ，`value2` = `other(输入)`

### 剪辑(Clips)

```javascript
{
    "name": <clip_definition>
}
```

与参数一样，不同类型的剪辑<clip_definition>`格式不同，但简单的剪辑是：

- `IdentityClip`: 字符串 `#identity`
- `ClipReference`: 要引用的剪辑名，前缀为`#`，例如`＃my_amazing_clip`
- `ModelClip`: 模型资源位置+`@`+骨骼剪辑的名称，例如 `mymod:block/test@default` ，`mymod:block/test#facing=east@moving`

#### 覆盖输入 (`TimeClip`)

格式: `[ "apply", <clip_definition>, <parameter_definition> ]`

##### 示例:
```json
["apply", "mymod:block/animated_thing@moving", "#cycle_time"]
["apply", [ "apply", "mymod:block/animated_thing@moving", [ "+", 3 ] ], "#cycle"]
```

##### 说明

`TimeClip`采用另一个剪辑并使用自定义参数而不是当前时间来调用它。 通常用于使用参数而不是当前时间来调用`ModelClip`。

##### 示例说明:

- `mymod:block/animated_thing@moving(#cycle_time)`
- `mymod:block/animated_thing@moving(#cycle + 3)`

#### 触发事件 (`TriggerClip`)

格式: `[ "trigger_positive", <clip_definition>, <parameter_definition>, "<event_text>"]`

##### 示例

```json
[ "trigger_positive", "#default", "#end_cycle", "!transition:moving" ]
[ "trigger_positive", "mymod:block/animated_thing@moving", "#end_cycle", "boop" ] 
```

##### 说明

`TriggerClip`在看起来上相当于`TimeClip`，但当`parameter_description`变为正时，也会在`event_text`中触发事件。
同时，它将`clip_definition`中的剪辑应用于相同的`parameter_description`。

##### 示例说明

- 在给定参数`end_cycle`的输入的情况下应用名为`default`的剪辑，当`end_cycle`为正变为到`moving`状态
- 在给定参数`end_cycle`的输入的情况下应用名为`mymod:block/animated_thing@moving`的剪辑，当`end_cycle`为正触发`boop`事件

#### 两个剪辑之间的混合 (`SlerpClip`)

格式: `[ "slerp", <clip_definition>, <clip_definition>, <parameter_definition>, <parameter_definition> ]`

##### 示例

```json
[ "slerp", "#closed", "#open", "#identity", "#progress" ]
[ "slerp", [ "apply", "#move", "#mover"], "#end", "#identity", "#progress" ]
```

##### 说明

`SlerpClip`在两个独立的剪辑之间执行球形线性混合(spherical linear blend)。 换句话说，它会将一个剪辑平滑地变换为另一个剪辑。

两个`clip_definition`分别是要混合的剪辑。 第一个`parameter_definition`是“输入”。 from和to剪辑都以当前动画时间传递此参数的输出。 第二个`parameter_definition`是“progress”，一个介于0和1之间的值，表示我们在混合中的距离。 将此剪辑与trigger_positive和转换特殊事件组合可以允许两个固态之间的简单转换。

###### 示例说明

- 将“关闭”剪辑混合到“打开”剪辑中，为两个剪辑提供未更改的时间作为输入并混合进度`#progress`。
- 当输入参数`mover`给定结束剪辑时，将移动剪辑的结果混合，其中未改变的时间作为混合进度`#progress`的输入。

### 状态(States)

状态部分只是所有可能状态的列表。

例如

```json
"states": [
  "open",
  "closed",
  "opening",
  "closing",
  "dancing"
]
```
定义5 种状态: open, closed, opening, closing 和 dancing.

### 转换(Transitions)

The transitions section defines which states can go to what other states. A state can go to 0, 1, or many other states.
To define a state as going to no other states, omit it from the section. To define a state as going to only one other state, create a key with the value of the state it can go to, for example `"open": "opening"`. To define a state as going to many other states, do the same as if it were going to only one other state but make the value a list of all possible recieving states instead, for example: `"open": ["closed", "opening"]`.

转换部分定义哪些状态可以转到其他状态。 状态可以进入0,1或许多其他状态。
若要一个状态不能进入其他状态，请从该部分中省略它。 若要一个状态仅能转到另一个状态，请创建一个具有其可以进入的状态值的键，例如`"open": "opening"`。 要将状态定义为转到许多其他状态，请执行相同的操作，就好像它只转到另一个状态，而是将值作为所有可能的接收状态的列表，例如：`"open": ["closed", "opening"]`。

一个完整的例子:

```json
"transitions": {
  "open": "closing",
  "closed": [ "dancing", "opening" ],
  "closing": "closed",
  "opening": "open",
  "dancing": "closed"
}
```

这个例子是说:

- open状态可以进入closing状态
- closed状态可以进入dancing和opening状态
- closing状态可以进入closed状态
- opening状态可以进入open状态
- dancing状态可以进入closed状态

[implementing]: implementing.md
