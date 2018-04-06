# Introducing Idyll

Idyll is a JSON-like, lightweight data interchange format. It could be used for data interchange, data storage, configuration, data description, etc. As a text-based and semi-structured format, Idyll is completely human readable and language independent. Its design goals were to be simple, easy to use, explicit and elegant.

## Basic rules

Idyll follows these basic rules:


- A Idyll text MUST encoded in UTF-8
- A Idyll text is a sequence of tokens, the set of tokens includes Objects, Arrays, Strings, Numbers, Booleans, Nulls and six structural characters
- A Idyll text is an Object
- Six structural characters: `{`, `}`, `[`, `]`, `=`, `,`
- Insignificant comments and whitespaces are allowed before and after any token
- Whitespace: tab(`%x09`), space(`%x20`), line-break
- Line-break: `CR`, `LF`, `CRLF`


## Object

An Object structure is represented as a pair of curly brackets(`{}`) surrounding zero or more Key-Value pairs. A Key is a String, and a Value could be one of Object, Array, String, Number, Boolean or Null.

Key and Value are separated by equals sign(`=`), and Key-Value pairs are separated by comma(`,`). A single comma is allowed after the last Key-Value pair in Object.

Example:

```
{
    firstName = Thomas,
    lastName = Anderson,
    born = "March 11, 1962",
}
```

In general, Keys within an Object are unique. But, more than one Key-Value pair that has the same Key is allowed. In this situation the implementation MUST report all the Key-Value pairs to the user. Keys are case sensitive, and MUST have at least one character.

## Array

An Array structure is represented as a pair of square brackets(`[]`) surrounding an ordered list of zero or more Values. A Value could be one of Object, Array, String, Number, Boolean or Null.

Values are separated by comma(`,`). A single comma is allowed after the last Value in Array.

Example:

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

Values in an Array may have different types.

## String

A String is a sequence of zero or more Unicode characters. Four String notations are provided:  quoted, unquoted, raw and multiline.

#### Quoted notation

