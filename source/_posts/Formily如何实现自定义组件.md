---
title: Formily如何实现自定义组件
date: 2021-05-28 13:02:39
tags:
---
>注：本文使用的Formily为v1版本，21年4月中旬阿里团队已经推出v2版本 https://v2.formilyjs.org/ ，但是跨度非常大，为`Breaking change`，相当于框架底层完全重构，对于v1版本的项目来说迁移成本非常大。

## 背景
### Formily是什么？
https://formilyjs.org/#/bdCRC5/dzUZU8il
这里引用官网的描述：
**Formily 是一个由阿里巴巴集团多部共建的面向中后台复杂场景的表单解决方案，它也是一个表单框架。**
### Formily实现了什么？
- 在复杂联动场景下更加清晰简单的描述联动的方式
- 在超多表单项场景下可以获得更好的表单操作性能
- 在跨终端场景下实现通用表单解决方案


### Formily组件说明
`Formily v1`提供给了我们3种`React`的`UI`框架

以下这些`UI`框架经过`formily`的二次封装，UI组件经过封装后，完美兼容了`Formily`

3种`UI`框架分别为：
- `Antd`
- `Next`
- `Meet`


[😊 Antd的Formily API文档](https://formilyjs.org/#/zoi8i0/6dt3t7FjI4)


官方文档中并没有介绍基于`JSON Schema`使用自定义组件的方法，但是给出了
1.基于`JSX`表单的自定义组件制作方法， [理解表单扩展机制](https://formilyjs.org/#/0yTeT0/vVSBSNhmHr)（虽然文档依旧欠缺，不过库的`TypeScript`的类型做的还算很不错的，相当于一份文档吧）
2.[实现超复杂自定义组件](https://formilyjs.org/#/0yTeT0/A1T4TPhXUZ)，但是该文档并不是很实用，由于我们是`JSON Schema`开发模式，用`useFromEffects`来控制`state`显然不现实

## 制作自定义组件
>这里以基于`Antd`二次封装`Radio`业务组件举例
### 在容器中引入自定义组件

首先，为了使用`JSON Schema`模式，我们使用`SchemaForm`组件。
```jsx
<SchemaForm
    components={components}
    onSubmit={onSubmit}
    expressionScope={createRichTextUtils(history)}
    schema={formData.schema}
    onChange={onCheckCanClickNextStep}
    actions={actions}
    onValidateFailed={onValidateFailed}
>
</SchemaForm>
```
可以注意到，`props`有一个属性为`components`，该属性代表了全部`JSON Schema`通过`x-component`可以使用的组件们。

`components`对象如下：

```jsx
import CustomRadio from '../../custom-components/CustomRadio/CustomRadio';
import 'xxx' from 'xxx'// 此处省略其他import.....

const components = {
    /** 自定义组件们 */
    CustomRadio,
    // others....
    /** 官方组件们 */
    Input,
    Radio: Radio.Group,
    Checkbox: Checkbox.Group,
    TextArea: Input.TextArea,
    NumberPicker,
    Select,
    Switch,
    DatePicker,
    DateRangePicker: DatePicker.RangePicker,
    YearPicker: DatePicker.YearPicker,
    MonthPicker: DatePicker.MonthPicker,
    WeekPicker: DatePicker.WeekPicker,
    TimePicker,
    TimeRangePicker: TimePicker.RangePicker,
    Upload,
    Range,
    Rating,
    Transfer,
    // others....
};
```

我们在对象中添加一个`CustomRadio`，至此成功引入了一个我们要自定义的组件

### Formily JSON Schema协议规范
在真正开始写组件逻辑之前，你需要了解`JSON Schema`的`Schema`规范，这一部分

实际上比如`Antd`的`Form`组件，大体构成为

```jsx
<Form>
    <Form.Item
        className="form-item"
        label="test_radio"
        name="test_radio"
        rules={[{ required: true }]}
    >
        <CustomRadio terminalValue={[ 1 ]} />
    </Form.Item>
</Form>
```

那么如何用`JSON`去描述这一行为呢？`Formily`约定的`Schema`如下：


```json
{
    "type": "object",
    "properties": {
    "test_radio": {
            "key": "test_radio",
            "type": "number",
            "title": "测试customRadio",
            "name": "test_radio",
            "required": true,
            "enum": [
                {
                    "label": "yes",
                    "value": 1
                },
                {
                    "label": "no",
                    "value": 0
                }
            ],
            "x-props": {
                "itemClassName": "form-item"
            },
            "x-component": "CustomRadio",
            "x-component-props": {
                "terminalValue": [
                    1
                ]
            },
            "default": 1
        },
    }
}
```
简单解释一下：
`x-props`表示`<Form.item />`的`props`
`x-component-props`表示表单`UI`组件的`props`


### 制作表单组件

> 为了方便，我们直接在`Antd`的`Radio`组件基础上封装我们的业务Radio组件

```jsx
// CustomRadio.jsx
import styles from './CustomRadio.module.scss';
import { Radio } from 'antd';
import { useEffect, useState } from 'react';

export default ({ onChange, terminalValue = [], dataSource = [], value, ...rest }) => {
    const [radioValue, setRadioValue] = useState(value);

    const onChangeRadioGroup = event => {
        const value = event.target?.value;

        /** 触发业务逻辑，可以做一些事情，比如弹窗，等等操作 */
        if (terminalValue.includes(value)) {
            // Do things
        }

        setRadioValue(value);
        /** 改变SchemaForm内部value */
        onChange(value);
    };

    return (
        <div className={styles.container}>
            <Radio.Group onChange={onChangeRadioGroup} value={radioValue}>
                {dataSource.map((radio, index) => (
                    <Radio key={index} value={radio.value}>
                        {radio.label}
                    </Radio>
                ))}
            </Radio.Group>
        </div>
    );
};
```
#### Formily为组件注入的Props项
在`Formily`框架中，在调用组件时，会自动注入几个属性
- `onChange`
- `onBlur`
- `onFocus`
- `value`
- `dataSource`
- 我们在`x-component-props`层定制的属性
还有其它的默认属性们

| 字段 | 作用 |
| -- | -- |
| onChange | 调用该方法才可以真正对`Form`实例对象上的对应`name`表单项的`value`进行修改 |
| value | 对应着JSON中的`default`字段改 |
| dataSource | 比如在Radio中，就是通过JSON描述的Enum值 |
| 自定义属性 | - |


#### 注入业务逻辑
我们主动去监听`Radio.Group`的`onChange`事件，当其`value`值满足我们的业务逻辑时，我们触发相关业务逻辑。

## 结语
至此，我们完成了一个最基础的业务组件。如果还有什么疑问的话，可以参考 [理解表单扩展机制](https://formilyjs.org/#/0yTeT0/vVSBSNhmHr) 。

吐槽：简单看了一下v2的介绍，Formily官方文档中描述的意思是，官方也意识到了v1版本组件扩展的问题，所以在v2版本完全重构了。