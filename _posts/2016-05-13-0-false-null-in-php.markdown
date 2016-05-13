---
layout:     post
title:      "PHP中比较0、false、null和""的"坑""
subtitle:   ""
date:       2016-05-13
author:     "haitian-coder"
header-img: "img/bg.jpg"
tags:
    - PHP
---


###测试代码:

```
//test.php
echo '0 == false: ';
var_dump(0 == false);
echo '0 === false: ';
var_dump(0 === false);
echo "\n";
echo '0 == null: ';
var_dump(0 == null);
echo '0 === null: ';
var_dump(0 === null);
echo "\n";
echo 'false == null: ';
var_dump(false == null);
echo 'false === null: ';
var_dump(false === null);
echo "\n";
echo '"0" == false: ';
var_dump("0" == false);
echo '"0" === false: ';
var_dump("0" === false);
echo "\n";
echo '"0" == null: ';
var_dump("0" == null);
echo '"0" === null: ';
var_dump("0" === null);
echo "\n";
echo '"" == false: ';
var_dump("" == false);
echo '"" === false: ';
var_dump("" === false);
echo "\n";
echo '"" == null: ';
var_dump("" == null);
echo '"" == null: ';
var_dump("" === null);
```

###测试结果:

```
→ php test.php
0 == false: bool(true)
0 === false: bool(false)

0 == null: bool(true)
0 === null: bool(false)

false == null: bool(true)
false === null: bool(false)

"0" == false: bool(true)
"0" === false: bool(false)

"0" == null: bool(false)
"0" === null: bool(false)

"" == false: bool(true)
"" === false: bool(false)

"" == null: bool(true)
"" == null: bool(false)
```

###[官方解释][1]:

![PHP比较运算符][2]

###所以

除非你真的知道你在用`==`比较什么，一般情况用`===`更安全。

比如像[`array_search`][3]，没找到返回`false`，找到了返回`key`，而`key`是可能为`0`的。


  [1]: http://php.net/manual/zh/language.operators.comparison.php
  [2]: /img/bVvdtw
  [3]: http://php.net/manual/zh/function.array-search.php
