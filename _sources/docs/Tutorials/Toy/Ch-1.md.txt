# 第1章：Toy语言和AST（抽象语法树）

[TOC]

## 语言

本教程将使用一个我们称之为“Toy”的玩具语言来进行说明（命名很难……）。Toy是一种基于张量的语言，允许您定义函数、执行一些数学计算并输出结果。

为了保持简单，代码生成将仅限于秩<=2的张量，而Toy中唯一的数据类型是64位浮点数类型（在C语言中称为“double”）。因此，所有值都是隐式的双精度，值是不可变的（即每个操作都返回一个新分配的值），并且自动管理内存释放。但是，足够长的描述了，最好通过一个例子来更好地理解：

```toy
def main() {
  #在这段代码中使用字面值初始化一个变量a，并且没有显式地指定变量的形状。
  #编译器会从提供的字面值中推断出变量的形状为<2, 3>
  var a = [[1, 2, 3], [4, 5, 6]];

  #在这段代码中使用相同的字面值初始化了变量b和a。
  #由于变量b被显式地声明为形状为<2, 3>的张量，编译器会将字面值隐式地重塑为该形状。
  #需要注意的是，如果我们想要重塑张量，我们需要定义一个新变量，
  #并确保新变量的元素数量与原始张量相同。
  var b<2, 3> = [1, 2, 3, 4, 5, 6];

  #在这段代码中使用transpose()函数将变量a和b进行转置，它们进行逐元素乘法运算。
  #最后，使用print()函数打印结果。transpose()和print()是该语言唯一的内置函数
  # 其他函数都需要用户定义
  print(transpose(a) * transpose(b));
}
```

类型检查是通过类型推断静态执行的；语言只在需要时才需要指定张量形状的类型声明。函数是通用的：它们的参数是未排序的（换句话说，我们知道这些是张量，但我们不知道它们的维度）。它们在调用站点为每个新发现的签名进行专门化。让我们通过添加一个用户定义的函数来重新审视上一个例子：

```toy
# 这是一个操作未知形状参数的用户定义通用函数的示例.
def multiply_transpose(a, b) {
  return transpose(a) * transpose(b);
}

def main() {
  # Define a variable `a` with shape <2, 3>, initialized with the literal value.
  var a = [[1, 2, 3], [4, 5, 6]];
  var b<2, 3> = [1, 2, 3, 4, 5, 6];

  # This call will specialize `multiply_transpose` with <2, 3> for both
  # arguments and deduce a return type of <3, 2> in initialization of `c`.
  var c = multiply_transpose(a, b);

  # A second call to `multiply_transpose` with <2, 3> for both arguments will
  # reuse the previously specialized and inferred version and return <3, 2>.
  var d = multiply_transpose(b, a);

  # A new call with <3, 2> (instead of <2, 3>) for both dimensions will
  # trigger another specialization of `multiply_transpose`.
  var e = multiply_transpose(c, d);

  # Finally, calling into `multiply_transpose` with incompatible shape will
  # trigger a shape inference error.
  var f = multiply_transpose(transpose(a), c);
}
```

## The AST

上面代码的AST（抽象语法树）非常简单，以下是它的输出：:

```
Module:
  Function 
    Proto 'multiply_transpose' @test/Examples/Toy/Ch1/ast.toy:4:1'
    Params: [a, b]
    Block {
      Return
        BinOp: * @test/Examples/Toy/Ch1/ast.toy:5:25
          Call 'transpose' [ @test/Examples/Toy/Ch1/ast.toy:5:10
            var: a @test/Examples/Toy/Ch1/ast.toy:5:20
          ]
          Call 'transpose' [ @test/Examples/Toy/Ch1/ast.toy:5:25
            var: b @test/Examples/Toy/Ch1/ast.toy:5:35
          ]
    } // Block
  Function 
    Proto 'main' @test/Examples/Toy/Ch1/ast.toy:8:1'
    Params: []
    Block {
      VarDecl a<> @test/Examples/Toy/Ch1/ast.toy:11:3
        Literal: <2, 3>[ <3>[ 1.000000e+00, 2.000000e+00, 3.000000e+00], <3>[ 4.000000e+00, 5.000000e+00, 6.000000e+00]] @test/Examples/Toy/Ch1/ast.toy:11:11
      VarDecl b<2, 3> @test/Examples/Toy/Ch1/ast.toy:15:3
        Literal: <6>[ 1.000000e+00, 2.000000e+00, 3.000000e+00, 4.000000e+00, 5.000000e+00, 6.000000e+00] @test/Examples/Toy/Ch1/ast.toy:15:17
      VarDecl c<> @test/Examples/Toy/Ch1/ast.toy:19:3
        Call 'multiply_transpose' [ @test/Examples/Toy/Ch1/ast.toy:19:11
          var: a @test/Examples/Toy/Ch1/ast.toy:19:30
          var: b @test/Examples/Toy/Ch1/ast.toy:19:33
        ]
      VarDecl d<> @test/Examples/Toy/Ch1/ast.toy:22:3
        Call 'multiply_transpose' [ @test/Examples/Toy/Ch1/ast.toy:22:11
          var: b @test/Examples/Toy/Ch1/ast.toy:22:30
          var: a @test/Examples/Toy/Ch1/ast.toy:22:33
        ]
      VarDecl e<> @test/Examples/Toy/Ch1/ast.toy:25:3
        Call 'multiply_transpose' [ @test/Examples/Toy/Ch1/ast.toy:25:11
          var: c @test/Examples/Toy/Ch1/ast.toy:25:30
          var: d @test/Examples/Toy/Ch1/ast.toy:25:33
        ]
      VarDecl f<> @test/Examples/Toy/Ch1/ast.toy:28:3
        Call 'multiply_transpose' [ @test/Examples/Toy/Ch1/ast.toy:28:11
          Call 'transpose' [ @test/Examples/Toy/Ch1/ast.toy:28:30
            var: a @test/Examples/Toy/Ch1/ast.toy:28:40
          ]
          var: c @test/Examples/Toy/Ch1/ast.toy:28:44
        ]
    } // Block
```

你可以在examples/toy/Ch1/目录下复现这个结果，并尝试运行以下命令进行测试。
```
path/to/BUILD/bin/toyc-ch1 test/Examples/Toy/Ch1/ast.toy -emit=ast
```
词法分析器的代码非常简单，全部都在单个头文件:
`examples/toy/Ch1/include/toy/Lexer.h`. 解析器的代码可以在
`examples/toy/Ch1/include/toy/Parser.h`中找到; 它是一个递归下降解析器。如果你不熟悉这样的词法分析器和解析器, 它们与LLVM Kaleidoscope非常相似，这些内容在
[Kaleidoscope Tutorial教程](https://llvm.org/docs/tutorial/MyFirstLanguageFrontend/LangImpl02.html)的前两章中有详细介绍。

[下一章](Ch-2.md) 将演示如何将这个AST转换为MLIR。
