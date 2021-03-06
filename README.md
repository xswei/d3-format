# d3-format

有没有注意到, 有时候 `JavaScript` 对数字的显示方式与预想的不一样, 比如下面这个简单的循环：

```js
for (var i = 0; i < 10; i++) {
  console.log(0.1 * i);
}
```

你会得到如下结果:

```js
0
0.1
0.2
0.30000000000000004
0.4
0.5
0.6000000000000001
0.7000000000000001
0.8
0.9
```

欢迎来到 [binary floating point](https://en.wikipedia.org/wiki/Double-precision_floating-point_format)! ಠ_ಠ

但是, 处理这种舍入问题并不是本模块出现的唯一原因. 一个表格中数字的格式应该一致, 以便进行比较；在上述例子中, `0.0` 要比 `0` 更容易进行对比. 大数值的数字应该进行单位分组(比如 `42,000`) 或者使用科学计数法(比如 `4.2e+4`, `42k`). 货币应该有固定的精确度(比如 `$3.50`). 有时候数值结果应四舍五入为有效数字(比如 `4021` 舍入为 `4000`). 数字格式应该适合于读者的地区(比如 `42.000,00` 和 `42,000.00`), 等等. 

将数值格式化为人类友好的格式是 `d3-format` 的目标, 这个模块仿制了 `Python 3` 的 [format specification mini-language](https://docs.python.org/3/library/string.html#format-specification-mini-language) ([PEP 3101](https://www.python.org/dev/peps/pep-3101/)). 重新查看上面的示例：

```js
var f = d3.format(".1f");
for (var i = 0; i < 10; i++) {
  console.log(f(0.1 * i));
}
```

Now you get this:

```js
0.0
0.1
0.2
0.3
0.4
0.5
0.6
0.7
0.8
0.9
```

但是 `d3-format` 的功能远远不止于类似于 [number.toFixed](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Number/toFixed)! 下面是个简单的例子:

```js
d3.format(".0%")(0.123);  // rounded percentage, "12%"
d3.format("($.2f")(-3.5); // localized fixed-point currency, "(£3.50)"
d3.format("+20")(42);     // space-filled and signed, "                 +42"
d3.format(".^20")(42);    // dot-filled and centered, ".........42........."
d3.format(".2s")(42e6);   // SI-prefix with two significant digits, "42M"
d3.format("#x")(48879);   // prefixed lowercase hexadecimal, "0xbeef"
d3.format(",.2r")(4223);  // grouped thousands with two significant digits, "4,200"
```

参考 [*locale*.format](#locale_format) 获取说明符的详细介绍, 运行 [d3.formatSpecifier](#formatSpecifier) 来理解说明符的含义.

## Installing

`NPM` 安装 `npm install d3-format`. 此外还可以从 [d3js.org](https://d3js.org) 以 [standalone library](https://d3js.org/d3-format.v1.min.js) 或作为 [D3 4.0](https://github.com/d3/d3) 的一部分下载或直接引用. 支持 `AMD`, `CommonJS`, 以及基本的标签引入形式. 如果使用标签引入, 会暴露全局 `d3` 变量:

```html
<script src="https://d3js.org/d3-format.v1.min.js"></script>
<script>

var format = d3.format(".2s");

</script>
```

区域文件托管在 [unpkg](https://unpkg.com/), 可以使用 [d3.json](https://github.com/d3/d3-request/blob/master/README.md#json) 去加载. 例如将默认区域设置为 `Russian`:

```js
d3.json("https://unpkg.com/d3-format@1/locale/ru-RU.json", function(error, locale) {
  if (error) throw error;

  d3.formatDefaultLocale(locale);

  var format = d3.format("$,");

  console.log(format(1234.56)); // 1 234,56 руб.
});
```

[在浏览器中测试 `d3-format`.](https://tonicdev.com/npm/d3-format)

## API Reference

<a name="format" href="#format">#</a> d3.<b>format</b>(<i>specifier</i>) [<源码>](https://github.com/d3/d3-format/blob/master/src/defaultLocale.js#L4 "Source")

[default locale](#formatDefaultLocale) 下 [*locale*.format](#locale_format) 的别名. 

<a name="formatPrefix" href="#formatPrefix">#</a> d3.<b>formatPrefix</b>(<i>specifier</i>, <i>value</i>) [<源码>](https://github.com/d3/d3-format/blob/master/src/defaultLocale.js#L5 "Source")

[default locale](#formatDefaultLocale) 下 [*locale*.formatPrefix](#locale_formatPrefix) 的别名.

<a name="locale_format" href="#locale_format">#</a> <i>locale</i>.<b>format</b>(<i>specifier</i>) [<源码>](https://github.com/d3/d3-format/blob/master/src/locale.js#L18 "Source")

根据指定的 *specifier* 字符串返回一个新的格式化函数. 返回的函数只接受数值一个参数, 并且返回经过格式化之后的数值. 一般的格式说明符如下：

```
[​[fill]align][sign][symbol][0][width][,][.precision][~][type]
```

填充字符 *fill* 可以是任何字符. 填充字符的位置由紧跟其后的 *align* 字符决定, 对齐字符必须是如下几个其中之一：

* `>` - 在可用空间内强制右对齐 (默认行为).
* `<` - 在可用空间内强制左对齐.
* `^` - 在可用空间内居中对齐.
* `=` - 与 `>` 类似, but with any sign and symbol to the left of any padding.

*sign* 可以是:

* `-` - 只有负数有减号, 而零和正数没有 (默认行为.)
* `+` - 零和正数前面有加号而负数前面有减号.
* `(` - 只有负数前面有括弧.
* ` ` (空格) - 零个正数前面有空格, 而负数前面是减号.

*symbol* 可以是:

* `$` - 与区域定义一致的货币符号.
* `#` - 对于二进制, 八进制或者十六进制的前缀表示, 前缀分别为 `0b`, `0o`, 或 `0x`.

*zero* (`0`) 选项用来表示启用 `0` 填充, 这隐式地将 `fill` 设置为 `0` 并对齐到 `=`. *width* 定义了最小字段宽度; 如果没有指定, 则宽度将由内容决定. *comma* (`,`) 选中表示启用分组分割, 比如使用逗号表示千分位.

根据 *type* 的不同, *precision* 要么指示小数点后的位数(类型为 `f` 和 `%` 时), 要么指示有效位数(类型为 `​`, `e`, `g`, `r`, `s` and `p` ). 如果没有指定 `precision`, 则除了 ` `（none） 以外的精度都默认为 `6`, ` `(none) 默认为 `12`. 在格式化整数时精度会被忽略(`b`, `o`, `d`, `x`, `X` 和 `c`). 参考 [precisionFixed](#precisionFixed) 和 [precisionRound](#precisionRound) 以帮助选取合适的精度.

`~` 选项用来在所有格式类型中删除无关紧要的尾随零. 通常与类型 `r`, `e`, `s` 和 `%` 一起使用. 例如: 

```js
d3.format("s")(1500);  // "1.50000k"
d3.format("~s")(1500); // "1.5k"
```

可选的 *type* 值如下:

* `e` - 指数符号.
* `f` - 定点符号.
* `g` - 十进制或指数记数法, 四舍五入到有效数字.
* `r` - 十进制记数法, 四舍五入到有效数字.
* `s` - 带 [SI 前缀](#locale_formatPrefix) 的十进制记数法, 四舍五入到有效数字.
* `%` - 乘以 100 然后带有百分号的十进制计数法.
* `p` - 乘以100, 四舍五入到有效数字, 然后是带百分号的十进制记数法.
* `b` - 二进制记数法, 四舍五入为整数.
* `o` - 八进制记数法, 四舍五入为整数.
* `d` - 八进制记数法, 四舍五入为整数.
* `x` - 十六进制记数法, 使用小写字母, 四舍五入为整数.
* `X` - 十六进制记数法, 使用大写字母, 四舍五入为整数.
* `c` - 在打印前将整数转换为相应的 `unicode` 字符.

`type` 为 `​` (none) 还可以作为 `~g` 的简写 (精度是 `12` 而不是 `6`), `type` 为 `n` 可以作为 `,g` 的简写. 对于 `g`, `n` 和 `​` (none) 类型, 如果结果字符串的精度或位数更少则使用十进制记数法; 否则使用指数计数法. 例如:

```js
d3.format(".2")(42);  // "42"
d3.format(".2")(4.2); // "4.2"
d3.format(".1")(42);  // "4e+1"
d3.format(".1")(4.2); // "4"
```

<a name="locale_formatPrefix" href="#locale_formatPrefix">#</a> <i>locale</i>.<b>formatPrefix</b>(<i>specifier</i>, <i>value</i>) [<源码>](https://github.com/d3/d3-format/blob/master/src/locale.js#L127 "Source")

等价于 [*locale*.format](#locale_format), 但是返回的函数会将值转化值的科学计数法形式, 然后再格式化为定点表示法, 支持以下[SI 前缀](https://en.wikipedia.org/wiki/Metric_prefix#List_of_SI_prefixes):

* `y` - yocto, 10⁻²⁴
* `z` - zepto, 10⁻²¹
* `a` - atto, 10⁻¹⁸
* `f` - femto, 10⁻¹⁵
* `p` - pico, 10⁻¹²
* `n` - nano, 10⁻⁹
* `µ` - micro, 10⁻⁶
* `m` - milli, 10⁻³
* `​` (none) - 10⁰
* `k` - kilo, 10³
* `M` - mega, 10⁶
* `G` - giga, 10⁹
* `T` - tera, 10¹²
* `P` - peta, 10¹⁵
* `E` - exa, 10¹⁸
* `Z` - zetta, 10²¹
* `Y` - yotta, 10²⁴

与 [*locale*.format](#locale_format) 使用 `s` 格式化类型不同, 这个方法始终返回一个固定的科学计数法前缀而不是为每个数值动态计算前缀. 此外, 给定说明符的精度表示小数点后的位数（用 `f` 表示定点数）而不是有效数字的数目. 例如:

```js
var f = d3.formatPrefix(",.0", 1e-6);
f(0.00042); // "420µ"
f(0.0042); // "4,200µ"
```

当将多个数字格式化为相同的单位以便比较时, 此方法非常有用. 参考 [precisionPrefix](#precisionPrefix) 选取合适的精度. 以及示例 [bl.ocks.org/9764126](http://bl.ocks.org/mbostock/9764126).

<a name="formatSpecifier" href="#formatSpecifier">#</a> d3.<b>formatSpecifier</b>(<i>specifier</i>) [<源码>](https://github.com/d3/d3-format/blob/master/src/formatSpecifier.js "Source")

解析指定的说明符 *specifier* 返回与 [format specification mini-language](#locale_format) 对应的字段描述对象和一个构造说明符的 `toString` 方法. 例如, `formatSpecifier("s")` 返回:

```js
{
  "fill": " ",
  "align": ">",
  "sign": "-",
  "symbol": "",
  "zero": false,
  "width": undefined,
  "comma": false,
  "precision": undefined,
  "trim": false,
  "type": "s"
}
```

此方法对于理解如何解析格式说明符以及派生新的说明符非常有用. 例如你可以基于你想要使用 [precisionFixed](#precisionFixed) 格式化的数字计算一个适当的精度然后创建一个新的格式:

```js
var s = d3.formatSpecifier("f");
s.precision = d3.precisionFixed(0.01);
var f = d3.format(s);
f(42); // "42.00";
```

<a name="precisionFixed" href="#precisionFixed">#</a> d3.<b>precisionFixed</b>(<i>step</i>) [<源码>](https://github.com/d3/d3-format/blob/master/src/precisionFixed.js "Source")

根据指定的 *step* 为定点记数法返回建议的十进制精度. *step* 表示将被格式化的值之间的最小绝对差. (假设要格式化的值也是 *step* 的倍数) 例如, 给定数值 `1`, `1.5` 和 `2`, *step* 应该被指定为 `0.5`, 则建议的精度为 `1`:

```js
var p = d3.precisionFixed(0.5),
    f = d3.format("." + p + "f");
f(1);   // "1.0"
f(1.5); // "1.5"
f(2);   // "2.0"
```

而对于数字 `1`, `2` 和 `3`, 步长应为 `1`, 建议精度为 `0`:

```js
var p = d3.precisionFixed(1),
    f = d3.format("." + p + "f");
f(1); // "1"
f(2); // "2"
f(3); // "3"
```

注意, 对于 `%` 格式类型, 应减去 `2`:

```js
var p = Math.max(0, d3.precisionFixed(0.05) - 2),
    f = d3.format("." + p + "%");
f(0.45); // "45%"
f(0.50); // "50%"
f(0.55); // "55%"
```

<a name="precisionPrefix" href="#precisionPrefix">#</a> d3.<b>precisionPrefix</b>(<i>step</i>, <i>value</i>) [<源码>](https://github.com/d3/d3-format/blob/master/src/precisionPrefix.js "Source")

根据指定的 *step* 和 *value* 返回与 [*locale*.formatPrefix](#locale_formatPrefix) 一起使用的建议的十进制精度. *step* 表示将被格式化的值之间的最小绝对差, *value* 决定使用哪个 `SI` 前缀. (假设要格式化的值也是 *step* 的倍数) 例如, 给定数值 `1.1e6`, `1.2e6` 和 `1.3e6`, *value* 为 `1.3e6`, 建议的精度为 `1`:

```js
var p = d3.precisionPrefix(1e5, 1.3e6),
    f = d3.formatPrefix("." + p, 1.3e6);
f(1.1e6); // "1.1M"
f(1.2e6); // "1.2M"
f(1.3e6); // "1.3M"
```

<a name="precisionRound" href="#precisionRound">#</a> d3.<b>precisionRound</b>(<i>step</i>, <i>max</i>) [<源码>](https://github.com/d3/d3-format/blob/master/src/precisionRound.js "Source")

根据指定的 *step* 和 *max* 返回四舍五入为有效数字的格式建议的十进制精度. *step* 表示要被格式化的值之间的最小绝对差, *max* 表示将被格式化的值的最大绝对值. (假设要格式化的值也是 *step* 的倍数) 例如, 给定数值 `0.99`, `1.0` 和 `1.01`, *step* 应该为 `0.01`, *max* 应该为 `1.01`, 建议的精度为 `3`：

```js
var p = d3.precisionRound(0.01, 1.01),
    f = d3.format("." + p + "r");
f(0.99); // "0.990"
f(1.0);  // "1.00"
f(1.01); // "1.01"
```

而对于数字 `0.9`、`1.0` 和 `1.1`, *step* 应该是 `0.1`, *max* 应该是 `1.1`, 建议的精度是 `2`:

```js
var p = d3.precisionRound(0.1, 1.1),
    f = d3.format("." + p + "r");
f(0.9); // "0.90"
f(1.0); // "1.0"
f(1.1); // "1.1"
```

注意:对于 `e` 格式类型, 要减去 `1`:

```js
var p = Math.max(0, d3.precisionRound(0.01, 1.01) - 1),
    f = d3.format("." + p + "e");
f(0.01); // "1.00e-2"
f(1.01); // "1.01e+0"
```

### Locales

<a name="formatLocale" href="#formatLocale">#</a> d3.<b>formatLocale</b>(<i>definition</i>) [<源码>](https://github.com/d3/d3-format/blob/master/src/locale.js "Source")

根据指定的 *definition* 返一个包含 [*locale*.format](#locale_format) 和 [*locale*.formatPrefix](#locale_formatPrefix) 方法的 *locale* 对象. *definition* 必须包含以下属:

* `decimal` - 小数点 (e.g., `"."`).
* `thousands` - 分组标点 (e.g., `","`).
* `grouping` - 分组大小 (e.g., `[3]`), 循环需要.
* `currency` - 货币前缀符号 (e.g., `["$", ""]`).
* `numerals` - 可选; 由十个字符串组成的数组, 用来替换数字 `0-9`.
* `percent` - 可选; 百分号后缀 (默认 `"%"`).

注意 *thousands* 属性是一个不准确的用词, 因为分组定义允许比千大的组.

<a name="formatDefaultLocale" href="#formatDefaultLocale">#</a> d3.<b>formatDefaultLocale</b>(<i>definition</i>) [<源码>](https://github.com/d3/d3-format/blob/master/src/defaultLocale.js "Source")

等价于 [d3.formatLocale](#formatLocale), 只不过 [d3.format](#format) 和 [d3.formatPrefix](#formatPrefix) 被重新定义为新的本地化的 [*locale*.format](#locale_format) 和 [*locale*.formatPrefix](#locale_formatPrefix). 如果没有设置本地化则默认为 [U.S. English](https://github.com/d3/d3-format/blob/master/locale/en-US.json).
