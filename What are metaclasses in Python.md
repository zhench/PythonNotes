# <center> What are metaclasses in Python?

原文地址：https://stackoverflow.com/a/6581949/9612118

在理解元类之前，需要先掌握Python中的类（classes）。在python中，类是十分特殊的概念，Python中的类是从SmallTalk中借用来的。

在大多数语言中，类是一段描述怎样生成对象的代码。在Python中也是一样。

``` python
>>> class ObjectCreator(object):
...     pass
... 
>>> my_object= ObjectCreator()
>>> print(my_object)
<__main__.ObjectCreator object at 0x7f9a336c8438>
```

但是Python的的类也不只是生成对象的代码，Python中的类也是对象。

在Python中使用关键字class时，Python会执行代码并创建一个**对象**，下面这段代码：
``` python
>>> class ObjectCreator(object):
...     pass
```
在内存中创建了一个名为**ObjectCreator**的对象。

**这个对象（或者类）是一个具有创建对象（实例）的能力的对象——这就是为什么它是类。**

但是它仍然是一个对象，所以它有以下特点：
* 可以复制给一个变量
* 可以复制
* 可以增加属性
* 可以作为参数传递给一个函数

比如：

```python
>>> print(ObjectCreator)
<class '__main__.ObjectCreator'>
>>> def echo(o):
...     print(o)
... 
>>> echo(ObjectCreator)
<class '__main__.ObjectCreator'>
>>> print(hasattr(ObjectCreator,'new_attribute'))
False
>>> ObjectCreator.new_attribute='foo'
>>> print(hasattr(ObjectCreator,'new_attribute'))
True
>>> print(ObjectCreator.new_attribute)
foo
>>> ObjectCreatorMirror=ObjectCreator
>>> print(ObjectCreatorMirror())
<__main__.ObjectCreator object at 0x7f9a3300e588>
```

### 动态创建类
类既然是一个对象（类对象），那么就可以像一样在运行时创建。

首先，可以在函数中用class关键字创建一个类对象。

``` python
def choose_class(name):
    if name == 'foo':
        class Foo(object):
            pass
        return Foo
    else:
        class Bar(object):
            pass
        return Bar

MyClass=choose_class('foo')
print(MyClass)
print(MyClass())

OUTPUT:
<class '__main__.choose_class.<locals>.Foo'>
<__main__.choose_class.<locals>.Foo object at 0x0000018B38FD6C88>
```

但是这样不是那么的动态（在运行时创建类对象），依然需要把整个类的代码都写出来。

因为类也是对象（类对象），所以类也是被某种东西生成的。

当使用class关键字时，Python自动地创建类对象（也就是类）。但是和Python中其他的对象一样，允许手动创建。

### Type

可以通过一个简单的函数（print）输出一个对象是什么类型：

```python
>>> print(type(1))
<class 'int'>
>>> print(type("1"))
<class 'str'>
>>> print(type(ObjectCreator()))
<class '__main__.ObjectCreator'>
```
**type** 还有一个完全不同的能力，它也可以在运行时创建类。type接受类的描述作为参数，然后返回一个类（对象）

type的用法如下：
```python
type(name of the class,
     tuple of the parent class (for inheritance, can be empty),
     dictionary containing attributes names and values)
```
比如下面的类可以用type来创建
```python
>>> class MyShinyClass(object):
...       pass
```
用type来创建：
```python
>>> MyShinyClass=type('MyShinyClass',(),{})
>>> print(MyShinyClass)
<class '__main__.MyShinyClass'>
>>> print(MyShinyClass())
<__main__.MyShinyClass object at 0x7f9a3300e588>
```

"MyShinyClass"即是类的名字，也是指向类对象的变量。它们可以不同（类似于ObjectCreatorMirror和ObjectCreator），但是没有必要弄得那么复杂。

