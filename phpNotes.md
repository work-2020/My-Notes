# PHP

## 数据类型

### 类型检测

- `var_dump()` 函数可以查看表达式的值和类型
- `gettype()` 函数用于检测变量类型
- `is_` 加类型，如 `is_int()`，`is_array()` 等，判断变量是否为该类型

### 布尔类型

`TRUE/FALSE`不区分大小写。

**注意**当下列情况转换为 Boolean 时，其值为 FALSE

- 布尔值 FALSE 本身
- 整型值 0（零）
- 浮点型值 0.0（零）
- 空字符串，以及字符串 "0" 
- 不包括组
- 特殊类型 NULL（包括尚未赋值的变量）
- 从空标记生成的 SimpleXML 对象

### 整数类型

整型值可以使用十进制，十六进制，八进制或二进制表示，前面可以加上可选的符号（- 或者 +）。

**注意**：要使用八进制表达，数字前必须加上 `0`。要使用十六进制表达，数字前必须加上 `0x`。要使用二进制表达，数字前必须加上 `0b`。

**整数溢出**：如果给定的一个数超出了 integer 的范围，将会被解释为 float。同样如果执行的运算结果超出了 integer 范围，也会返回 float。

### 字符串

php字符串有4种表达方式：

- **单引号**：单引号内特殊字符和变量不会被解析。

- **双引号**：双引号内的特殊字符和变量会被解析。

- **Heredoc**：Heredoc 类似与双引号，内部转义字符和变量可以被解析，句法结构为

  ```
  <<<EOT
  
  字符串
  
  EOT;
  ```

  其中 `EOT` 为标识符，可以自定义，但是首尾标识符必须相同。开始标识符 EOT 后需换行，结束标识符 EOT 必须独占一行，且前面不许有空格。

- **Nowdoc**：Nowdoc 类似于单引号，无法解析转移字符和变量。句法结构类似 Heredoc，但是需要在开始标识符加上单引号。

### Array数组

数组实际上是一个有序映射。映射是一种把 values 关联到 keys 的类型。由于数组元素的值也可以是另一个数组，树形结构和多维数组也是允许的。

定义数组可以用 `array()` 或 `[]` 来新建一个数组。它接受任意数量用逗号分隔的键（key） => 值（value）对。key 可以是 integer（索引数组）或者 string（关联数组），value 可以是任意类型，如对象、数组。

- 如果没有键名，则数组默认使用从 0 开始的数字键名
- 打印数组不存在的 key 的值时，直接返回 NULL
- 数组可以多维嵌套，通过键名可以获取特定值

### Object对象

使用 `new` 可以创建一个新的对象。

```php
<?php

class foo
{
    function do()
    {
        echo "Action do"; 
    }
}

$f = new foo;
$f->do();
```

**转换为对象**

如果将一个对象转换成对象，它将不会有任何变化。如果其它任何类型的值被转换成对象，将会创建一个内置类 stdClass 的实例。如果该值为  NULL，则新的实例为空。 array 转换成 object 将使键名成为属性名并具有相对应的值，除了数字键，不迭代就无法被访问。

字符串 "A" 转换为对象时，自动生成 scalar 属性。

数组 ['hello'=>'world'] 转换为对象时，键名 hello 作为属性，键值 world 为属性值。

### 类型转换

PHP隐式转换的优先级为：浮点型 > 整型 > 字符串。

**强制转换**

```
-  (int), (integer) - 转换为整形 integer
-  (bool), (boolean) - 转换为布尔类型 boolean
-  (float), (double), (real) - 转换为浮点型 float
-  (string) - 转换为字符串 string
-  (array) - 转换为数组 array
-  (object) - 转换为对象 object
-  (unset) - 转换为 NULL (PHP 5)

```



## 变量

**预定义变量**

PHP 提供了大量的预定义变量。其中一些变量依赖于运行的服务器的版本和设置，及其它因素。

