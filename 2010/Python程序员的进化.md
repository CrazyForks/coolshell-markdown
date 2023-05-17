# Python程序员的进化
>date: 2010-02-01T17:42:48+08:00


以前本站发布过一篇《[程序员的进化](/2009/%E5%B9%BD%E9%BB%98%EF%BC%9A%E7%A8%8B%E5%BA%8F%E5%91%98%E7%9A%84%E8%BF%9B%E5%8C%96.md)》，以一种幽默的代码展现方式调侃了程序。下面这篇是关于Python程序员的。以阶乘为例，很有意思。




目录



* [新手程序员](#%E6%96%B0%E6%89%8B%E7%A8%8B%E5%BA%8F%E5%91%98 "新手程序员")
* [第一年的刚学完Pascal的新手](#%E7%AC%AC%E4%B8%80%E5%B9%B4%E7%9A%84%E5%88%9A%E5%AD%A6%E5%AE%8CPascal%E7%9A%84%E6%96%B0%E6%89%8B "
第一年的刚学完Pascal的新手")
* [第一年的刚学完C语言的新手](#%E7%AC%AC%E4%B8%80%E5%B9%B4%E7%9A%84%E5%88%9A%E5%AD%A6%E5%AE%8CC%E8%AF%AD%E8%A8%80%E7%9A%84%E6%96%B0%E6%89%8B "
第一年的刚学完C语言的新手")
* [第一年刚学完SICP的新手](#%E7%AC%AC%E4%B8%80%E5%B9%B4%E5%88%9A%E5%AD%A6%E5%AE%8CSICP%E7%9A%84%E6%96%B0%E6%89%8B "
第一年刚学完SICP的新手")
* [第一年刚学完Python的新手](#%E7%AC%AC%E4%B8%80%E5%B9%B4%E5%88%9A%E5%AD%A6%E5%AE%8CPython%E7%9A%84%E6%96%B0%E6%89%8B "
第一年刚学完Python的新手")
* [爱偷懒的程序员](#%E7%88%B1%E5%81%B7%E6%87%92%E7%9A%84%E7%A8%8B%E5%BA%8F%E5%91%98 "
爱偷懒的程序员")
* [更懒的 Python 程序员](#%E6%9B%B4%E6%87%92%E7%9A%84_Python_%E7%A8%8B%E5%BA%8F%E5%91%98 "
更懒的 Python 程序员")
* [Python 专家](#Python_%E4%B8%93%E5%AE%B6 "
Python 专家")
* [Python 黑客](#Python_%E9%BB%91%E5%AE%A2 "Python 黑客")
* [专家级程序员](#%E4%B8%93%E5%AE%B6%E7%BA%A7%E7%A8%8B%E5%BA%8F%E5%91%98 "
专家级程序员")
* [英语系的专家级程序员](#%E8%8B%B1%E8%AF%AD%E7%B3%BB%E7%9A%84%E4%B8%93%E5%AE%B6%E7%BA%A7%E7%A8%8B%E5%BA%8F%E5%91%98 "
英语系的专家级程序员")
* [Web 设计者](#Web_%E8%AE%BE%E8%AE%A1%E8%80%85 "
Web 设计者")
* [Unix 程序员](#Unix_%E7%A8%8B%E5%BA%8F%E5%91%98 "
Unix 程序员")
* [Windows 程序员](#Windows_%E7%A8%8B%E5%BA%8F%E5%91%98 "
Windows 程序员")
* [公司里的程序员](#%E5%85%AC%E5%8F%B8%E9%87%8C%E7%9A%84%E7%A8%8B%E5%BA%8F%E5%91%98 "
公司里的程序员")

#### 新手程序员



```
def factorial(x):
    if x == 0:
        return 1
    else:
        return x * factorial(x - 1)
print factorial(6)
```

#### 
第一年的刚学完Pascal的新手



```
def factorial(x):
    result = 1
    i = 2
    while i <= x:
        result = result * i
        i = i + 1
    return result
print factorial(6)
```

#### 


#### 
第一年的刚学完C语言的新手



```
def fact(x): #{
    result = i = 1;
    while (i <= x): #{
        result *= i;
        i += 1;
    #}
    return result;
#}
print(fact(6))
```

#### 
第一年刚学完SICP的新手



```
@tailcall
def fact(x, acc=1):
    if (x > 1): return (fact((x - 1), (acc * x)))
    else:       return acc
print(fact(6))
```

#### 
第一年刚学完Python的新手



```
def Factorial(x):
    res = 1
    for i in xrange(2, x + 1):
        res *= i
    return res
print Factorial(6)
```

#### 
爱偷懒的程序员



```
def fact(x):
    return x > 1 and x * fact(x - 1) or 1
print fact(6)
```

#### 
更懒的 Python 程序员



```
f = lambda x: x and x * f(x - 1) or 1
print f(6)
```

#### 
Python 专家



```
import operator as op
import functional as f
fact = lambda x: f.foldl(op.mul, 1, xrange(2, x + 1))
print fact(6)
```

#### Python 黑客



```
import sys
@tailcall
def fact(x, acc=1):
    if x: return fact(x.__sub__(1), acc.__mul__(x))
    return acc
sys.stdout.write(str(fact(6)) + '\n')
```

#### 
专家级程序员



```
import c_math
fact = c_math.fact
print fact(6)
```

#### 
英语系的专家级程序员



```
import c_maths
fact = c_maths.fact
print fact(6)
```

#### 
Web 设计者



```
def factorial(x):
    #-------------------------------------------------
    #--- Code snippet from The Math Vault          ---
    #--- Calculate factorial (C) Arthur Smith 1999 ---
    #-------------------------------------------------
    result = str(1)
    i = 1 #Thanks Adam
    while i <= x:
        #result = result * i  #It's faster to use *=
        #result = str(result * result + i)
           #result = int(result *= i) #??????
        result str(int(result) * i)
        #result = int(str(result) * i)
        i = i + 1
    return result
print factorial(6)
```

#### 
Unix 程序员



```
import os
def fact(x):
    os.system('factorial ' + str(x))
fact(6)
```

#### 
Windows 程序员



```
NULL = None
def CalculateAndPrintFactorialEx(dwNumber,
                                 hOutputDevice,
                                 lpLparam,
                                 lpWparam,
                                 lpsscSecurity,
                                 *dwReserved):
    if lpsscSecurity != NULL:
        return NULL #Not implemented
    dwResult = dwCounter = 1
    while dwCounter <= dwNumber:
        dwResult *= dwCounter
        dwCounter += 1
    hOutputDevice.write(str(dwResult))
    hOutputDevice.write('\n')
    return 1
import sys
CalculateAndPrintFactorialEx(6, sys.stdout, NULL, NULL, NULL, NULL, NULL, NULL, NULL, NULL, NULL, NULL, NULL, NULL, NULL, NULL, NULL, NULL)
```

#### 
公司里的程序员



```
def new(cls, *args, **kwargs):
    return cls(*args, **kwargs)

class Number(object):
    pass

class IntegralNumber(int, Number):
    def toInt(self):
        return new (int, self)

class InternalBase(object):
    def __init__(self, base):
        self.base = base.toInt()

    def getBase(self):
        return new (IntegralNumber, self.base)

class MathematicsSystem(object):
    def __init__(self, ibase):
        Abstract

    @classmethod
    def getInstance(cls, ibase):
        try:
            cls.__instance
        except AttributeError:
            cls.__instance = new (cls, ibase)
        return cls.__instance

class StandardMathematicsSystem(MathematicsSystem):
    def __init__(self, ibase):
        if ibase.getBase() != new (IntegralNumber, 2):
            raise NotImplementedError
        self.base = ibase.getBase()

    def calculateFactorial(self, target):
        result = new (IntegralNumber, 1)
        i = new (IntegralNumber, 2)
        while i <= target:
            result = result * i
            i = i + new (IntegralNumber, 1)
        return result

print StandardMathematicsSystem.getInstance(new (InternalBase, new (IntegralNumber, 2))).calculateFactorial(new (IntegralNumber, 6))
```

摘自：[来源](https://gist.github.com/289467)


