## Blockly学习——定义块（define blocks)

> 定义块是blockly开发最重要的环节，可以利用官方工具：[Blockly Developer Tools](https://blockly-demo.appspot.com/static/demos/blockfactory/index.html) ，也可以自己定义，本文记录整理通过代码定义方法，原网址为：
>
> https://developers.google.com/blockly/guides/create-custom-blocks/define-blocks

---

####  JSON vs JavaScript

- 有两种定义 block 的方式: JSON objects 和 JavaScript functions。

- 一般来说 JSON 形式更推荐，但是有些高级特性 JSON 不能满足。
- 可以通过 `InitJson` 函数实现混合使用。

```javascript
var mathChangeJson = {
  "message0": "change %1 by %2",
  "args0": [
    {"type": "field_variable", "name": "VAR", "variable": "item", "variableTypes": [""]},
    {"type": "input_value", "name": "DELTA", "check": "Number"}
  ],
  "previousStatement": null,
  "nextStatement": null,
  "colour": 230
};

Blockly.Blocks['math_change'] = {
  init: function() {
    this.jsonInit(mathChangeJson);
    // Assign 'this' to a variable for use in the tooltip closure below.
    var thisBlock = this;
    this.setTooltip(function() {
      return 'Add a number to variable "%1".'.replace('%1',
          thisBlock.getFieldValue('VAR'));
    });
  }
};
```

#### Block 颜色

主题颜色通过 JSON `colour` 属性或者`block.setColour(..)` 来设置。

#### 连接状态

- 向下连接型![](https://developers.google.com/blockly/images/set-next-statement.png)

```json
// Untyped:
{
    ...,
    "nextStatement": null,
}
// Typed:
{
  "nextStatement": "Action",
  ...
}
```

```javascript
// Untyped:
this.setNextStatement(true);  // false implies no next connector, the default

// Typed:
this.setNextStatement(true, 'Action');
```

- 向上连接型![](https://developers.google.com/blockly/images/set-previous-statement.png)

```json
// Untyped:
{
  ...,
  "previousStatement": null,
}
// Typed:
{
  "previousStatement": "Action",
  ...
}
```

```javascript
// Untyped:
this.setPreviousStatement(true);  // false implies no previous connector, the default

// Typed:
this.setPreviousStatement(true, 'Action');
```

#### 块的输出

- 带有输出的块用一个公口的拼图表示，可以连接输入值的口，这种块又被叫做 `value blocks` ![](https://developers.google.com/blockly/images/set-output.png)

```json
// Untyped:
{
  ...,
  "output": null,
}
// Typed:
{
  "output": "Number",
  ...
}
```



```javascript
//Untyped:
init: function() {
  // ...
  this.setOutput(true);
}

//Typed:
init: function() {
  // ...
  this.setOutput(true, 'Number');
}
```

#### 块的输入

存在三种类型的输入：![](https://developers.google.com/blockly/images/input-types.png)

- Value input：用于连接`value block`的输出接口
- Statement input：用于连接状态块 (statement block) 的向上接口。比如while块的执行模块。
- Dummy input：不连接块，用于额外的值的输入。

  ###### Json 格式`input` 和 `fields`

JSON通过带插值（1%, 2%）的消息字符串(message0, message1,...)来定义输入。![](https://developers.google.com/blockly/images/variables-set.png)

```json
{
  "message0": "set %1 to %2",
  "args0": [
    {
      "type": "field_variable",
      "name": "VAR",
      "variable": "item",
      "variableTypes": [""]
    },
    {
      "type": "input_value",
      "name": "VALUE"
    }
  ]
}
```

插值的顺序和声明的`type` 会决定块最终的结构，比如将`set %1 to %2` 改为 `put %2 in %1` 。块会变成：![](https://developers.google.com/blockly/images/variables-put.png)

每一个对象都有一个`type` 属性，剩下的参数取决于`type` :

- Fields:
  - field_input
  - field_dropdown
  - field_checkbox
  - field_colour
  - field_number
  - field_angle
  - field_variable
  - field_date
  - field_label
  - field_image
- Inputs:
  - input_value
  - input_statement
  - input_dummy

每一个对象还有一个`alt` 属性，比如说有一个新的`field` ，使用这个`field` 的块就可以用`alt` 定义`field input` fallback。

```json
{
  "message0": "sound alarm at %1",
  "args0": [
    {
      "type": "field_time",
      "name": "TEMPO",
      "hour": 9,
      "minutes": 0,
      "alt":
        {
          "type": "field_input",
          "name": "TEMPOTEXT",
          "text": "9:00"
        }
    }
  ]
}
```

`dummy input` 可以不用在argsN里声明。

多行可以用多个`message` ![](https://developers.google.com/blockly/images/repeat.png)

```json
{
  "type": "controls_repeat_ext",
  "message0": "repeat %1 times",
  "args0": [
    {"type": "input_value", "name": "TIMES", "check": "Number"}
  ],
  "message1": "do %1",
  "args1": [
    {"type": "input_statement", "name": "DO"}
  ],
  "previousStatement": null,
  "nextStatement": null,
  "colour": 120
}
```

###### js格式`input` 和 `fields`

JS为每一个输入类型提供`append` 方法![](https://developers.google.com/blockly/images/append-input.png)

```json
this.appendDummyInput()
    .appendField('for each')
    .appendField('item')
    .appendField(new Blockly.FieldVariable());
this.appendValueInput('LIST')
    .setCheck('Array')
    .setAlign(Blockly.ALIGN_RIGHT)
    .appendField('in list');
this.appendStatementInput('DO')
    .appendField('do');

```

每个方法带有一个`identifier string` 供`code generators` 使用，`Dummy inputs` 不需要。

`setCheck` 用于检查输入类型，没有的话可以连接任何块。

```Javascript
input.setCheck('Number');
```

`setAlign` 用于设置`fields` 对齐，参数有：`Blockly.ALIGN_LEFT`  `Blockly.ALIGN_RIGHT` `Blockly.ALIGN_CENTRE` 默认左对齐。

`appendField` ,一个block被用`appendInput` 赋予输入时，可以被赋予多个fileds，fields经常用作描述input的标签。

![](https://developers.google.com/blockly/images/append-field.png) 最简单的filed元素是text。

```javascript
input.appendField('hello');
```

<img src="https://developers.google.com/blockly/images/append-field-label.png"  /> 需要指定fileds类名，通过`CSS` 定义风格时用

`appendField(new Blockly.FieldLabel('hello'))`

```javascript
input.appendField('hello')
     .appendField(new Blockly.FieldLabel('Neil', 'person'));
```

#### Fields

`Fileds` 定义块的UI，包括字符串标签，图像和输入文本数值。最简单的`math_number` 块，用`field_input` 让用户输入一个数。![](https://developers.google.com/blockly/images/math-number.png)

Blockly提供很多内建fields（text inputs, color pickers, images)。也可以创建自己的fields。

#### Tooltips

帮助提示，可以是静态的字符串，

```javascript
init: function() {
  this.setTooltip("Tooltip text.");
}
```

也可以是动态的，根据下拉框选项选择。

```javascript
Blockly.Blocks['math_arithmetic'] = {
  init: function() {
    // ...

    // Assign 'this' to a variable for use in the tooltip closure below.
    var thisBlock = this;
    this.setTooltip(function() {
      var mode = thisBlock.getFieldValue('OP');
      var TOOLTIPS = {
        'ADD': Blockly.Msg.MATH_ARITHMETIC_TOOLTIP_ADD,
        'MINUS': Blockly.Msg.MATH_ARITHMETIC_TOOLTIP_MINUS,
        'MULTIPLY': Blockly.Msg.MATH_ARITHMETIC_TOOLTIP_MULTIPLY,
        'DIVIDE': Blockly.Msg.MATH_ARITHMETIC_TOOLTIP_DIVIDE,
        'POWER': Blockly.Msg.MATH_ARITHMETIC_TOOLTIP_POWER
      };
      return TOOLTIPS[mode];
    });
  }
};
```

#### Change Listeners and Validators

主要用来设置block的⚠警告，或者用户通知。通过`setOnChange` 。

#### Mutator

mutator可以通过JSON同`mutator` key添加

```json
{
  // ...,
  "mutator":"if_else_mutator"
}
```

------

> 下面基本用不到

#### Per-block configuration

一些其他特性：

###### Deletable State

有这一条语句的表示不可删除：

```json
block.setDeletable(false);
```

###### Editable State

```json
block.setEditable(false);  // Web or Android
```

###### Movable State

#### Context Menus