- `$GLOBALS` — 引用全局作用域中可用的全部变量
- `$_SERVER` — 服务器和执行环境信息
- `$_GET` — HTTP GET 变量
- `$_POST` — HTTP POST 变量
- `$_FILES` — HTTP 文件上传变量
- `$_REQUEST` — HTTP Request 变量
- `$_SESSION` — Session 变量
- `$_ENV` — 环境变量
- `$_COOKIE` — HTTP Cookies
- `$php_errormsg` — 前一个错误信息
- `$HTTP_RAW_POST_DATA` — 原生POST数据
- `$http_response_header` — HTTP 响应头



## 常量

**定义方式**

+ 使用 `define()` 函数定义

  ```php
  <?php
  
  define('HELLO', 'Hello');
  
  //使用 defined() 来判断一个常量是否被定义
  defined('SHIYANLOU') or define('SHIYANLOU', 'shiyanlou');
  ```

+ 注意使用 `const` 只能在类外部定义，且必须处于最顶端的作用区域，因为用此方法是在编译时定义的。这就意味着不能在函数内，循环内以及 if 语句之内用 const 来定义常量。

  ```php
  <?php
  
  const HELLO = 'Hello';
  const SHIYANLOU = 'shiyanlou';
  
  class Test
  {
      public function sayHi()
      {
          define(HELLO, 'Hi');
          echo HELLO;
      }
  }
  
  $t = new Test();
  $t->sayHi();
  ```

  - 常量前面没有美元符号 `$`
  - 常量只能通过 `define()` 或 `const` 定义，而不能通过赋值语句
  - 常量可以不用理会变量的作用域而在任何地方定义和访问
  - 常量一旦定义就不能被重新定义或者取消定义
  - 常量的值只能是标量

**魔术变量**

有八个魔术常量它们的值随着它们在代码中的位置改变而改变。例如 `__LINE__` 的值就依赖于它在脚本中所处的行来决定，这些特殊的常量不区分大小写。

- `__LINE__`，文件中的当前行号。
- `__FILE__`，文件的完整路径和文件名。如果用在被包含文件中，则返回被包含的文件名。
- `__DIR__`，文件所在的目录。如果用在被包括文件中，则返回被包括的文件所在的目录。
- `__FUNCTION__`，函数名称，返回该函数被定义时的名字。
- `__CLASS__`，类的名称，返回该类被定义时的名字。
- `__TRAIT__`，Trait 的名字（PHP 5.4.0 新加）。自 PHP 5.4 起此常量返回 trait 被定义时的名字（区分大小写）。Trait 名包括其被声明的作用区域（例如 Foo\Bar）。
- `__METHOD__`，类的方法名，返回该方法被定义时的名字（区分大小写）。
- `__NAMESPACE__`，当前命名空间的名称（区分大小写）。



## 运算符

**比较运算符**

如同它们名称所暗示的，允许对两个值进行比较。

- `$a == $b`，如果类型转换后 a等于a 等于 a等于b，返回 TRUE。
- `$a === $b`，如果 a等于a 等于 a等于b，并且它们的类型也相同，返回 TRUE。
- `$a != $b`，如果类型转换后 a不等于a 不等于 a不等于b，返回 TRUE。
- `$a <> $b`，等同于 `!=`
- `$a !== $b`，如果 a和a 和 a和b 的值或类型不同，返回 TRUE。
- `$a < $b`    ，如果 a严格小于a 严格小于 a严格小于b，返回 TRUE。
- `$a > $b`，如果 a严格大于a 严格大于 a严格大于b，返回 TRUE。
- `$a <= $b`，如果 a小于或者等于a 小于或者等于 a小于或者等于b，返回 TRUE。
- `$a >= $b`，如果 a大于或者等于a 大于或者等于 a大于或者等于b，返回 TRUE。

注意：

- null 或 String 和 string 比较时，将 null 转换为 ""，进行数字或词汇比较
- bool 或 null 和其他类型比较时，转换为 bool，FALSE < TRUE
- string，resource 或 number 相互比较时，将字符串或资源转换为数字，按普通数字比较
- array 之间比较时，具有较少成员的数组较小
- object 和其他类型比较时，object 总是更大
- array 和其他类型比较时，array 总是更大，但是比对象小
- switch 中第一个条件满足时，不会执行后面满足条件的语句

