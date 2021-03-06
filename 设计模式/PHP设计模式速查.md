# 设计模式

by HyperQing 20170511 禁止转载

>示例代码以 PHP 语言书写，使用PHP 7.0语法。

[TOC]

你可以在本文中了解到这些：

- 设计模式写成实际代码是怎样的。
- 以通俗语言解释设计模式的部分抽象概念。
- 简单提及一些模式的使用场合。
- 快速查阅代码模板。

本文不会提及这些：

- 某种设计模式的故事化描述。
- 某种设计模式的好处和优点。
- 不会指导如何应用到实际生产中。

## 一些原则

### 单一职责原则

术语：单一职责原则，就一个类而言，应该仅有一个引起它变化的原因。

一句话描述：一个类管好自己做的事，别什么都堆在一个类里做。

例如：业务流程和显示就可以分成两个类。常见的有：网站MVC框架等。

### 开放-封闭原则

术语：开放-封闭原则，是说软件实体（类、模块、函数等）应该可以扩展，但是不可修改。

一句话描述：对于可能会频繁修改的需求，应该通过添加新代码来实现，而不是修改旧代码。

例如：代码充斥着如同排比一样else来做业务分支，加业务就加else的，是时候该换成添加类了。

### 依赖倒转原则

术语：
A、高层模块不应该依赖低层模块。两个都应该依赖抽象。
B、抽象不应该依赖细节。细节应该依赖抽象。

一句话描述：面向接口编程。

例如：业务类使用了某基础服务类，当要重用业务类或更换基础服务类时，由于两者的结合调用时写死在代码里，想把其中一个类分离出来就相当困难了。如果业务类通过自己写的一个基础服务接口来调用基础服务，此时基础服务你爱怎么换就怎么换，而业务类重用时也不用过多考虑基础服务类的调用情况。

### 里氏代换原则

术语：子类型必须能够代替掉它们的父类型。

一句话描述：自己写个接口类，统一各种子类的使用方法，这个此时实例化子类时，爱换哪个换哪个，因为接口和用法都一样。

例如：自己写个缓存类时，可能同时有文件缓存类、MySQL缓存类、Redis缓存类，为了让这些类方便随时替换，这些类都实现同样的“缓存接口”，如“读方法”，“写方法”等，而且这些方法的用法是一样的。

下方示例代码最后实例化语句中，由于PHP是弱类型语言，故没有类型声明。在强类型语言中，应该声明为`CacheDriver`类型，使得`cache`变量可以兼容实现同一接口的子类。如果要替换缓存类型，直接修改`CacheFactory`的`getInstance`方法即可，或按自己需求实现实例化方法（如读取配置来确认实例化何种类型的缓存）。

```php
interface CacheDriver
{
    public function save();

    public function load();
}

class MysqlCache implements CacheDriver
{

    public function save()
    {
        // 写入缓存
    }

    public function load()
    {
        // 读取缓存
    }
}

class RedisCache implements CacheDriver
{

    public function save()
    {
        // 写入缓存
    }

    public function load()
    {
        // 读取缓存
    }
}

class CacheFactory
{
    public static function getInstanse()
    {
        return new RedisCache;
    }
}

$cache = CacheFactory::getInstanse();
$cache->save();
$cache->load();
```

## 简单工厂模式(Simple Factory Pattern)

一句话描述：该工厂类提供一个**静态方法**，通过给该方法传参得到具体的实例对象。（实例化什么、如何实例化看需求。）

说明：如果有大量`if`,`switch`的场合，为了避免误改其他代码，应使用这种模式实现，只需添加类或修改某个类即可，无需触碰其他类代码。

```php
// 若干个业务类
class A{}
class B{}
class C{}

// 简单工厂类
class SimpleFactory
{
    public static function getInstance($operation)
    {
        switch ($operation) {
            case 'a':
                return new A;
            case 'b':
                return new B;
            case 'c':
                return new C;
        }
        return null;
    }
}

$class_a = SimpleFactory::getInstance('a');
$class_b = SimpleFactory::getInstance('b');
$class_c = SimpleFactory::getInstance('c');
```

## 策略模式(Strategy Pattern)

上下文类通过构造函数传入不同的策略对象，最终调用策略对象同一个接口方法。

说明：通过添加策略类来丰富策略，具体想用哪个策略，实例化传参到上下文类即可。

