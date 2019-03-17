骨骼文件
==============

骨骼文件用于定义模型动画的关节和剪辑。

文件结构
----------------

一个骨骼文件示例，取自forge debug mod
```json
{
  "joints": {
    "stick": {"2": [1.0]},
    "cube": {"3": [1.0]}
  },
  "clips": {
    "default": {
      "loop": true,
      "joint_clips": {
        "stick": [
          {
            "variable": "offset_x",
            "type": "uniform",
            "interpolation": "linear",
            "samples": [0, 0.6875, 0]
          }
        ],
        "cube": [
          {
            "variable": "offset_x",
            "type": "uniform",
            "interpolation": "linear",
            "samples": [0, 0.6875, 0]
          },
          {
            "variable": "axis_z",
            "type": "uniform",
            "interpolation": "nearest",
            "samples": [ 1 ]
          },
          {
            "variable": "origin_x",
            "type": "uniform",
            "interpolation": "nearest",
            "samples": [ 0.15625 ]
          },
          {
            "variable": "origin_y",
            "type": "uniform",
            "interpolation": "nearest",
            "samples": [ 0.40625 ]
          },
          {
            "variable": "angle",
            "type": "uniform",
            "interpolation": "linear",
            "samples": [0, 120, 240, 0, 120, 240]
          }
        ]
      },
      "events": {}
    }
  }
}

```

该文件分为两部分，即关节和剪辑。

关节
--------
每个关节定义一个在动画中作为一个整体一起动的对象。 使用原本JSON模型，这意味着多个元素可以属于同一个关节。

格式如下
```javascript
 {
    "joints": {
        <joint>, ...
    }
}
    
---

<joint> ::= {
    <string>: {  // joint_name
        <joint_definition>, ...
    }
}

<joint_definition> ::= {
    "<index_model>": [ <float> ] // index_model, joint_weight (只能指定一个)
}

```

- `joint_name`是关节的名字
- `index_model`是一个0索引号（其中0是模型中定义的第一个元素），表示此关节控制的模型元素。 必须是一个字符串（参见示例）
- `joint_weight`是一个权重（0-1），表示如果在多个关节中使用该元素，该关节将对该元素的最终转换做出多少贡献。

!!! note

	对于更简单的动画，权重通常可以设置为1.0，但如果您希望剪辑中的多个关节以不同方式设置动画，可以用这种方法实现。

并非所有元素都需要关节，只有你运动的元素。
如果元素出现在多个关节中，则最终运动是每个关节的变换的加权平均值。

剪辑
-------

剪辑本质上是关于如何使用值来动画某些关节集合的说明。
它们还包括在某些点发射的事件。

格式如下:
```javascript

{
    "clips": {
        "clip_name": {
            "loop": <true/false>,
            "joint_clips": {
                <joint_clip_list>, ...
            },
            "events": {
                <event> ...
            }
            
        }
    }
}

-------

<joint_clip_list> ::= {
    "joint_name": [
        <joint_clip>, ...
    ]
}

<joint_clip> ::= {
    "variable": <variable>,
    "type": "uniform",
    "interpolation": <interpolation>,
    "samples": [ float, ... ]
}


```

- loop:如果为真，则当参数值超过1时，动画将会循环，否则它将仅限于最后的状态。

### Joint Clips
每个`joint_clip`是改变关节的一组变量。 `type`属性目前被忽略，但必须是`uniform`。

`samples`定义动画将采用什么值（类似于传统动画中的关键帧），它取决于`interpolation`的值解释。

`interpolation`，即如何将`samples`列表转换为（尽可能）连续动画，可以是以下之一：

 - `nearest` - 如果值<0.5，则使用第一个samples，否则使用第二个samples。 如果只给出一个值，则对静态变量很有用。
 - `linear` - 在samples之间线性插值。 samples之间的时间是1 /samples数.

`variable` 可以是下面之一:

- `offset_x`, `offset_y`, `offset_z` - 平移
- `scale` - 均匀缩放
- `scale_x`, `scale_y`, `scale_x` - 延坐标轴缩放
- `axis_x`, `axis_y`, `axis_z` - 旋转轴
- `angle` - 旋转角度
- `origin_x`, `origin_y`, `origin_z` - 旋转起点(rotation origin)

### Events

每个剪辑都可以触发事件，格式如下：
```javascript
<event> :: {
    <event_time>: "event_text"
}
```
有关事件和`event_text`含义的更多信息，请参阅[动画状态机][asm]页面。

`event_time`是一个表示何时触发事件的值（通常介于0和1之间）。 当控制此剪辑的参数达到等于或大于`event_time`的点时，将触发该事件。

[asm]: asm.md
