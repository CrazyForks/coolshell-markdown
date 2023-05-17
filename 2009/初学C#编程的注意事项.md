# 初学C#编程的注意事项
>date: 2009-08-31T12:15:05+08:00


下面是8个C#编程时的注意事项是给初学者的，可能你知道，也可能你不知道，不过这些都是一些可能会让人疏忽的地方，还是要注意一下。


**1.使用String变量:**


考虑有下面的一个程序想判断一下字符串是否有内容。



```
if (someString.Length > 0)
{
    // …
}

```

但是，这个字符串对象很可能是个空对象，所以，最好先判断一下null



```
if  (!String.IsNullOrEmpty(someString))
{
    // 是不是更好一些？
}

```


**2.字符器连接**



```
string s = “dev”;
s += “-”;
s += “the”;
s += “-”;
s += “web”;
s += “.”;
s += “com”;

```

这样做没什么问题，只不过性能不高，因为+=操作符其实调用的是String类的Append访问，所以，+=会有两次函数调用，下面的则会好一些。



```
StringBuilder s = new StringBuilder();
s.Append(”dev”);
s.Append(”-”);
s.Append(”the”);
s.Append(”-”);
s.Append(”web”);
s.Append(”.”);
s.Append(”com”);

```

**3.使用Console**



```
Console.WriteLine("A= " + 1 + " B=" + 2 + " C= " + someValue);

```

和第二点说的一样，这并没有效率，使用下面的语句，会更有效率。



```
Console.WriteLine(”A: {0}\nB: {1}\nC: {2}”, 1, 2, someValue);

```

**4.字符串转整型**



```
int i = int.Parse(Request.QueryString["id"]);

```

这样做的问题是，如果有人这样请求你的页面：yourpage.aspx?id=A6，那么A6将会导致你的程序抛出一个异常。因为A6不是一个整数字符串。使用TryParse会好一点。



```
int i;
if (!int.TryParse(Request.QueryString["id"] , out i))
{
    //…
}

```

**5. 调用IDbConnection 的 Close 方法**



```
IDbConnection dbConn = null;

try
{
    dbConn = new SqlConnection(”some Connection String”);
    dbConn.Open();
}
finally
{
    dbConn.Close();
}

```

调用SqlConnection的构造函数可能会出现一个异常，如果是这样的话，我们还需要调用Close方法吗？



```
IDbConnection dbConn = null;

try
{
    dbConn = new SqlConnection(”Some Connection String”);
    dbConn.Open();
}
finally
{
    if (dbConn != null)
    {
        dbConn.Close();
    }
}

```

**6.使用List类**



```
public void SomeMethod(List<SomeItem> items)
{
    foreach(var item in items)
    {
        // do something with the item…
    }
}

```

如果我们只是遍历List容器中的所有内容的话，那么，使用IEnumerable接口会更好一些。因为函数参数传递一个List对象要比一个IEnumerable接口要花费更多的开销。



```
public void SomeMethod(IEnumerable<SomeItem> items)
{
    foreach(var item in items)
    {
        // do something with the item…
    }
}

```

**7.直接使用数字**



```
if(mode == 1) { … }
else if(mode == 2) { … }
else if(mode == 3) { … }

```

为什么不给你的这些数字取个名字呢？比如使用Enumerations。



```
public enum SomeEnumerator
{
    DefaultMode = 1,
    SafeMode = 2,
    NormalMode = 3
}

if(mode == SomeEnumerator.DefaultMode) { … }
else if(mode == SomeEnumerator.SafeMode) { … }
else if(mode == SomeEnumerator.NormalMode) { … }

```

**8.字符串替换**



```
string s = "www.coolshell.cn is a amazing site";
s.Replace("amazing", "awful");

```

字符串s的内容什么也不会改变，因为string返回的是替换过的字串。这点很多初学者经常忘了。下面就没有问题了。



```
s = s.Replace("amazing", "awful");

```

文章：[来源](http://dev-the-web.com/blog/2009/08/27/top-csharp-programming-mistakes/)


