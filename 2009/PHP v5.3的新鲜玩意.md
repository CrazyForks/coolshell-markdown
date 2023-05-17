# PHP v5.3的新鲜玩意
>date: 2009-03-02T13:40:50+08:00


PHP v5.3马上就要release了，这里让我们看看他有一些什么样的新特性。


**1）\_callStatic() magic 方法**



```
class Foo
{
    public static function __callStatic( $name, $args )
    {
        echo "Called method $name statically";
    } 

    public function __call( $name, $args )
    {
        echo "Called method $name";
    }
}
```



```
Foo::dog();       // outputs "Called method dog statically"
$foo = new Foo;
$foo-&gt;dog();      // outputs "Called method dog"
```

**2）`动态调用函数`**



```
class Dog
{
    public function bark()
    {
        echo "Woof!";
    }
<span style="color: #333399;">} 

$class = "Dog"
$action = "bark";
$x = new $class(); // instantiates the class "Dog"
$x-&gt;$action();     // outputs "Woof!" </span>
```

**3) 标准****PHP****库（SPL）**


加了了少数几个容器类，比如，栈（SplStack）和固定数组（SplFixedArray）



```
$stack = new SplStack(); 

// push a few new items on the stack
$stack-&gt;push("a");
$stack-&gt;push("b");
$stack-&gt;push("c"); 

// see how many items are on the stack
echo count($stack); // returns 3 

// iterate over the items in the stack
foreach ( $stack as $item )
    echo "[$item],";
// the above outputs: [c][/c]

 [/c],[b],[a]  // pop an item off the stack echo $stack-&gt;pop(); // returns "c"   // now see how many items are on the stack echo count($stack); // returns 2
```

**4) Closures 功能**


关于Closures，这是一个把函数定义成变量的玩意。让我们看几个例子：


示例一：



```
$string = "Hello World!";
$closure = function() use ($string) { echo $string; };

$closure();
```

**Output:**  

Hello World!  

示例二 使用引用的变量



```
$x = 1
$closure = function() use (&amp;$x) { ++$x; }

echo $x . "\\n";
$closure();
echo $x . "\\n";
$closure();
echo $x . "\\n";
```

**Output**:  

1  

2  

3  

示例三，返回值



```
function getAppender($baseString)
{
      return function($appendString) use ($baseString)
  { return $baseString .$appendString; };
}
```

示例四，Reflection



```
class Counter
{
      private $x;

      public function __construct()
      {
           $this-&gt;x = 0;
      }

      public function increment()
      {
           $this-&gt;x++;
      }

      public function currentValue()
      {
           echo $this-&gt;x . "\\n";
      }
}
$class = new ReflectionClass("Counter");
$method = $class-&gt;getMethod("currentValue");
$closure = $method-&gt;getClosure()
$closure();
$class-&gt;increment();
$closure();
```

**Output**:  

0  

1  

示例五，Reflection API



```
$closure = function ($x, $y = 1) {};
$m = new ReflectionMethod($closure);
Reflection::export ($m);
<strong>Output</strong>:
Method [  public method __invoke ] {
  - Parameters [2] {
    Parameter #0 [  $x ]
    Parameter #1 [  $y ]
  }
}
```

示例六，Uses Case



```
$logdb = function ($string) { Logger::log("debug","database",$string);};
$db = mysqli_connect("server","user","pass");
$logdb("Connected to database");
$db-&gt;query("insert into parts (part, description) values
 ("Hammer","Pounds nails");
$logdb("Insert Hammer into to parts table");
$db-&gt;query("insert into parts (part, description) values
       ("Drill","Puts holes in wood");
$logdb("Insert Drill into to parts table");
$db-&gt;query("insert into parts (part, description) values
 ("Saw","Cuts wood");
$logdb("Insert Saw into to parts table");
```

更为详细的文章，请参考这里，[链接](http://www.ibm.com/developerworks/opensource/library/os-php-5.3new2/index.html)。


**5) 使用namespace**


