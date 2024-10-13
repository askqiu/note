```
'\x61'//a
2 "\x61"//a
3 `\x61`//a
4`\x61\x62` //ab
5 function a(){}
6 \x61()//fails
```

```
1 '\u0061'//a
2 "\u0061"//a
3 `\u0061`//a
4
5 function a(){}
6 \u0061()//correctly calls the function
```
```

1 '\u{61}'//a
2 "\u{000000000061}"//a
3 `\u{0061}`//a
4
5 function a(){}
6 \u{61}()//correctly calls the function
7
8 \u{3134a}=123//unicode character "3134a" is allowed as a variable
```

```
八进制
1 '\141'//a
2 "\8"//number outside the octal range so 8 is returned
3 `\9`//number outside the octal range so 9 is returned
```
``

# eval和转义
```
eval('\u0061\u006C\u0065\u0072\u0074\u0028\u0029');  //eval('alert()')
```

```
1 eval('\\u0061=123')
2 //\u0061 = 123
3 //a = 123
为了防止提前转义，比如说\u被转义，于是我们使用双反斜杠
```

混合转义
```
1 eval('\\u0061=123')//unicode escape using "a" assignment
2 eval('\\u\x30061=123')//hex encode the first zero
3 eval('\\u\x300\661=123')//octal encode the 6
```

# 字符串
字符串有三种形式，单引号，双引号和模板字符串
我们可以在所有类型的字符串使用各种编码，以及单字符转义序列

1 '\b'//backspace
2 '\f'//form feed
3 '\n'//new line
4 '\r'//carriage return
5 '\t'//tab
6 '\v'//vertical tab
7 '\0'//null
8 '\''//single quote
9 '\"'//double quote
10 '\\'//backslash

也可以转义不属于以上序列的任何字符，
```
"\H\E\L\L\O"//HELLO    绕过waf
```

```
1 'I continue \        //在行尾使用反斜杠继续到下一行
2 onto the next line'   
```