type接受一个字典值来定义类的属性。
```python
>>> Foo = type('Foo', (), {'bar':True})
```
同样也可以当做普通类来使用
```python
>>> print(Foo)
<class '__main__.Foo'>
>>> print(Foo.bar)
True
>>> f = Foo()
>>> print(f)
<__main__.Foo object at 0x8a9b84c>
>>> print(f.bar)
True
```
也可以用来继承
```python
>>>   class FooChild(Foo):
...         pass
>>> FooChild = type('FooChild', (Foo,), {})
>>> print(FooChild)
<class '__main__.FooChild'>
>>> print(FooChild.bar) # bar is inherited from Foo
True
```
还可以给类增加方法，只要定义一个函数，然后作为一个属性赋值给类。

```python
>>> def echo_bar(self):
...       print(self.bar)
...
>>> FooChild = type('FooChild', (Foo,), {'echo_bar': echo_bar})
>>> hasattr(Foo, 'echo_bar')
False
>>> hasattr(FooChild, 'echo_bar')
True
>>> my_foo = FooChild()
>>> my_foo.echo_bar()
True
```

在动态创建类（对象）之后也可以增加方法，就像给用普通方法创建的类一样
```python
>>> def echo_bar_more(self):
...       print('yet another method')
...
>>> FooChild.echo_bar_more = echo_bar_more
>>> hasattr(FooChild, 'echo_bar_more')
True
```

可以看出在Python中，类就是对象，可以在运行时创建。
在使用**class**关键字时，Python就进行了这些处理。用metaclass也可以做到。

### 什么是元类
元类是用于创建类的。
类是用来创建对象的，在Python中类也是对象，而元类就是用来创建类的，是类的"类"，关系如下：
> MyClass = MetaClass()
> my_object = MyClass()

type 的用法是 
> MyClass = type('MyClass', (), {})

这是由于`type`实际上是一个元类。`type`是Python用来创建所有类的元类。
那么type为什么是小写的而不是用Type呢（类名首字母一般为大写）？可能是为了和字符类型的关键字`str`、整型类型的`int`保持一致。`type`只是用于创建类对象的类。

可以通过"__class__"属性类查看。

在Python中所有的东西都是对象，包括整型、字符串、函数以及类。所有的都是对象，这些对象都是由一个类创建的。

```python
>>> age = 35
>>> age.__class__
<type 'int'>
>>> name = 'bob'
>>> name.__class__
<type 'str'>
>>> def foo(): pass
>>> foo.__class__
<type 'function'>
>>> class Bar(object): pass
>>> b = Bar()
>>> b.__class__
<class '__main__.Bar'>
```
任意一个`__class__`的“__class__”又是什么呢？

```python
>>> age.__class__.__class__
<type 'type'>
>>> name.__class__.__class__
<type 'type'>
>>> foo.__class__.__class__
<type 'type'>
>>> b.__class__.__class__
<type 'type'>
```
所以元类就是用来创建类对象的，可以称之为**类工厂**。`type`是Python内建的元类，同样的元类也可自定义。

**__metaclass__**属性
在声明一个类时可以使用`__metaclass__`
```python
class Foo(object):
	__metaclass__ = something...
	pass
```
这样的话，Python就会使用元类来创建Foo类。
使用元类时要十分小心。当Pyton读取到`class Foo(object)`时，Foo类对象还没有在内存中创建。Python会转到`__metaclass`指向的元类定义，如果找到，就会用这个元类来创建Foo类对象。如果没有找到，Python就会用`type`来创建类。

下面的代码:
```python
class Foo(Bar):
    pass
```
Python会有以下执行过程：
1. Foo中是否有`__metaclass__`
2. 如果有，使用指向的元类在内存中创建类对象。
3. 如果没有，就会在模块级别寻找`__metaclass__`,然后用它来创建类（不过只是创建没有继承任何父类的类，也就是经典类）
4. 如果没有找到任何`__metaclass__`，就会使用`Bar`（第一个父类）的元类（可能是默认的type）来创建类对象。

