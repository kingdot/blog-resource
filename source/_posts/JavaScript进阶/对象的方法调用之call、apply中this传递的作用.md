---
title: 对象的方法调用之call、apply中this传递的作用
date: 2019-1-16 11:39:01
tag: JavaScript核心
category: JavaScript
---
#### 当调用的函数作为对象的方法时，推荐使用call或者apply方法

> 形式为：someObj.someFunction.call(anotherObj, var1...)

比如想要使用数组的slice方法:
   > Array.prototype.slice(var1, var2);

可以采用：   
   - Array.prototype.slice.call(obj, var1, var2);
   - [].slice.call(obj, var1, var2);
   
显然，slice方法是Array的原型对象上的方法。意味着所有Array的实例都可以通过原型链查找到此方法。也即是说，可以以这种方式使用slice方法；

> [1,2,3,4].slice(1, 3); // [2,4]

但是，如果一个函数不位于原型对象中，而是仅仅作为某个特定对象的方法，此时，要想使用此方法时，只能使用someObj.someFunction(var1...);

然而，如果这个someFunction()的作用是对其宿主对象进行某些操作，即与宿主对象紧耦合的情况下，想把它分离出来复用，就不能再使用someObj.someFunction(var1...),只能使用someObj.someFunction.call/apply(anotherObj, var1...)。

综上，call和apply的作用其实是让对象的方法与其宿主对象解耦，便于另一个对象使用。而如果一个函数并不需要上下文，即与其宿主对象没有交互，此时可以直接使用someObj.someFunction(var1...)调用。

所以call、apply多适用于：同一个构造函数的不同实例，使用该构造函数原型对象的方法（公有方法），或者只是想切换this。

再比如，Object构造函数原型上有一个toString方法，即`Object.prototyoe.toString()`，所有Object的实例都可以使用此方法，然而其他诸如String、Boolean的实例可以通过call来使用：`Object.prototyoe.toString.call('test')`