新版的PHP会开始支持C++式的namespace，请参看示例：


示例一



```
/* Foo.php */
&lt;?php
namespace Foo;
function bar()
{
    echo "calling bar....";
}
?&gt; 

/* File1.php */
&lt;?php
include "./Foo.php";
Foo/bar(); // outputs "calling bar....";
?&gt; 

/* File2.php */
&lt;?php
include "./Foo.php";
use Foo as ns;
ns/bar(); // outputs "calling bar....";
?&gt; 

/* File3.php */
&lt;?php
include "./Foo.php";
use Foo;
bar(); // outputs "calling bar....";
?&gt;
<!--p include "./Foo.php"; use Foo; bar(); // outputs "calling bar....";-->
```

示例二，多重namespace



```
<!--p namespace Foo; class Test {}  namespace Bar; class Test {}  $a = new Foo\\Test; $b = new Bar\\Test;  var_dump($a, $b);--> &lt;?php
namespace Foo;
class Test {} 

namespace Bar;
class Test {} 

$a = new Foo\\Test;
$b = new Bar\\Test; 

var_dump($a, $b); 

Output:
object(Foo\\Test)#1 (0) {
}
object(Bar\\Test)#2 (0) {
}
<strong>Output:</strong>
object(Foo\\Test)#1 (0) { }
object(Bar\\Test)#2 (0) { }
```

示例三，不同文件中的namespace



```
/*定义*/
/* global.php */
&lt;?php
function hello()
{
    echo "hello from the global scope!";
}
?&gt; 

/* Foo.php */
&lt;?php
namespace Foo;
function hello()
{
    echo "hello from the Foo namespace!";
}
?&gt; 

/* Foo_Bar.php */
&lt;?php
namespace Foo/Bar;
function hello()
{
    echo "hello from the Foo/Bar namespace!";
}
?&gt;
<!--p namespace Foo/Bar; function hello() {     echo "hello from the Foo/Bar namespace!"; }-->

/*使用 */
<!--p include "./global.php"; include "./Foo.php"; include "./Foo_Bar.php"; use Foo;  hello();         // outputs "hello from the Foo namespace!" Bar\\hello();   // outputs "hello from the Foo/Bar namespace!" \\hello();       // outputs "hello from the global scope!"-->&lt;?php
include "./global.php";
include "./Foo.php";
include "./Foo_Bar.php";

use Foo; 

hello();         // outputs "hello from the Foo namespace!"
Bar\\hello();   // outputs "hello from the Foo/Bar namespace!"
\\hello();       // outputs "hello from the global scope!"
?&gt;
```

更为详细的文章，请参考这里，[链接](http://www.ibm.com/developerworks/opensource/library/os-php-5.3new3/index.html)。


**6)开始支持Achieve包**


正像JAR一样，PHP也要开始支持自己的Achieve包了，叫作，Phar。PHP提供了一整套函数来帮助开发人员创建和使用Phar，正如下面的示例所示：


**创建**：



```
$p = new Phar("/path/to/my.phar",
 CURRENT_AS_FILEINFO | KEY_AS_FILENAME, "my.phar");
$p-&gt;startBuffering();
```

**创建文件存根**（stub）


`$p-&gt;setStub("<!--p Phar::mapPhar(); include "phar://myphar.phar/index.php"; __HALT_COMPILER();-->");`


**加入文件**：



```
$p["file.txt"] = "This is a text file";
$p["index.php"] = file_get_contents("index.php");
$p["big.txt"] = "This is a big text file";
$p["big.txt"]-&gt;setCompressedBZIP2();
//加入某目录下所有的文件
$p-&gt;buildFromDirectory("/path/to/files","./\\.php$/");
```

**使用Phar**



```
include "myphar.phar";
include "phar://myphar.phar/file.php";
```

更为详细的文章，请参考这里，[链接](http://www.ibm.com/developerworks/opensource/library/os-php-5.3new4/index.html)。