Quoted notation is similar to conventions used in the C family languages: A String is represented as a pair of double quotation marks(`""`) surrounding a character sequence. All Unicode characters may be placed in the sequence, except for the characters that must be escaped: double quotation mark(`"`), backslash(`\`) and line-breaks.

Example:

```
{
    Morpheus = "What is \"real\"? How do you define \"real\"?"
}
```

Any character may be escaped. Available escape sequences are as follow:

| Escape sequence | ASCII (hex) | Character represented |
| ---- | ---- | ---- |
| `\"` | 22 | Double quotation mark |
| `\\` | 5C | Backslash |
| `\0` | 00 | Null |
| `\b` | 08 | Backspace |
| `\f` | 0C | Form feed |
| `\n` | 0A | Newline |
| `\r` | 0D | Carriage Return |
| `\t` | 09 | Horizontal Tab |
| `\uhhhh` |  | Unicode Code point where h is a hex-digit |
| `\Uhhhhhhhh` |  | Unicode Code point where h is a hex-digit |

Unicode escape sequence has two forms: Begins with `\u ` and followed by 4 hex-digits (such as `\u5b57`), or begins with `\U` and followed by 8 hex-digits (such as `\U0001D711`). The hex-digits are case insensitive.

Quoted notation supports string literal concatenation, where adjacent Strings are implicitly joined into a single String, for example:

```
{
    Spoon boy = "Do not try and bend the spoon - that's "
                "impossible. Instead, only try to realize "
                "the truth."
}
```

Comments and whitespaces are allowed between adjacent Strings.

#### Unquoted notation

Unquoted notation has no explicit delimiter, it is similar to identifier's grammar rule in most programming languages, it has strict limitations.

Allowed characters: letters(`A-Z, a-z`), digits(`0-9`), underscore(`_`), hyphen(`-`), dot(`.`) and space. The first character must be a letter or underscore. Consecutive spaces are not allowed, tailing spaces will be ignored.

Examples:

```
{
    timeout = 90,
    ip-address = "127.0.0.1",
    config.cipher = aes256-ctr,
    _length_ = 4096,
    access = allow from all,
}
```

Unquoted notation disallowed using `true`, `false`, `null`, `inf`, or `nan` as Key, which may lead to confusion.

#### Raw notation

Raw notation avoided the need for escaping by providing custom delimiter.

Raw notation begins with `'delimiter(` and ends with `)delimiter'`, between them is a character sequence. All Unicode characters may be placed in the sequence, except line-breaks.

`delimiter` is made of zero to sixteen repeated letters or digits. For example, `a`, `ddd` or `555` is valid `delimiter`, but `abc`, `27` or `@@` is not.

As long as `)delimiter'` is not a subsequence of the character sequence, the String is well-formed. In the general cases, if the character sequence does not contain `)'`, then `delimiter` could be empty.

Examples:

```
{
    image-tag = '(<\s*img[^>]+src\s*=\s*(["'])(.*?)\1[^>]*>)',
    path = '(\\?\UNC\server\share\resource)',
    bool-var = 'ddd(\w+\s*=\s*'(true|false)')ddd'
}
```

Raw notation supports string literal concatenation, where adjacent Strings are implicitly joined into a single String, for example:

```
{
    date = '((0?[1-9]|[12][0-9]|3[01])([ \/\-]))'
           '((0?[1-9]|1[012])\2([0-9][0-9][0-9][0-9]))'
}
```

Adjacent Strings could have different `delimiter`, comments and whitespaces are allowed between them.

#### Multiline notation

Multiline notation consists of one or more lines. Every line begins with vertical bar(`|`), and ends with line-break. Between them is a character sequence. All Unicode characters may be placed in the sequence. Insignificant tabs and spaces are allowed before the vertical bar.

Example:

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

A single line-break exists between two adjacent character sequences, namely, between $n$ character sequences there are $n - 1$ line-breaks.

In the example above, there is no line-break at the end of the String. If a line-break is needed at the end of a String, according to the rule, just retain an empty line, for example:

```
{
    text =
        |Then you will see that it is not the spoon
        |that bends, it is only yourself.
        |
}
```

These line-breaks are not come from the Idyll text but the parser. Line-breaks in a text may change during transmission between different systems and software programs. A parser SHOULD allow users to set what kind of line-break to use.

## Number

The representation of Numbers is similar to that used in most programming languages. A Number is represented in base 10 using decimal digits. It supports integer, fixed floating-point and exponential notation.

A Number has the form:

```
[sign] significand [exponent]
```

The `significand` has the form:

```
( "0" / digit1-9 *DIGIT ) [ "." 1*DIGIT ]
```

The `exponent` has the form:

```
e [sign] ( "0" / digit1-9 *DIGIT )
```

`sign` is either `+` or `-`.

`significand` contains integer part and fractional part. Integer part is required, fractional part is optional. In cases where fractional part does not exist, the decimal point should not exist as well. For example, `0`, `123` or `1.23` is valid, but `.1` or `2.` is not valid.

`e` is either upper case `E` or lower case `e`.

The value of a Number is `significand` times 10 raised to the power of `exponent`, so `1.23e4`'s mathematical meaning is $1.23 \times 10 ^ 4$.

In addition to normal values, Number also has two special values: `inf` and `nan`, which representing infinite and NaN(not a number), they could have sign as well.

Examples:

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

Booleans have only two values: `true` and `false`, which are commonly used to represent the truth values of logic.

Examples:

```
{
    async = true,
    dev = false
}
```

## Null

Nulls have only one value: `null`, it is commonly used to represent empty, none, no value, invalid, etc.

Example:

```
{
    list = null
}
```

## Comment

Idyll supports two styles of comment: single line and block. A single line comment begins with a single number sign(`#`), following characters will be ignored until line-break. A block comment begins with two or more number signs and ends with the same number of number signs, all characters in the middle will be ignored.

Examples:

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

## Parser

A parser MUST strictly conform to the Idyll grammar.

A parser may set limits on the size of input text that it accepts. A parser may set limits on the maximum depth of nesting Object and Array. A parser may set limits on the precision and range of Numbers. A parser may set limits on the length of Strings.

## Generator

The resulting text MUST strictly conform to the Idyll grammar.

