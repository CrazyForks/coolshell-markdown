# C++的字符串格式化库
>date: 2010-11-02T08:59:06+08:00


这里向大家介绍一个C++的字符串格式化库，叫cpptempl，这个库支持对字符串格式的条件，循环，变量插入。看上去很不错，只不过其是基于boost库的。


下面是一个例子：



```
// The text template
wstring text = L"I heart {$place}!" ;
// Data to feed the template engine
cpptempl::data_map data ;
// {$place} => Okinawa
data[L"place"] = cpptempl::make_data(L"Okinawa");
// parse the template with the supplied data dictionary
wstring result = cpptempl::parse(text, data) ;
```

输出结果是：



> I heart Okinawa!
> 
> 


是不是很方便？让我们看一个更复杂的例子：




```
// You'd probably load this template from a file in real life.
wstring text = L"<h3>Locations</h3>\n<ul>\n"
    L"{% for place in places %}"
    L"<li>{$place}</li>\n"
    L"{% endfor %}"
    L"</ul>" ;
// Create the list of items
cpptempl::data_list places;
places.push_back(cpptempl::make_data(L"Okinawa"));
places.push_back(cpptempl::make_data(L"San Francisco"));
// Now set this in the data map
cpptempl::data_map data ;
data[L"places"] = cpptempl::make_data(places);
// parse the template with the supplied data dictionary
wstring result = cpptempl::parse(text, data) ;
```

输出结果是：



> <h3>Locations</h3>  
> 
> <ul>  
> 
> <li>Okinawa</li>  
> 
> <li>San Francisco</li>  
> 
> </ul>
> 
> 


更为详细的说明请到这里：[http://bitbucket.org/ginstrom/cpptemplate/wiki/Home](https://bitbucket.org/ginstrom/cpptemplate/wiki/Home)。


Google也有一个类似的库叫ctemplate：[http://code.google.com/p/google-ctemplate/](https://code.google.com/p/google-ctemplate/) 提供相似的方法，你也可以试试看。与Google相对应的Java库叫Hapax：[http://code.google.com/p/hapax/](https://code.google.com/p/hapax/)。


