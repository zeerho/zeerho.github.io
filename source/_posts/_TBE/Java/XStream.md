---
title: XStream
date: 2017-01-01 09:00:00
tags: [Java]
---

# 常用注解
|注解|作用|作用目标
|:--|:--:|:--:|
|`@XStreamAlias("label_name")`|别名注解|作用目标: 类,字段  
|`@XStreamImplicit`|隐式集合|集合字段  
|`@XStreamImplicit(itemFieldName="item_field_name")`|隐式集合，|并指定集合内标签名|集合字段  
|`@XStreamConverter(SingleValueCalendarConverter.class)`|注入转换器|对象  
|`@XStreamAsAttribute`|设为该标签的属性|字段  
|`@XStreamOmitField`|忽略字段|字段
|`xstream.autodetectAnnotations(true);`|自动侦查注解  
*自动侦查注解与 `XStream.processAnnotations(Class[] cls)` 的区别在于性能.自动侦查注解将缓存所有类的类型.*  

## 简单示例
###1.实体类
以下所有实体类省略了 getter 和 setter 方法。
```java
package com.demo.entity;

import com.thoughtworks.xstream.annotations.XStreamAlias;

@XStreamAlias("Student")
public class Student {
	@XStreamAlias("Name")
	private String name;
	@XStreamAlias("Subjects")
	private Subjects subjects;
	@XStreamAlias("Pets")
	private Pets pets;
}
```
某个标签下包含多个同名标签 ↓
```java
package com.demo.entity;

import java.util.List;
import com.thoughtworks.xstream.annotations.XStreamImplicit;

public class Subjects {
	@XStreamImplicit(itemFieldName="Subject")//itemFieldName定义重复字段的名称
/*  <Subjects>					                            <Subjects>
	    <subjectList class="java.util.Arrays$ArrayList">        <Subject>语文</Subject>
			<a class="string-array">        加上该注解=>		    <Subject>数学</Subject>
			    <string>语文</string>                       </Subjects>
		    	<string>数学</string>
			</a>
		</subjectList>
	</Subjects>*/
	private List<String> subjectList;
}
```
某一标签下包含多个其他对象的标签 ↓
```java
package com.demo.entity;

import java.util.List;
import com.thoughtworks.xstream.annotations.XStreamAlias;
import com.thoughtworks.xstream.annotations.XStreamImplicit;

public class Pets {
	@XStreamImplicit(itemFieldName="pet")
	private List<Animal> animalList;
}
```
被包含的对象的标签 ↓
```java
package com.demo.entity;

import com.thoughtworks.xstream.annotations.XStreamAlias;

public class Animal {
	@XStreamAlias("Specie")
	private String specie;
	@XStreamAlias("Age")
	private String age;
	
	public Animal(String specie,String age){
		this.specie=specie;
		this.age=age;
	}
}
```
###2.测试类
bean->xml ↓
```java
package com.demo.test;

import java.util.ArrayList;
import java.util.Arrays;
import java.util.List;

import com.demo.entity.*;
import com.thoughtworks.xstream.XStream;
import com.thoughtworks.xstream.io.xml.DomDriver;

public class TestToXML {	
	public static void main(String[] args) {
        /*省略实例化Student stu及其子对象的过程*/

        /*1.只使用.toXML()而不使用.fromXML()时也可不传入解析器DomDriver，即XStream xstream = new XStream();
        2.使用.toXML()方法也可将未使用XStream注解的实体类转为XML*/
		XStream xstream = new XStream(new DomDriver());
		xstream.processAnnotations(Student.class);
		xstream.toXML(stu,System.out);
	}
}
```
测试结果 ↓
```xml
<Student>
    <Name>Tom</Name>
    <Subjects>
        <Subject>语文</Subject>
        <Subject>数学</Subject>
    </Subjects>
    <Pets>
        <pet>
            <Specie>狗</Specie>
            <Age>2</Age>
        </pet>
        <pet>
            <Specie>猫</Specie>
            <Age>3</Age>
        </pet>
    </Pets>
</Student>
```
xml->bean
```java
package com.demo.test;

import com.demo.entity.*;
import com.thoughtworks.xstream.XStream;
import com.thoughtworks.xstream.io.xml.DomDriver;

public class TestFromXML {	
	public static void main(String[] args) {
        /*省略创建String xmlstr的过程*/

        /*此处必须传入DomDriver作为参数，否则报错*/
		XStream xstream = new XStream(new DomDriver());
		xstream.processAnnotations(Student.class);
		Student stu = (Student) xstream.fromXML(xmlstr);
	}
}
```

# 使用 Java 语句实现 bean 字段与 xml 标签的映射

```java
public static void main(String[] args){
    XStream xstream = new XStream(new DomDriver());
    xstream.alias("Student",Student.class);//@XStreamAlias("Student")
    xstream.setMode(XStream.NO_REFERENCES);//详见下文
    xstream.addImplicitCollection(Subjects.class,"SubjectList");//对应Subjects类中，private List<String> subjectList;上的@XStreamImplicit
    xstream.useAttributeFor(Student.class,"name");//对应@XStreamAsAttribute，以上示例中未实现
}
```

# 处理对象引用的不同方式