```php
// 策略接口
interface Strategy
{
    public function algorithm();
}

// A策略
class A implements Strategy
{
    public function algorithm()
    {
        return 'A策略的算法';
    }
}

// B策略
class B implements Strategy
{
    public function algorithm()
    {
        return 'B策略的算法';
    }
}

// C策略
class C implements Strategy
{
    public function algorithm()
    {
        return 'C策略的算法';
    }
}

// 上下文类
class Context
{
    private $strategy;

    public function __construct(Strategy $strategy)
    {
        $this->strategy = $strategy;
    }

    public function getResult()
    {
        return $this->strategy->algorithm();
    }
}

// 使用策略A
$context = new Context(new A);
$context->getResult();
// 使用策略B
$context = new Context(new B);
$context->getResult();
// 使用策略C
$context = new Context(new C);
$context->getResult();
```
### 策略模式与简单工程模式结合

除了可以给上下文类传入策略对象，还可以传字符串，在上下文类构造函数里实例化指定策略类。

注：下面示例代码中，构造函数参数`string $str`的`string`类型声明是PHP 7.0.0以上才有的。

```php
// 策略接口
interface Strategy
{
    public function algorithm();
}

// A策略
class A implements Strategy
{
    public function algorithm()
    {
        return 'A策略的算法';
    }
}

// B策略
class B implements Strategy
{
    public function algorithm()
    {
        return 'B策略的算法';
    }
}

// C策略
class C implements Strategy
{
    public function algorithm()
    {
        return 'C策略的算法';
    }
}

// 上下文类
class Context
{
    private $strategy;

    public function __construct(string $str)
    {
        switch ($str) {
            case 'a':
                $this->strategy = new A;
                break;
            case 'b':
                $this->strategy = new B;
                break;
            case 'c':
                $this->strategy = new C;
                break;
        }
    }

    public function getResult()
    {
        return $this->strategy->algorithm();
    }
}

// 使用策略A
$context = new Context('a');
$context->getResult();
// 使用策略B
$context = new Context('b');
$context->getResult();
// 使用策略C
$context = new Context('c');
$context->getResult();
```

## 装饰模式(Decorator Pattern)

一句话描述：类A存有类B对象，类B又存有类C对象，他们都实现相同的接口。当类A执行动作方法时，会自动调用B的动作方法，B执行动作也会自动调用，C的动作方法，以此类推实现有顺序的执行方式。

说明：使用效果形似链表。

除了下面代码这种写法，还有利用构造函数代替then()方法的写法。

如下图代码所示（暂时不看接口和超类），核心在于类A、B、C都有`$filter`成员属性、`then()`方法和`doThis()`方法。

- `then()`用于存入下一个要操作的 filter。如`$filterA->then($filterB);`，即运行A后的动作后就执行B。
- `doThis()`是当前对象要做的事，在方法最后应使用`$this->filter->doThis()`来调用下一个对象的`doThis()`方法。

由于类A、B、C都要求实现相同的接口(`then()`,`doThis()`)继承相同的成员(`$filter`)，故增加接口和超类。并且将`$this->filter->doThis()`换成`parent::doThis()`。
```php
// 通过装饰模式实现过滤器
interface FilterInterface
{
    /**
     * 指定下一个要执行的过滤器
     * @param FilterInterface $filter
     */
    public function then(FilterInterface $filter);

    /**
     * 该过滤器要做的事情
     */
    public function doThis();
}

class SuperFilter implements FilterInterface
{
    /**
     * @var FilterInterface $filter 下一个过滤器对象
     */
    protected $filter;

    /**
     * 指定下一个要执行的过滤器
     * @param FilterInterface $filter
     */
    public function then(FilterInterface $filter)
    {
        $this->filter = $filter;
    }

    /**
     * 该过滤器要做的事情
     */
    public function doThis()
    {
        if ($this->filter) {
            $this->filter->doThis();
        }
    }
}

class FilterA extends SuperFilter
{
    public function doThis()
    {
        echo "执行过滤器A\n";
        parent::doThis();
    }
}

class FilterB extends SuperFilter
{
    public function doThis()
    {
        echo "执行过滤器B\n";
        parent::doThis();
    }
}

class FilterC extends SuperFilter
{
    public function doThis()
    {
        echo "执行过滤器C\n";
        parent::doThis();
    }
}

$filterA = new FilterA();
$filterB = new FilterB();
$filterC = new FilterC();
$filterB->then($filterA);
$filterC->then($filterB);
$filterC->doThis();
echo "======\n";
$filterA = new FilterA();
$filterB = new FilterB();
$filterC = new FilterC();
$filterB->then($filterC);
$filterA->then($filterB);
$filterA->doThis();
```
输出结果
```
执行过滤器C
执行过滤器B
执行过滤器A
======
执行过滤器A
执行过滤器B
执行过滤器C
```

