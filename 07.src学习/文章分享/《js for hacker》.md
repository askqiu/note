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