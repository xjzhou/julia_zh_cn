.. _man-control-flow:

********
 控制流  
********

Julia 提供一系列控制流：

-  :ref:`man-compound-expressions` ： ``begin`` 和 ``(;)`` 
-  :ref:`man-conditional-evaluation` ： ``if``-``elseif``-``else`` 和 ``?:`` （三重运算符）
-  :ref:`man-short-circuit-evaluation` ： ``&&`` 、 ``||`` 和链式比较
-  :ref:`man-loops` ： ``while`` 和 ``for`` 
-  :ref:`man-exception-handling` ： ``try``-``catch`` 、 ``error`` 和 ``throw`` 
-  :ref:`man-tasks` ： ``yieldto`` 

前五个控制流机制是高级编程语言的标准。任务则不是：它提供了非本地的控制流，便于在临时暂停的计算中进行切换。在 Julia 中，异常处理和协同多任务都是使用的这个机制。

.. _man-compound-expressions:

复合表达式
----------

用一个表达式按照顺序对一系列子表达式求值，并返回最后一个子表达式的值，有两种方法： ``begin`` 块儿和 ``(;)`` 链。 ``begin`` 块儿的例子： ::

    julia> z = begin
             x = 1
             y = 2
             x + y
           end
    3

这个块儿很短也很简单，可以用 ``(;)`` 链语法将其放在一行上： ::

    julia> z = (x = 1; y = 2; x + y)
    3

这个语法在 :ref:`man-functions` 中的单行函数定义非常有用。 ``begin`` 块儿也可以写成单行， ``(;)`` 链也可以写成多行： ::

    julia> begin x = 1; y = 2; x + y end
    3

    julia> (x = 1;
            y = 2;
            x + y)
    3

.. _man-conditional-evaluation:

条件求值
--------

一个 ``if``-``elseif``-``else`` 条件表达式的例子： ::

    if x < y
      println("x is less than y")
    elseif x > y
      println("x is greater than y")
    else
      println("x is equal to y")
    end

这是它用在实际中的例子： ::

    julia> function test(x, y)
             if x < y
               println("x is less than y")
             elseif x > y
               println("x is greater than y")
             else
               println("x is equal to y")
             end
           end

    julia> test(1, 2)
    x is less than y

    julia> test(2, 1)
    x is greater than y

    julia> test(1, 1)
    x is equal to y

``elseif`` 及 ``else`` 块儿是可选的。

如果条件表达式的值是除 ``true`` 和 ``false`` 之外的值，会出错： ::

    julia> if 1
             println("true")
           end
    type error: lambda: in if, expected Bool, got Int64

“问号表达式”语法 ``?:`` 与 ``if``-``elseif``-``else`` 语法相关，但是适用于单行表达式： ::

    a ? b : c

``?`` 之前的 ``a`` 是条件表达式，如果为 ``true`` ，问号表达式对 ``:`` 之前的 ``b`` 表达式求值，如果为 ``false`` ，问号表达式对 ``:`` 的 ``c`` 表达式求值。

用问号表达式来重写，可以使前面的例子更加紧凑。先看一个二选一的例子： ::

    julia> x = 1; y = 2;

    julia> println(x < y ? "less than" : "not less than")
    less than

    julia> x = 1; y = 0;

    julia> println(x < y ? "less than" : "not less than")
    not less than

三选一的例子需要链式调用问号表达式： ::

    julia> test(x, y) = println(x < y ? "x is less than y"    :
                                x > y ? "x is greater than y" : "x is equal to y")

    julia> test(1, 2)
    x is less than y

    julia> test(2, 1)
    x is greater than y

    julia> test(1, 1)
    x is equal to y

链式问号表达式的结合规则是从右到左。

与 ``if``-``elseif``-``else`` 类似， ``:`` 前后的表达式，只有在对应条件表达式为 ``true`` 或 ``false`` 时才求值： ::

    v(x) = (println(x); x)

    julia> 1 < 2 ? v("yes") : v("no")
    yes
    "yes"

    julia> 1 > 2 ? v("yes") : v("no")
    no
    "no"

.. _man-short-circuit-evaluation:

短路求值
--------

 ``&&`` 和 ``||`` 布尔运算符被称为短路求值，它们连接一系列布尔表达式，仅计算最少的表达式来确定整个链的布尔值。这意味着：

-  在表达式 ``a && b`` 中，只有 ``a`` 为 ``true`` 时才计算子表达式 ``b`` 
-  在表达式 ``a || b`` 中，只有 ``a`` 为 ``false`` 时才计算子表达式 ``b`` 

