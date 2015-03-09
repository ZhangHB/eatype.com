# 自执行匿名函数

When learning JavaScript, with all the attention given to variables, functions, ‘if’ statements, loops and event handlers, often little is done to educate you on how you might cleanly organise your code into a cohesive, structurally-sound whole.

当我们在学习 JS 的时候，更多关注的是变量、函数、if 语句、循环和事件处理程序。而往往忽略了对于如何干净利落的组织代码，使代码更具凝聚力和一致性的相关学习。

Let’s take the following code for example:

让我们看看下边这个例子：

    var foo = 'Hello';
    var bar = 'World!';

    function baz(){
      return foo  + ' ' + bar;
    }

    console.log(baz());

This style of code looks quite normal, works fine and doesn’t cause any problems. At least for now.

这写代码的风格看起来很正常，而且能正常运行，并且至少现在不会造成任何问题。

