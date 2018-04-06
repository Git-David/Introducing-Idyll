# Introducing Idyll

Idyll 是一种类似 JSON 的轻量级数据交换格式，可用于数据交换、数据存储、配置、描述其它数据等。作为一种基于文本的半结构化数据格式，Idyll 完全是人类可读和语言无关的。它的设计目标是简单、易用、明确和优雅。

## 基本规则

Idyll 遵循以下基本规则：


- Idyll 文本必须以 UTF-8 编码
- Idyll 文本由一系列标记（token）构成，包括 Object、Array、String、Number、Boolean、Null 以及 6 个结构化字符
- 一个 Idyll 文本是一个 Object
- 结构化字符：`{`，`}`，`[`，`]`，`=`，`,`
- 标记前后允许出现任意个注释（comment）和空白符（whitespace）
- 空白符：水平制表符（`%x09`），空格（`%x20`），换行符（line-break）
- 换行符：`CR`，`LF`，`CRLF`


##Object

Object 是由一对花括号（`{}`）包围的 Key-Value Pair 列表，其中 Key 必须是一个 String，而 Value 可以是 Object、Array、String、Number、Boolean 和 Null 中的任意一种。

Key 与 Value 之间以等号（`=`）分隔，而 Key-Value Pair 之间以逗号（`,`）分隔。在最后一个 Key-Value Pair 之后允许出现一个无意义的逗号。

例子：

```
{
    firstName = Thomas,
    lastName = Anderson,
    born = "March 11, 1962",
}
```

在一般情况下，Object 中的 Key 具有唯一性。但是，多个 Key-Value Pair 具有相同的 Key 是合法的。在这种情况下，实现应该保留所有重复的 Key-Value Pair。Key 是大小写敏感的，至少要包含一个字符。

## Array

Array 是由一对方括号（`[]`）包围的有序 Value 列表，Value 可以是 Object、Array、String、Number、Boolean 和 Null 中的任意一种。

Value 之间以逗号（`,`）分隔。在最后一个 Value 之后允许出现一个无意义的逗号。

例子：

```
{
    vector = [1, 2, 3, 4],
    arguments = [
        0.1,
        2.0,
        "ordered",
        false,
        null,
    ],
}
```

列表中的 Value 并不要求有相同的类型。

## String

String 是一个由零到多个 Unicode 字符构成的字符序列。String 支持 4 种标记法（notation），分别是 quoted、unquoted、raw 和 multiline 标记法。

#### Quoted 标记法

Quoted 标记法与 C 语言家族中的惯用法类似：以一对双引号（`"`）包围一个字符序列（character sequence）来指定一个 String。任何字符都允许出现在字符序列之内，除了一些必须被转义（escaped）的字符：双引号、反斜杠（`\`）和换行符。

例子：

```
{
    Morpheus = "What is \"real\"? How do you define \"real\"?"
}
```

任何字符都可以被转义。可用的转义序列（escape sequence）如下：

| 转义序列 | ASCII 值（16 进制） | 描述 |
| ---- | ---- | ---- |
| `\"` | 22 | 双引号（Double quotation mark） |
| `\\` | 5C | 反斜杠（Backslash） |
| `\0` | 00 | 空字符（Null） |
| `\b` | 08 | 退格（Backspace） |
| `\f` | 0C | 换页（Form feed） |
| `\n` | 0A | 换行（Newline） |
| `\r` | 0D | 回车（Carriage Return） |
| `\t` | 09 | 水平制表符（Horizontal Tab） |
| `\uhhhh` |  | Unicode 码位（Code point），`h` 代表单个 16 进制数位 |
| `\Uhhhhhhhh` |  | Unicode 码位（Code point），`h` 代表单个 16 进制数位 |

Unicode 转义序列有两种形式：以 `\u` 开始，跟着 4 个十六进制数位（例如 `\u5b57`）；或以 `\U` 开始，跟着 8 个十六进制数位（例如 `\U0001D711`）。十六进制数位是大小写不敏感的。

Quoted 标记法支持串接（string literal concatenation），多个连续的 String 会连接为一个 String，例如：

```
{
    Spoon boy = "Do not try and bend the spoon - that's "
                "impossible. Instead, only try to realize "
                "the truth."
}
```

连续的 String 之间允许出现注释和空白符。

#### Unquoted 标记法

Unquoted 标记法没有显式的定界符，它的语法规则类似于多数语言中的标识符（identifier），有着严格的限制条件。

Unquoted 标记法允许使用的字符包括 26 个英文字母、数字、下划线（`_`）、连字符（`-`）、点（`.`）和空格，并且只能以字母或下划线开始。中间的空格不能连续出现，尾随的空格将被忽略。

一些例子：

```
{
    timeout = 90,
    ip-address = "127.0.0.1",
    config.cipher = aes256-ctr,
    _length_ = 4096,
    access = allow from all,
}
```

Unquoted 标记法不允许将 `true`、`false`、`null`、`inf` 和 `nan` 作为 Key 使用，因为这可能会引起混淆。

#### Raw 标记法

Raw 标记法支持自定义的定界符（delimiter），于是消除了对转义的需求。