**错误控制运算符**

PHP 支持一个错误控制运算符：`@`。当将其放置在一个 PHP 表达式之前，该表达式可能产生的任何错误信息都被忽略掉。

**执行运算符**

PHP 支持一个执行运算符：反引号（``）。注意这不是单引号！PHP 将尝试将反引号中的内容作为外壳命令来执行，并将其输出信息返回。

**字符串运算符**

- 第一个是连接运算符 `.`，它返回其左右参数连接后的字符串
- 第二个是连接赋值运算符 `.=`，它将右边参数附加到左边的参数后。

**数组运算符**

常见数组运算符

- `$a + $b`，a和a 和 a和b 的联合
- `$a == $b`，a和a 和 a和b 键和值都相同则为 TRUE
- `$a === $b`，a和a 和 a和b 键和值且顺序和类型都相同返回 TRUE
- `$a != $b`，a和a 和 a和b 中键或值不同返回 TRUE
- `$a <> $b`，等同于 !=
- `$a !== $b`，a和a 和 a和b 中键，值，顺序或类型，其中一个不相同则返回 TRUE

注意

- `+` 运算符把右边的数组元素（除去键值与左边的数组元素相同的那些元素）附加到左边的数组后面，但是重复的键值不会被覆盖。
- `===`，需要数组的键，值，类型和顺序都相同，才返回 TRUE。

**NULL合并运算符**

NULL 合并运算符使用 `??` 表示，意味着如果 `??` 之前的变量存在且值不为 NULL，它就会返回自身的值，否则返回 `??` 后的操作数。

## 控制结构

**if语句**

```php
<?php
if ($a > $b) {
  echo "a is greater than b";
} else {
  echo "a is NOT greater than b";
}
```

```php
<?php
if ($a > $b) {
    echo "a is bigger than b";
} elseif ($a == $b) {
    echo "a is equal to b";
} else {
    echo "a is smaller than b";
}
```

**foreach**

```php
foreach (array as $value)
    statement
foreach (array as $key => $value)
    statement
```

- 第一种格式遍历给定的 array 数组。每次循环中，当前单元的值被赋给 $value 并且数组内部的指针向前移一步。 
- 第二种格式做同样的事，只除了当前单元的键名也会在每次循环中被赋给变量 $key。 

**break**

break 结束当前 for，foreach，while，do-while 或者 switch 结构的执行。 **break 可以接受一个可选的数字参数来决定跳出几重循环**。 

**continue**

continue 在循环结构用来跳过本次循环中剩余的代码并在条件求值为真时开始执行下一次循环。 

**Note: 注意在 PHP 中 switch 语句被认为是可以使用 continue 的一种循环结构。**

continue 接受一个可选的数字参数来决定跳过几重循环到循环结尾。默认值是 1，即跳到当前循环末尾。 

**switch**

当有多个 if elseif 条件时，可以使用 switch 来替换。

```php
<?php

switch ($a) {
    case 'apple':
        //do something
    break;    
    case 'bnanan':
        //do something
    break;    
    case 'oringe':
        //do something
    break;    
    default:
        //do something
}
```

**declare**语句

一般用法是 declare(ticks=N);

每执行N条低级语句就去执行一次 register_tick_function() 注册的函数。
 可以粗略的理解为每执行一句php代码（例如:$num=1;）就去执行下数。

```php
declare(ticks=1);
// 开始时间
$time_start = time();
// 检查是否已经超时
function check_timeout(){
     // 开始时间
     global $time_start;
    // 5秒超时
   $timeout = 5;
   if(time()-$time_start > $timeout){
       exit("超时{$timeout}秒\n");
   }
}
// Zend引擎每执行一次低级语句就执行一下check_timeout
register_tick_function('check_timeout');
 
while(1){
    $num = 1;
}
```

