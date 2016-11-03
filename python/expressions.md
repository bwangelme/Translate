6\. 表达式
==========

### 6.2.9 Yield 表达式

```
yield_atom ::= "(" yield_expression ")"
yield_expression ::= "yield" [ expression_list | "from" expression ]
```

yield 表达式仅仅用在定义一个生成器函数上，因此仅仅能够用在一个函数定义体中。在一个函数体中使用 yield 表达式将会使函数变成一个生成器。

当一个生成器函数被调用的时候，它返回一个称作生成器的迭代器。那个生成器控制着生成器函数的执行。当一个生成器的方法被调用的时候生成器开始执行。在那时，生成器运行到第一个 yield 表达式。在这里，它再次休眠，返回一个表达式列表的值给生成器的调用者。通过暂停，我们意味着保留了所有的本地状态，包括当前绑定的本地变量，指令指针，~~内部评估堆栈~~，~~和任何异常处理的状态~~。当执行被一个生成器的方法回复的时候，函数能够精确地处理它就好像 yield 表达式仅仅是一个外部调用一样。回复后 yield 表达式的值取决于回复执行的方法。如果 `__next__()` 被使用了(典型地通过内建的`for`或者`next()`来访问)，那么结果是`None`。否则的话，如果`send()`被使用了，那么结果将会是被传递到那个方法的值。

所有的这些使得生成器非常像协程。他们 yield 多次，他们有多个入口点，他们的执行被暂停。唯一的不同是生成器函数不能控制在它 yield 之后什么时候再来开始执行。控制权被转入到了生成器的调用者。

yield 表达式允许在 try 结构的任意部分去使用。如果生成器在它终止之前没有被恢复(索引计数为0或者被垃圾回收)，生成器的`close()`函数将会被调用，允许任何待定的 finally 子句去执行。

当`yield from <expr>`被使用的时候，它将提供的表达式视为一个子迭代器。所有由子迭代器产生的值将会被直接传送到当前生成器方法的调用者那里。任何通过`send()`方法或者传入的值，或者通过`throw()`方法抛出的异常将会被立刻传递到底层的迭代器，如果它们有合适的方法的话。如果没有的话，`send()`方法将会抛出一个`AttributeError`或者`TypeError`，而`throw()`将仅仅会将传入的异常直接抛出。

当底层迭代器完成的时候，抛出的`StopIteration`实例的值属性将会变成 yield 表达式的值。它能够被精确地设置成抛出一个`StopIteration`，或者自动地抛出当子迭代器是一个生成器的时候(通过从子生成器的返回值)。

> Changed in version 3.3: 增加了`yield from <expr>`去代表子迭代器的控制流。

TODO
========