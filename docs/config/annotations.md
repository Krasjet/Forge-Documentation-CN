`@Config`
=======

`@Config` 注解可以代替 `Configuration`. 

目录
-------------

* [基础][basics]
* [@Config 用法][configuse]
* [@Comment 用法][commentuse]
* [@Name 用法][nameuse]
* [@RangeInt 用法][rangeintuse]
* [@RangeDouble 用法][rangedoubleuse]
* [@LangKey 用法][langkeyuse]
* [@RequiresMcRestart 用法][requiresmcrestartuse]
* [@RequiresWorldRestart 用法][requiresworldrestartuse]
* [子类别][subcategories]
* [@Ignore][ignoreuse]

<a id="basics">基础</a>
------

用`@Config`注释的类将把任何字段变成配置选项。 可以使用`@Config`类中提供的过多注释来注释所述字段以添加信息。

!!! note "提示"

    枚举配置值生成的注释与枚举值一样长。

示例请见[`Forge's Test`][forgetest]

<a id="config-use">@Config 用法</a>
-----------

这个注解表示一个类里面有配置选项。

有4个参数:

|     参数 | 类型     |     默认值      | 描述                                                         |
| -------: | :------- | :-------------: | :----------------------------------------------------------- |
|    modid | `String` |       N/A       | 使用该配置的modid                                            |
|     name | `String` |      `""`       | 用户友好的配置文件名,默认为modid                             |
|     type | `Type`   | `Type.INSTANCE` | 配置的类型,现在只有一个值`Type.INSTANCE`。该参数为了对Forge控制文件向上兼容。 |
| category | `String` |   `"general"`   | 根元素类别。 如果这是一个空字符串，则禁用根类别。            |

!!! important "重要"

    如果你禁用根类型，你要创建[子类别][subcategories]，否则会报错。

<a id="comment-use">@Comment 用法</a>
------------

`@Comment` 用于注解字段，用于在配置文件中生成注释

它有一个参数:

|     参数 | 类型     |     默认值      | 描述                                                         |
|---------:|:--------------------|:-------------:|:------------------------------------------------------------------------------------------------------------------------------|
|    value | `String[]`/`String` |      N/A      | 可以通过`String[]`传值,并且每一个元素会生成新的一行。 |

### 示例
```java
@Comment({
  "你可以这样增加注释",
  "数组可以创建多行注释"
})
public static boolean doTheThing = true;
```
会生成如下配置:
```
# 你可以这样增加注释
# 数组可以创建多行注释
B:doTheThing=true   
```

<a id="name-use">@Name 用法</a>
---------

 `@Name`注解用于在配置文件中生成用户友好的名称。

它有1个参数:

|     参数 | 类型     |     默认值      |
|---------:|:---------|:-------------:|
|    value | `String` |      null      |

### 示例
```java
@Name("FE/T for the thing")
public static int thingFE = 50; 
```
会生成如下配置:
```
I:"FE/T for the thing"=50
```

<a id="rangeint-use">@RangeInt 用法</a>
-------------

你可以用`@RangeInt`限制配置中`int` 或`Integer`的取值范围。

它有2个参数:

|     参数 | 类型     |     默认值      |
|---------:|:------|:-------------------:|
|      min | `int` | `Integer.MIN_VALUE` |
|      max | `int` | `Integer.MAX_VALUE` |

### 示例
```java
@RangeInt(min = 0) 
public static int thingFECapped = 50;
```
它会生成如下配置:

```
# Min: 0
# Max: 2147483647
I:thingFECapped=50
```

<a id="rangedouble-use">@RangeDouble 用法</a>
----------------

你可以用`@RangeDouble`限制配置中`double` 或`Double `的取值范围。

它有2个参数:

|     参数 | 类型     |     默认值      |
|---------:|:---------|:------------------:|
|      min | `double` | `Double.MIN_VALUE` |
|      max | `double` | `Double.MAX_VALUE` |

### 示例
```java
@RangeDouble(min = 0, max = Math.PI) 
public static double chanceToDrop = 2;
```
它会生成如下配置:
```
# Min: 0.0
# Max: 3.141592653589793
D:chanceToDrop=2.0
```

!!! note "提示"

    现在(1.12.2中)没有`@RangedFloat` , `@RangedLong`或其它类型的限制值。

<a id="langkey-use">@LangKey 用法</a>
------------

如果你要在mod配置菜单中添加翻译,那么给字段加上`@LangKey`。

它有1个参数:

|     参数 | 类型     |     默认值      |
|---------:|:---------|:-------------:|
|    value | `String` |      null      |

<a id="requiresmcrestart-use">@RequiresMcRestart 用法</a>
----------------------
`@RequiresMcRestart`注解用于必须要重启才能生效的字段。

### 示例
```java
@RequiresMcRestart
public static boolean overlayEnabled = false;
```
这会导致在配置菜单里更改该值之后,游戏必须重启。

<a id="requiresworldrestart-use">@RequiresWorldRestart 用法</a>
-------------------------

加上该注解,如果在mod配置菜单中更改该值,世界会强制重启。

### 示例
```java
@RequiresWorldRestart
public static boolean someOtherworldlyThing = false;
```
如果在mod配置菜单中更改该值,世界会强制重启。

<a id="subcategories">子类别</a>
--------------
子类别是一种把一组相关的配置放在一起的方式，可以更好的组织你的配置文件。要创建一个子类别，要把一个对象作为静态字段加入到主类别类中。该对象的成员变量会变成子类型的配置。

一个设置子类型的示例:
```java
@Config(modid = "modid")
public class Configs {
  public static SubCategory subcat = new SubCategory();

  private static class SubCategory {
    public boolean someBool; 
    public int relatedInt;
  }
}
```
在配置文件中,会是这样:
```
subcat {
  B:someBool=false
  I:relatedInt=0
}
```

<a id="ignore-use">@Ignore 用法</a>
-----------
加上在配置类的字段中加上`@Ignore`注解之后， `ConfigManager` 在解析你的配置文件时会跳过该字段。

!!! note "提示"

    只在 forge 版本 >= 1.12.2-14.23.1.2602 有效, 因为这个功能在[这次更新][ignoreupdate]中加入

[basics]: #basics
[configuse]: #config-use
[commentuse]: #comment-use
[nameuse]: #name-use
[rangeintuse]: #rangeint-use
[rangedoubleuse]: #rangedouble-use
[langkeyuse]: #langkey-use
[requiresmcrestartuse]: #requiresmcrestart-use
[requiresworldrestartuse]: #requiresworldrestart-use
[ignoreuse]: #ignore-use
[forgetest]: https://github.com/MinecraftForge/MinecraftForge/blob/603903db507a483fefd90445fd2b3bdafeb4b5e0/src/test/java/net/minecraftforge/debug/ConfigTest.java
[ignoreupdate]: https://github.com/MinecraftForge/MinecraftForge/commit/ca7a5eadc05c427a21fb7ae745e5fd9a5d906267
[subcategories]: #sub-categories