上述代码虽然最后有个死循环，但是执行时间不会超过5秒。

**包含文件**

- include
- include_once
- require
- require_once

include 和 require 都可以加载文件，不同点在于如果加载的文件包含错误，include 发出警告，继续执行后面的语句，而 require 则发出致命错误，终止程序执行。

include_once 和 require_once 的区别同 include 和 require，都是遇到错误时是否继续执行。

include 和 include_once，以及 require 和 require_once 的区别在于是否进行重复检测。

当一个文件被包含时，其中所包含的代码继承了 include 所在行的变量范围。从该处开始，调用文件在该行处可用的任何变量在被调用的文件中也都可用。不过所有在包含文件中定义的函数和类都具有全局作用域。 

**可变函数**

```php
<?php

class Test
{
    public static $actionB = "property B";

    public function actionA()
    {
        echo "method A";
    }
    public static function actionB()
    {
        echo "method B";
    }
}

function sayHi() {
    echo "Hi".PHP_EOL;
}

function sayHello($word = '') {
    echo "Hello $word";
}


$func = 'sayHi';
$func();

$func = 'sayHello';
$func('World');

$func = 'actionA';
(new Test())->$func();

echo Test::$actionB;
$actionB = 'actionB';
Test::$actionB();

```

## 类与对象

**创建实例**

```php
<?php
class A
{
    //默认值
    public $a = 'Hi';
    private $b = 'Hello';

    //方法
    public function actionA()
    {

    }
}

$a = new A();//创建类 A 的实例
```

**对象继承**

一个类可以在声明中用 `extends` 关键字继承另一个类的方法和成员。PHP 不支持多重继承，一个类只能继承一个类。

被继承的方法和成员可以通过用同样的名字重新声明被覆盖，除非父类定义方法时使用了 `final` 关键字。可以通过 `parent::` 来访问被覆盖的方法或成员。

**属性**

类的变量成员叫做`属性`。属性声明是由访问控制关键字 `public`，`protected`  或 `private` 和一个变量来组成，同时可以加上默认值。

在类的成员方法里面，可以通过 `$this->` 加变量名来访问类的属性和方法，但是要访问类的静态属性或者在静态方法要使用 `self::` 加变量名。

注意 `self::` 这种方式后的变量名需要加 `$` 符号，而 `$this->` 后的变量名不需要加

```php
<?php 

class A
{
    private $a = "Hello";

    protected $b = <<<EOT
This is property b
EOT;

    public static $c = 'This is a'.' static property';

    public function talk()
    {
        echo $this->a.PHP_EOL;
        echo $this->b.PHP_EOL;
        echo self::$c;
    }
}

(new A())->talk();
```

**类常量**

我们可以在类中定义常量。常量的值将始终保持不变。在定义和使用常量的时候不需要使用 `$` 符号。 

**构造函数**

void __constuct()

创建一个对象时（ `new` 操作），构造函数会自动调用。

如果子类中定义了构造函数则不会隐式调用其父类的构造函数。要执行父类的构造函数，需要在子类的构造函数中调用parent::__construct()。

```php
<?php
class A 
{
   public function __construct() 
   {
       echo "A";
   }
}

class B extends A 
{
   public function __construct() 
   {
       parent::__construct();
       echo "B";
   }
}

new A(); // A
new B(); // AB
```

**析构函数**

void __destruct( void )

析构函数会在到某个对象的所有引用都被删除或者当对象被显式销毁时执行。

和构造函数一样，父类的析构函数不会被引擎暗中调用。要执行父类的析构函数，必须在子类的析构函数体中显式调用 parent::__destruct()。 

析构函数即使在使用 `exit()` 终止脚本运行时也会被调用。在析构函数中 调用 `exit()` 将会中止其余关闭操作的运行。 

```php
<?php
class A {
   public function __construct() {
       echo 'Start...';
   } 

   public function sayHi()
   {
       echo "Hi...";    
   }

   public function __destruct() {
       echo "Finish";
   }
}

(new A())->sayHi(); // Start...Hi...Finish
```