通过 `xstream.setMode(XStream.NO_REFERENCES);` 来设置处理对象引用的方式。其中，`xstream` 为一个 XStream 实例；`NO_REFERENCES` 为 static final 形式，所以直接通过 XStream 类引用。
所有的模式如下：
```java
public static final int NO_REFERENCES = 1001;//0x3e9
public static final int ID_REFERENCES = 1002;//0x3ea
public static final int XPATH_RELATIVE_REFERENCES = 1003;//0x3eb
public static final int XPATH_ABSOLUTE_REFERENCES = 1004;//0x3ec
public static final int SINGLE_NODE_XPATH_RELATIVE_REFERENCES = 1005;//0x3ed
public static final int SINGLE_NODE_XPATH_ABSOLUTE_REFERENCES = 1006;//0x3ee
```
其中，`XPATH_RELATIVE_REFERENCES`为缺省值。
如下创建对象来测试六种方式的区别。
```java
Object ob = new Object();
List<Object> li = new ArrayList<Object>();
li.add(ob);
li.add(ob);//在List中重复添加某一对象的引用
li.add(li);//在List中添加List自身的引用
XStream xstream = new XStream(new DomDriver());
```
编写测试代码
```java
//1.XPATH_RELATIVE_REFERENCES（默认）
System.out.println("1.默认XPATH_RELATIVE_REFERENCES");
xstream.toXML(li, System.out);
System.out.println("");
//2.XPATH_ABSOLUTE_REFERENCES
System.out.println("2.XPATH_ABSOLUTE_REFERENCES");
xstream.setMode(XStream.XPATH_ABSOLUTE_REFERENCES);
xstream.toXML(li, System.out);
System.out.println("");
//3.ID_REFERENCES
System.out.println("3.ID_REFERENCES");
xstream.setMode(XStream.ID_REFERENCES);
xstream.toXML(li, System.out);
System.out.println("");
//4.NO_REFERENCES(此处会报错，须将其注释来完成后面两种方式的测试)
System.out.println("4.NO_REFERENCES");
xstream.setMode(XStream.NO_REFERENCES);
xstream.toXML(li, System.out);
System.out.println("");
//5.SINGLE_NODE_XPATH_RELATIVE_REFERENCES
System.out.println("5.SINGLE_NODE_XPATH_RELATIVE_REFERENCES");
xstream.setMode(XStream.SINGLE_NODE_XPATH_RELATIVE_REFERENCES);
xstream.toXML(li, System.out);
System.out.println("");
//6.SINGLE_NODE_XPATH_ABSOLUTE_REFERENCES
System.out.println("6.SINGLE_NODE_XPATH_ABSOLUTE_REFERENCES");
xstream.setMode(XStream.SINGLE_NODE_XPATH_ABSOLUTE_REFERENCES);
xstream.toXML(li, System.out);
System.out.println("");
```
测试结果
```xml
1.默认XPATH_RELATIVE_REFERENCES
<list>
  <object/>
  <object reference="../object"/>
  <list reference=".."/>
</list>
2.XPATH_ABSOLUTE_REFERENCES
<list>
  <object/>
  <object reference="/list/object"/>
  <list reference="/list"/>
</list>
3.ID_REFERENCES
<list id="1">
  <object id="2"/>
  <object reference="2"/>
  <list reference="1"/>
</list>
5.SINGLE_NODE_XPATH_RELATIVE_REFERENCES
<list>
  <object/>
  <object reference="../object[1]"/>
  <list reference=".."/>
</list>
6.SINGLE_NODE_XPATH_ABSOLUTE_REFERENCES
<list>
  <object/>
  <object reference="/list[1]/object[1]"/>
  <list reference="/list[1]"/>
</list>
```
4.NO_REFERENCES的报错信息
```
4.NO_REFERENCES
<list>
  <object/>
  <object/>
  <listException in thread "main" com.thoughtworks.xstream.core.TreeMarshaller$CircularReferenceException: Recursive reference to parent object
---- Debugging information ----
item-type           : java.util.ArrayList
converter-type      : com.thoughtworks.xstream.converters.collections.CollectionConverter
-------------------------------
	at com.thoughtworks.xstream.core.TreeMarshaller.convert(TreeMarshaller.java:63)
	at com.thoughtworks.xstream.core.TreeMarshaller.convertAnother(TreeMarshaller.java:58)
	at com.thoughtworks.xstream.core.TreeMarshaller.convertAnother(TreeMarshaller.java:43)
	at com.thoughtworks.xstream.converters.collections.AbstractCollectionConverter.writeItem(AbstractCollectionConverter.java:64)
	at com.thoughtworks.xstream.converters.collections.CollectionConverter.marshal(CollectionConverter.java:55)
	at com.thoughtworks.xstream.core.TreeMarshaller.convert(TreeMarshaller.java:70)
	at com.thoughtworks.xstream.core.TreeMarshaller.convertAnother(TreeMarshaller.java:58)
	at com.thoughtworks.xstream.core.TreeMarshaller.convertAnother(TreeMarshaller.java:43)
	at com.thoughtworks.xstream.core.TreeMarshaller.start(TreeMarshaller.java:82)
	at com.thoughtworks.xstream.core.AbstractTreeMarshallingStrategy.marshal(AbstractTreeMarshallingStrategy.java:37)
	at com.thoughtworks.xstream.XStream.marshal(XStream.java:895)
	at com.thoughtworks.xstream.XStream.marshal(XStream.java:884)
	at com.thoughtworks.xstream.XStream.toXML(XStream.java:872)
	at com.demo.test.TestXStream.main(TestXStream.java:58)
```
