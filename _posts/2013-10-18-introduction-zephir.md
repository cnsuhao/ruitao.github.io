---
layout: post
title: zephir，不用C也可以编写PHP扩展
---

# {{ page.title }}

在Python的工具链里面，我们可以不用直接写C，用cython这种来编写扩展
现在PHP也可以了！那帮编写了 [phalcon](http://phalconphp.com/en/) 为了 [让更多人参参与到phalcon这个框架的开发](http://blog.phalconphp.com/post/57161129440/phalcon-2-0-the-future) ，搞了一门新的语言出来

![zephir](http://media.tumblr.com/983f6927ede265a8647085b2c0dc41ad/tumblr_inline_mqx3a8ywwp1qz4rgp.png)

可以用高级语言写PHP的C扩展，并且phalcon 2.0已经改用zephir重写
根据 phalcon 团队发布的博文： [Phalcon 2.0 progress + benchmarks](http://blog.phalconphp.com/post/61527480269/phalcon-2-0-progress-benchmarks)
可以做到跟用C写几乎一样的性能（甚至更好一点）

## helloworld

简单记录一下如何用zephir写一个Helloworld扩展

按照zephir的 [README](https://github.com/phalcon/zephir) 在本地编译好

随便创建一个目录，比如 `helloworld`，zephir里面所有的文件都必须有且仅有一个类，
每一个类都必须有一个 namespace 而且要跟文件夹、文件名一样，比如这里，我们要创建
一个 `Test\Hello` 这样命名空间的类

```bash
mkdir helloworld
cd helloworld
mkdir test
vim test/hello.zep
```

粘贴如下的代码

```
namespace Test;

/**
 * This is a sample class
 */
class Hello
{
    /**
     * This is a sample method
     */
    public function say()
    {
        echo "Hello World!";
    }
}
```

新建一个 `config.json` 配置

```json
{
    "namespace": "test",
}
```

编译：

```bash
cd helloworld
../bin/zephir compile
```

安装

```bash
../bin/zephir install
```

测试

```php
<?php
dl('test.so');
$hello = new Test\Hello();
$hello->say();
?>
```

语法是不是很简单呢，按照phalcon官方的说法：

>The syntax of Zephir is inspired by C, PHP, Rust and Javascript

对于我等C Family码农来说，不是什么问题，可以看看 [官方的语法](http://zephir-lang.com/)

## 调用PHP函数

在zephir里面，你也可以直接调用php函数，比如下面的范例，调用 `bash64_encode`

```
namespace MyLibrary;

class Encoder
{

    public function encode(var text)
    {
        if strlen(text) != 0 {
            return base64_encode(text);
        }
        return false;
    }

}
```

更多的可以看 [](http://zephir-lang.com/functions.html)

## 调用c类库

zephir采用了跟go里面的cgo类似的方式调用c类库，具体可以看看zephir代码树里面的范例[cblock.zep](https://github.com/phalcon/zephir/blob/master/test/cblock.zep)
来自于 [这个PR](https://github.com/phalcon/zephir/pull/21)