``&&`` 和 ``||`` 都与右侧结合，但 ``&&`` 比 ``||`` 优先级高： ::

    t(x) = (println(x); true)
    f(x) = (println(x); false)

    julia> t(1) && t(2)
    1
    2
    true

    julia> t(1) && f(2)
    1
    2
    false

    julia> f(1) && t(2)
    1
    false

    julia> f(1) && f(2)
    1
    false

    julia> t(1) || t(2)
    1
    true

    julia> t(1) || f(2)
    1
    true

    julia> f(1) || t(2)
    1
    2
    true

    julia> f(1) || f(2)
    1
    2
    false

如果不想使用短路求值，可以用 :ref:`man-mathematical-operations` 中介绍的位布尔运算符 ``&`` 和 ``|`` ： ::

    julia> f(1) & t(2)
    1
    2
    false

    julia> t(1) | t(2)
    1
    2
    true

``&&`` 和 ``||`` 的操作数也必须是布尔值（ ``true`` 或 ``false`` ），否则会出现错误： ::

    julia> 1 && 2
    type error: lambda: in if, expected Bool, got Int64

.. _man-loops:

重复求值: 循环
--------------

有两种循环表达式： ``while`` 循环和 ``for`` 循环。下面是 ``while`` 的例子： ::

    julia> i = 1;

    julia> while i <= 5
             println(i)
             i += 1
           end
    1
    2
    3
    4
    5

上例也可以重写为 ``for`` 循环： ::

    julia> for i = 1:5
             println(i)
           end
    1
    2
    3
    4
    5

此处的 ``1:5`` 是一个 ``Range`` 对象，表示的是 1, 2, 3, 4, 5 序列。 ``for`` 循环遍历这些数，将其逐一赋给变量 ``i`` 。 ``while`` 循环和 ``for`` 循环的另一区别是变量的作用域。如果在其它作用域中没有引入变量 ``i`` ，那么它仅存在于 ``for`` 循环中。不难验证： ::

    julia> for j = 1:5
             println(j)
           end
    1
    2
    3
    4
    5

    julia> j
    j not defined

有关变量作用域，详见 :ref:`man-variables-and-scoping` 。

通常， ``for`` 循环可以遍历任意容器。这时，应使用另一个（但是完全等价的）关键词 ``in`` ，而不是 ``=`` ，它使得代码更易阅读： ::

    julia> for i in [1,4,0]
             println(i)
           end
    1
    4
    0

    julia> for s in ["foo","bar","baz"]
             println(s)
           end
    foo
    bar
    baz

手册中将介绍各种可迭代容器（详见 :ref:`man-arrays` ）。

有时要提前终止 ``while`` 或 ``for`` 循环。可以通过关键词 ``break`` 来实现： ::

    julia> i = 1;

    julia> while true
             println(i)
             if i >= 5
               break
             end
             i += 1
           end
    1
    2
    3
    4
    5

    julia> for i = 1:1000
             println(i)
             if i >= 5
               break
             end
           end
    1
    2
    3
    4
    5

有时需要中断本次循环，进行下一次循环，这时可以用关键字 ``continue`` ： ::

    julia> for i = 1:10
             if i % 3 != 0
               continue
             end
             println(i)
           end
    3
    6
    9

多层 ``for`` 循环可以被重写为一个外层循环，迭代类似于笛卡尔乘积的形式： ::

    julia> for i = 1:2, j = 3:4
             println((i, j))
           end
    (1,3)
    (1,4)
    (2,3)
    (2,4)

.. _man-exception-handling:

异常处理
--------

当遇到意外条件时，函数可能无法给调用者返回一个合理值。这时，要么终止程序，打印诊断错误信息；要么程序员编写异常处理。

``error`` 函数表明出现了中断正常控制流的意外条件。当对负数使用内建的 ``sqrt`` 函数时，将返回 ``DomainError()`` ： ::

    julia> sqrt(-1)
    DomainError()

如下改写 ``sqrt`` 函数，当参数为负数时，提示错误，立即停止执行： ::

    fussy_sqrt(x) = x >= 0 ? sqrt(x) : error("negative x not allowed")

    julia> fussy_sqrt(2)
    1.4142135623730951

    julia> fussy_sqrt(-1)
    negative x not allowed