>**注意：** `__metaclass__`属性并不会继承给子类，但是父类的元类是可以继承的（Bar.__class__)。如果`Bar`用`__metaclass__`属性类创建Bar类（用type()而不是`type.__new__()`，子类是不会继承的。

`__metaclass__`指向的时可以创建类的东西。什么可以创建类呢？那就是`type`,或其子类、或使用type的东西。

### 自定义元类

使用元类的主要目的是在类创建时动态的修改类对象。在API中经常会创建类来与当前上下文相匹配。举一个愚蠢的例子吧，在所有的类中让所有的属性名称用大写字母。有多重方法可以做到这一点，其中一个方法就是在模块级别使用元类。这个方法，在该模块中所有的类就会用这个元类来创建，在这个元类中就会令所有的属性都使用大写字母。

`__metaclass__`可以是任何可以被调用的东西，没有必要必须是一个正式的类（在名称中有class也不是必须是类）。

下面这个例子就是使用函数的：
```python
# the metaclass will automatically get passed the same argument
# that you usually pass to `type`
def upper_attr(future_class_name, future_class_parents, future_class_attr):
    """
      Return a class object, with the list of its attribute turned
      into uppercase.
    """

    # pick up any attribute that doesn't start with '__' and uppercase it
    uppercase_attr = {}
    for name, val in future_class_attr.items():
        if not name.startswith('__'):
            uppercase_attr[name.upper()] = val
        else:
            uppercase_attr[name] = val

    # let `type` do the class creation
    return type(future_class_name, future_class_parents, uppercase_attr)

__metaclass__ = upper_attr # this will affect all classes in the module

class Foo(): # global __metaclass__ won't work with "object" though
    # but we can define __metaclass__ here instead to affect only this class
    # and this will work with "object" children
    bar = 'bip'

print(hasattr(Foo, 'bar'))
# Out: False
print(hasattr(Foo, 'BAR'))
# Out: True

f = Foo()
print(f.BAR)
# Out: 'bip'
```
下面的例子是使用一个真正的类来实现的：
```python
# remember that `type` is actually a class like `str` and `int`
# so you can inherit from it
class UpperAttrMetaclass(type):
    # __new__ is the method called before __init__
    # it's the method that creates the object and returns it
    # while __init__ just initializes the object passed as parameter
    # you rarely use __new__, except when you want to control how the object
    # is created.
    # here the created object is the class, and we want to customize it
    # so we override __new__
    # you can do some stuff in __init__ too if you wish
    # some advanced use involves overriding __call__ as well, but we won't
    # see this
    def __new__(upperattr_metaclass, future_class_name,
                future_class_parents, future_class_attr):

        uppercase_attr = {}
        for name, val in future_class_attr.items():
            if not name.startswith('__'):
                uppercase_attr[name.upper()] = val
            else:
                uppercase_attr[name] = val

        return type(future_class_name, future_class_parents, uppercase_attr)
```
这样不太符合OOP思想，是直接调用type，而不是重写或调用父类的`__new__`方法。下面的代码是改写之后的：
```python
class UpperAttrMetaclass(type):

    def __new__(upperattr_metaclass, future_class_name,
                future_class_parents, future_class_attr):

        uppercase_attr = {}
        for name, val in future_class_attr.items():
            if not name.startswith('__'):
                uppercase_attr[name.upper()] = val
            else:
                uppercase_attr[name] = val

        # reuse the type.__new__ method
        # this is basic OOP, nothing magic in there
        return type.__new__(upperattr_metaclass, future_class_name,
                            future_class_parents, uppercase_attr)
```
这里有一个额外的参数`upperattr_metaclass`，这个参数没有什么特殊的，`__new__`会接受类本身的定义作为第一个参数，就像在类的方法中用`self`接受一个实例作为第一个参数，或者类的定义。这里的`upperattr_metaclass`是比较长的，和`self`一样，所有的参数会有一个合适的名称。所以真正的元类会是这样的：
```python
class UpperAttrMetaclass(type):

    def __new__(cls, clsname, bases, dct):

        uppercase_attr = {}
        for name, val in dct.items():
            if not name.startswith('__'):
                uppercase_attr[name.upper()] = val
            else:
                uppercase_attr[name] = val

        return type.__new__(cls, clsname, bases, uppercase_attr)
```
使用`super`可以时代码更加简洁，而且能够允许继承（元类可以继承自元类，继承自`type`），如下面代码：
```python
class UpperAttrMetaclass(type):

    def __new__(cls, clsname, bases, dct):

        uppercase_attr = {}
        for name, val in dct.items():
            if not name.startswith('__'):
                uppercase_attr[name.upper()] = val
            else:
                uppercase_attr[name] = val

        return super(UpperAttrMetaclass, cls).__new__(cls, clsname, bases, uppercase_attr)
```
>The reason behind the complexity of the code using metaclasses is not because of metaclasses, it's because you usually use metaclasses to do twisted stuff relying on introspection, manipulating inheritance, vars such as `__dict__`, etc.