## 代理模式(Proxy Pattern)

一句话描述：代理类和实体类实现相同的接口，由代理类调用实体类的方法。

```php
interface HttpInterface
{
    public function getBody();

    public function getHeader();
}

class Http implements HttpInterface
{

    public function getBody()
    {
        echo 'body';
    }

    public function getHeader()
    {
        echo 'header';
    }
}

class Proxy implements HttpInterface
{
    /**
     * @var Http
     */
    private $Http;

    public function __construct()
    {
        $this->Http = new Http();
    }

    public function getBody()
    {
        $this->Http->getBody();
    }

    public function getHeader()
    {
        $this->Http->getHeader();
    }
}

$proxy=new Proxy();
$proxy->getHeader();
$proxy->getBody();
```
输出样例
```
header
body
```

## 工厂方法模式(Factory Pattern)

一句话描述：在简单工厂模式的基础上，连工厂类都进一步抽象和子类化。

```
class A{}
class B{}
class C{}

//  工厂接口
interface FactoryInterface{
    public static function getInstance();
}

// 具体工厂类
class AFactory implements FactoryInterface
{
    public static function getInstance()
    {
        return new A;
    }
}
// 具体工厂类
class BFactory implements FactoryInterface
{
    public static function getInstance()
    {
        return new B;
    }
}
// 具体工厂类
class CFactory implements FactoryInterface
{
    public static function getInstance()
    {
        return new C;
    }
}
// 这里三个实例变量，在强类型语言中应声明为 FactoryInterface 类型
$class_a = AFactory::getInstance();
$class_b = BFactory::getInstance();
$class_c = CFactory::getInstance();
```

## 原型模式(Prototype Pattern)

一句话描述：使用语言自带的`clone()`方法。

```
class A
{
    public $str = 'hello';

    public $obj;

    public function __construct()
    {
        $this->obj = new stdClass();
    }

    // 深复制
    public function __clone()
    {
        $this->obj = clone $this->obj;
    }
}

$a = new A;
$b = $a; // 这是传引用
$b->str = 'world'; // 对$b对象的修改，将影响$a
echo $a->str; // 输出'world'

$a = new A;
$b = clone $a; // 正确复制对象到$b
$b->str = 'world'; // 对$b对象的修改，不会影响$a
echo $a->str; // 输出'hello'
// 这里输出的对象id是不同的，如果是浅复制，则id相同，引用了同一个对象
var_dump($a->obj);
var_dump($b->obj);
```

## 单例模式(Singleton Pattern)

一句话描述：该类包含一个**静态方法**和**静态变量**，静态方法会实例化指定类，并存放对象到静态变量中，如果静态变量已存在对象则直接返回，这样即使多次调用静态方法都返回同一个对象。

例如：数据库连接对象、缓存对象、重复使用的对象、贯穿生命周期的 Request 请求对象（Web框架概念）等。

下面例子中，输出结果显示，使用`DbDriver::getInstance()`来实例化时，对象id都是`#1`，表示第一第二步得到的是同一个对象。当第三步进行手动实例化时，对象id变成`#2`，意味着产生了新的对象。

```php
class DbDriver
{
    private static $obj;

    public static function getInstance()
    {
        if (self::$obj) {
            return self::$obj;
        } else {
            self::$obj = new stdClass();
            return self::$obj;
        }

    }
}
// 第一步
$db = DbDriver::getInstance();
echo "第一个对象";
var_dump($db);
// 第二步
$db = DbDriver::getInstance();
echo "第二个对象";
var_dump($db);
// 第三步
$db = new stdClass();
echo "第三个对象";
var_dump($db);
```
输出结果
```
第一个对象object(stdClass)#1 (0) {
}
第二个对象object(stdClass)#1 (0) {
}
第三个对象object(stdClass)#2 (0) {
}
```

## 版权

禁止转载。转到自己博客等可能公开的位置都不给。要看就在这里看，推荐收藏，不推荐下载保存，因为不定时更新。
禁止未授权出版。