当别的函数调用 ``fussy_sqrt`` 且参数为负数时，函数立即返回，并在交互式会话中显示错误信息： ::

    function verbose_fussy_sqrt(x)
      println("before fussy_sqrt")
      r = fussy_sqrt(x)
      println("after fussy_sqrt")
      return r
    end

    julia> verbose_fussy_sqrt(2)
    before fussy_sqrt
    after fussy_sqrt
    1.4142135623730951

    julia> verbose_fussy_sqrt(-1)
    before fussy_sqrt
    negative x not allowed

如果不想抛出错误，而是自己处理这种情形，可以使用关键字 ``try`` 和 ``catch`` 。下例通过处理 ``fussy_sqrt`` 抛出的错误，计算 ``x`` 绝对值的平方根： ::

    function sqrt_abs(x)
      try
        fussy_sqrt(x)
      catch
        fussy_sqrt(-x)
      end
    end

    julia> sqrt_abs(2)
    1.4142135623730951

    julia> sqrt_abs(-2)
    1.4142135623730951

写 ``sqrt(abs(x))`` 更简单高效，上例只是为了演示 ``try`` 和 ``catch`` 操作。

Throw 与 Error
~~~~~~~~~~~~~~

``error`` 函数说明有错误，这个函数是在 ``throw`` 函数的基础上构造的。上一节中的 ``try``-``catch`` 表达式，还有另外一种形式： ::

    try
      # execute some code
    catch x
      # do something with x
    end

如果 ``try`` 块儿调用了内建的 ``throw`` 函数， ``throw`` 函数的参数绑定给变量 ``x`` ，然后执行 ``catch`` 块儿。 ``error`` 函数每次都会抛出 ``ErrorException`` 类型的实例。下例中当出现除以零的错误时，抛出 ``DivideByZeroError`` 对象： ::

    julia> div(1,0)
    error: integer divide by zero

    julia> try
             div(1,0)
           catch x
             println(typeof(x))
           end
    DivideByZeroError

``DivideByZeroError`` 是 ``Exception`` 的具体子类型，抛出它表示有整数被零除。浮点函数会返回 ``NaN`` ，而不是抛出异常。

``error`` 应仅用来表示意外条件， ``throw`` 仅仅是个控制结构，可以对 ``try``-``catch`` 传递任何值： ::

    julia> try
             throw("Hello, world.")
           catch x
             println(x)
           end
    Hello, world.

.. _man-tasks:

任务（也称为协程）
------------------

任务是一种允许计算灵活地挂起和恢复的控制流，有时也被称为对称协程、轻量级线程、协同多任务等。

如果一个计算（比如运行一个函数）被设计为 ``Task`` ，有可能因为切换到其它 ``Task`` 而被中断。原先的 ``Task`` 在以后恢复时，会从原先中断的地方继续工作。切换任务不需要任何空间，同时可以有任意数量的任务切换，不需要考虑堆栈问题。任务切换与函数调用不同，可以按照任何顺序来进行。

任务比较适合生产者-消费者模式，一个过程用来生产值，另一个用来消费值。消费者不能简单的调用生产者来得到值，因为两者的执行时间不一定协同。在任务中，两者则可以正常运行。

Julia 提供了 ``produce`` 和 ``consume`` 函数来解决这个问题。生产者调用 ``produce`` 函数来生产值： ::

    function producer()
      produce("start")
      for n=1:4
        produce(2n)
      end
      produce("stop")
    end

要消费生产的值，先对生产者调用 ``Task`` 函数，然后对它重复调用 ``consume`` ： ::

    julia> p = Task(producer)
    Task

    julia> consume(p)
    "start"

    julia> consume(p)
    2

    julia> consume(p)
    4

    julia> consume(p)
    6

    julia> consume(p)
    8

    julia> consume(p)
    "stop"

可以在 ``for`` 循环中迭代任务，生产的值被赋值给循环变量： ::

    julia> for x in Task(producer)
             println(x)
           end
    start
    2
    4
    6
    8
    stop

注意 ``Task()`` 函数的参数，应为零参函数。生产者常常是参数化的，因此需要为其构造零参 :ref:`匿名函数 <man-anonymous-functions>` 。可以直接写，也可以调用宏： ::

    function mytask(myarg)
        ...
    end

    taskHdl = Task(() -> mytask(7))
    # 也可以写成
    taskHdl = @task mytask(7)

``produce`` 和 ``consume`` 适用于多任务，但它并不在不同的 CPU 发起线程。将在 :ref:`man-parallel-computing` 中，讨论真正的内核线程。