元类就是Python中的黑魔法，所示比较复杂难懂。但就它本身而言，是比较简单的。元类所做的就是：

 - 截断类的创建过程
 - 修改类
 - 返回修改后的类

### 为什么使用元类而不是函数？
既然`__metaclass__`接受的时可以调用的东西，那么为什么要用类呢，难道就是因为类明显复杂么？

有这么几个原因：

- 目的明显。当在代码中看到`UpperAttrMetaclass(type)时，就会知道会发生什么。
- 可以使用OOP思想。元类可以继承自元类，重写父类方法，元类还可以使用元类。
- 类的元类使用类，那么该类的子类可以继承其元类，使用函数则不行。
- 可以更好的构建代码。一般不会像上面那个愚蠢例子那样使用元类，通常用在复杂的地方。元类具有创建方法并组织方法的能力（类中相似的方法或同组的方法用元类来增加），对于代码增强可读性很有帮助。
- 可以拦截`__new__`,`__init__`,`__call__`,拦截之后就可以修改它们。有的人喜欢全部在`__new__`中修改，而有的是在`__init__`中。
- 被称作元类，嗯，肯定是有用的！

### 为什么使用元类？
最大的问题是为什么使用可能出问题的特性呢？
>Metaclasses are deeper magic that 99% of users should never worry about. If you wonder whether you need them, you don't (the people who actually need them know with certainty that they need them, and don't need an explanation about why).
>---Python Guru Tim Peters

元类主要用于创建API。Django中的ORM就是一个典型的例子。
ORM允许向下面这样定义对象：
```python
class Person(models.Model):
    name = models.CharField(max_length=30)
    age = models.IntegerField()
```

但是在下面的代码中：
```python
guy = Person(name='bob', age='35')
print(guy.age)
```
并不会返回一个`IntegerField`,而是会返回一个`int`,并且可以在数据库中直接取出来。
这很可能是因为在models.Model中定义了`__metaclass__`,使用了一些技巧，来返回数据库中的字段。
Django通过提供简单API以及使用元类来让一些复杂的过程看起来比较简单，在API中重新生成代码来在后台处理真正的工作。

### 最后
首先，类是可以创建实例的对象。
实际上，类就是自己的实例，对于元类：
```python
>>> class Foo(object): pass
>>> id(Foo)
142630324
```
在Python中，所有东西都是对象，他们或者是类的实例，或者是元类的实例。**`type`**除外。type实际上是自己的元类。在Python中type不是简单的再生成的，在Python的实现层次中，用了一点特殊的方法来实现type的。

其次，元类比较复杂难懂。在简单类的修改中不必使用。可以通过以下两中不同的方法来实现类的而修改：

- monkey patching(猴子补丁)
- class decorators(类装饰器)