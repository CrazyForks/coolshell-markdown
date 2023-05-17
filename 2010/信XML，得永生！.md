# 信XML，得永生！
>date: 2010-06-09T08:27:42+08:00


在计算机的世界里，什么最牛？[Javascript](https://coolshell.cn/?tag=javascript)？[C语言](https://coolshell.cn/articles/914.html)？[C++](https://coolshell.cn/articles/1724.html)？[iPad](https://coolshell.cn/articles/2086.html)？还是[brainfuck](https://coolshell.cn/articles/1142.html)？我个人觉得都不是，这个世界里，XML最NB，这世界到处都充斥着XML，正如在“[十条不错的编程观点](https://coolshell.cn/articles/2424.html)”文中所说，我们不用XML我们都不知道怎么编程了。下面，让我们来看一看XML的几个真实的示例，相信你会同意我的观点的。




目录



* [一、如何用XML返回数据库SQL查询结果](#%E4%B8%80%E3%80%81%E5%A6%82%E4%BD%95%E7%94%A8XML%E8%BF%94%E5%9B%9E%E6%95%B0%E6%8D%AE%E5%BA%93SQL%E6%9F%A5%E8%AF%A2%E7%BB%93%E6%9E%9C "一、如何用XML返回数据库SQL查询结果")
* [二、如何用XML序列化一个图片](#%E4%BA%8C%E3%80%81%E5%A6%82%E4%BD%95%E7%94%A8XML%E5%BA%8F%E5%88%97%E5%8C%96%E4%B8%80%E4%B8%AA%E5%9B%BE%E7%89%87 "二、如何用XML序列化一个图片")
* [三、如何让XML与CSV格式兼容](#%E4%B8%89%E3%80%81%E5%A6%82%E4%BD%95%E8%AE%A9XML%E4%B8%8ECSV%E6%A0%BC%E5%BC%8F%E5%85%BC%E5%AE%B9 "三、如何让XML与CSV格式兼容")
* [四、如何把XML当成数组来用](#%E5%9B%9B%E3%80%81%E5%A6%82%E4%BD%95%E6%8A%8AXML%E5%BD%93%E6%88%90%E6%95%B0%E7%BB%84%E6%9D%A5%E7%94%A8 "四、如何把XML当成数组来用")

#### 一、如何用XML返回数据库SQL查询结果



```
<?xml version="1.0" encoding="iso-8859-1" ?>
<result>
  <fields>
    <field>NAME</field>
    <field>LAST NAME</field>
    <field>MOTHER MAIDEN NAME</field>
    <field>BIRTHDATE</field>
    ...
  </fields>
  <data>
    <row>
      <value>MARLENE</value>
      <value>RUTH</value>
      <value>DE MARCO</value>
      <value>1973-02-24 00:00:00</value>
      ...
    </row>
  </data>
</result>
```


#### 二、如何用XML序列化一个图片



```
<attachments xmlns = "http://webservices..." >
  <bytes>37</bytes>
  <bytes>80</bytes>
  <bytes>68</bytes>
  <bytes>70</bytes>
  <bytes>45</bytes>
  <bytes>49</bytes>
  <bytes>46</bytes>
  <bytes>52</bytes>
  <bytes>10</bytes>
  <bytes>37</bytes>
  <bytes>-30</bytes>
  <bytes>-29</bytes>
  <bytes>-49</bytes>
  <bytes>-45</bytes>
  <bytes>10</bytes>
  <bytes>52</bytes>
  <bytes>32</bytes>
  <bytes>48</bytes>
  <bytes>32</bytes>
  <bytes>111</bytes>
  ...
  ...
  ...
```

#### 三、如何让XML与CSV格式兼容



```
<?xml version="1.0" encoding="iso8859-1" ?>
<import tag="1stTEST" type="data" mode="update">
<options>
    <dateformat mmddyyyy="true"/>
        <notification>
            <EMail>[[email protected]](/cdn-cgi/l/email-protection)</EMail>
        </notification>
    </options>
    <fields>
        <field name="name" type="char" mapsto="person.data"/>
        <field name="officeid" type="char" mapsto="custom.locationid"/>
        <field name="startyear" type="char" mapsto="person.yearstarted"/>
        <field name="personelid" type="int" mapsto="person.id"/>
        <field name="dob" type="date" mapsto="person.dateofbith"/>
        <field name="sex" type="char" mapsto="person.sex"/>
        <field name="modified" type="date" mapsto="record.modified"/>
    </fields>
    <csvdata columnheaders="false">
<![CDATA[
"Jack Wade",214,2002,111012,07/04/1975,"M",02/11/2006
"Sam Davidson",214,1999,104841,10/15/1967,"M",02/10/2006
"Denise V Law",214,1998,104660,01/21/1971,"F",02/17/2006
"Lisa Blake",214,1989,100987,08/01/1982,"F",01/21/2006
"Andrew Match",214,1991,101074,12/25/1980,"M",02/28/2006
]]>
    </csvdata>
</import>
```

#### 四、如何把XML当成数组来用



```
<rootNode>
   <numberOfAddresses>110</numberOfAddresses>
   <address_1>442 Fake St.</address_1>
   <address_2>61 Main St.</address_2>
   ...
   ...
   ...
   <address_110>3881 N 4th Ave. #5D</address_110>
</rootNode>
```

相信你一定有比这更牛X的例子，欢迎与我们分享！