Raw 标记法以 `'delimiter(` 开始，并以 `)delimiter'` 结束，它们之间是一个字符序列。除了换行符之外的任何字符都允许出现在序列之中。

`delimiter` 由 0 至 16 个重复的英文字母或数字构成。例如 `a`、`ddd`、`555` 都是合法的 `delimiter`，但 `abc`、`27` 或 `@@` 则是不合法的。

只要保证 `)delimiter'` 不是字符序列的子序列（subsequence），则该 String 就是良构的（well-formed）。在一般情况下，如果字符序列中不包含 `)'`，那么 `delimiter` 可以为空。

一些例子：

```
{
    image-tag = '(<\s*img[^>]+src\s*=\s*(["'])(.*?)\1[^>]*>)',
    path = '(\\?\UNC\server\share\resource)',
    bool-var = 'ddd(\w+\s*=\s*'(true|false)')ddd'
}
```

Raw 标记法支持串接（string literal concatenation），多个连续的 String 会连接为一个 String，例如：

```
{
    date = '((0?[1-9]|[12][0-9]|3[01])([ \/\-]))'
           '((0?[1-9]|1[012])\2([0-9][0-9][0-9][0-9]))'
}
```

连续的 String 可以使用不同的 `delimiter`，它们之间允许出现注释和空白符。

#### Multiline 标记法

Multiline 标记法由一到多个行组成。每行都以竖线（`|`）开始，并以换行符结束。竖线与换行符包围了一个字符序列，任何字符都可以出现在字符序列里。竖线之前允许出现零到多个无意义的水平制表符和空格。

例子：

```
{
    code =
        |for i from 1 to number_of_nodes do
        |    status <- tick(node(i))
        |    if status = running
        |        return running
        |    else if status = failure
        |        return failure
        |end
        |return success
}
```

两个连续的字符序列之间存在一个换行符，也就是在 $n$ 个字符序列之间存在 $n - 1$ 个换行符。

在上面的例子中，String 结尾处并没有换行符。如果要在 String 结尾处保留一个换行符，根据上述规则，只需要留一个空行，例如：

```
{
    text =
        |Then you will see that it is not the spoon
        |that bends, it is only yourself.
        |
}
```

这些换行符并非来自于 Idyll 文本，而是解析器。文本在不同的系统和程序之间传输时，其中的换行符可能发生改变。解析器应该允许用户配置使用何种风格的换行符。

## Number

Number 与多数计算机语言有着类似的表示法，使用十进制数字来表示实数。它支持整数（integer）、定点数（fixed floating-point）和指数（exponential）三种标记法。

Number 具有这样的一般形式：

```
[sign] significand [exponent]
```

`significand` 具有如下形式：

```
( "0" / digit1-9 *DIGIT ) [ "." 1*DIGIT ]
```

`exponent` 具有如下形式：

```
e [sign] ( "0" / digit1-9 *DIGIT )
```

`sign` 为符号，可以为正（`+`）或者负（`-`）。

`significand` 包含整数和小数两个部分，其中整数部分是必需的，小数部分是可选的。当小数部分不存在时，小数点也不应该存在。例如 `0`、`123` 和 `1.23` 都是合法的，但 `.1` 或者 `2.` 是不合法的。

`e` 可以为大写（`E`）或者小写（`e`）。

Number 所表示的值为 `significand` 乘以 `10` 的 `exponent` 次幂，所以 `1.23e4` 的数学意义是 $1.23 \times 10 ^ 4$。

除了一般的值以外，Number 还有两个特殊值：`inf` 和 `nan`。它们分别用于表示无穷和 NaN（not a number）。`inf` 和 `nan` 同样可以带符号。

一些例子：

```
{
    ii = -1,
    e = 2.7182818,
    length = 40075,
    speed = 3e8,
    mass = 1.98855e30,
    distance = inf,
    result = -nan,
}
```

## Boolean

Boolean 只有 `true` 和 `false` 两个值，它一般用于表示逻辑真值。

例子：

```
{
    async = true,
    dev = false
}
```

## Null

Null 唯一的值就是 `null`。它常常用于表示空（empty）、没有（none）、没有值（no value）或者无效（invalid）等。

例子：

```
{
    list = null
}
```

## 注释

Idyll 支持 single line 和 block 两种风格的注释。Single line 注释以单个井号（`#`）开始，忽略接下来的所有字符，直到行尾结束。Block 注释以两个或多个井号开始，遇到相同数量的井号结束，中间的所有字符将被忽略。

例子：

```
##
This is a block comment,
followed by a second line.
##
{
    # This is a single line comment.
    a = 0, ## block comment ## b = 1, c = 2, d = 3,

    ####
    obj = {
        n = 0.0,
        m = 1.0,
    },
    ####
}
```

## 解析与生成

解析器必须遵守 Idyll 的语法规则。

解析器可以限制文本的最大长度。解析器可以限制嵌套 Object 和 Array 的最大深度。解析器可以限制 Number 的范围和精度。解析器可以限制 String 的最大长度。

## 生成器

生成器生成的文本必须遵守 Idyll 的语法规则。

