---
layout: post
title:  "XML和Java对象之间的换转利器--XStream的使用（进阶版）"
keywords: "java xstream"
description: "这一篇主要写XStream的进阶应用，注解、转换器的使用，以及XML和Json之间的转换。"
category: java
tags: jvm-serializers xstream
---

###XStream注解的使用步骤

上一篇中使用编码方式设置别名，这种方式显得比较繁琐，

```java
	xstream.alias("teacher", Teacher.class);
	xstream.alias("note", Note.class);
	xstream.aliasField("teacherName", Teacher.class, "name");
	xstream.addImplicitCollection(Teacher.class, "notes");
	xstream.useAttributeFor(Teacher.class, "teacherName");
	xstream.aliasPackage("com.zeusjava.entity", "com.zeusjava.test");
```

下面通过java注解实现以上的功能

```java
@XStreamAlias("teacher")   //定义类级别别名
class Teacher {

   @XStreamAlias("name")   //定义字段级别别名
   @XStreamAsAttribute     //把这个字段定义为对象的属性
   private String teacherName;
   
   @XStreamImplicit        //定义list为一个隐式的集合
   private List<Note> notes = new ArrayList<Note>();
   
   @XStreamOmitField       //在XML中忽略这个属性
   private int type;
}
```
Junit测试代码

```java
 @Test
	 public void testAnnotation(){
		 Teacher t1=getTeacherDetail();
		 xstream.processAnnotations(Teacher.class);
		 String xml=xstream.toXML(t1);
		 System.out.println(formatXml(xml));
	 }
```

执行的结果为

	<?xml version="1.0" encoding="UTF-8"?><teacher name="张三">
	  <com.zeusjava.entity.Note>
	    <title>笔记1</title>
	    <description>数学</description>
	  </com.zeusjava.entity.Note>
	  <com.zeusjava.entity.Note>
	    <title>笔记2</title>
	    <description>语文</description>
	  </com.zeusjava.entity.Note>
	  <com.zeusjava.entity.Note>
	    <title>笔记3</title>
	    <description>英语</description>
	  </com.zeusjava.entity.Note>
	</teacher>

###XStream中转换器的使用

1.创建一个转换器
下面代码创建一乐一个简单的SingleValueConvertor用来将一个对象转换为一个字符串

```java
class NameConverter implements SingleValueConverter {

   public Object fromString(String name) {
      String[] nameparts = name.split(",");
      return new Name(nameparts[0], nameparts[1]);
   }
   
   public String toString(Object name) {
      return ((Name)name).getFirstName() + "," + ((Name)name).getLastName();
   }
   
   public boolean canConvert(Class type) {
      return type.equals(Name.class);
   }	
}
```

2.注册转换器

```java
xstream.registerConverter(new NameConverter());
```

###XStream中的对象流
XStrean提供对java对象流`java.io.ObjectInputStream`和`java.io.ObjectOutputStream`的实现，从而可以实现将XML转换为对象流，
这在要处理大量的对象集合的时候非常有用，一次向内存中读入一个对象，可以提高效率。

创建输出对象流

```java
ObjectOutputStream objectOutputStream = xstream.createObjectOutputStream(new FileOutputStream("test.txt"));
```

创建输入对象流

```java
ObjectInputStream objectInputStream = xstream.createObjectInputStream(new FileInputStream("test.txt"));
```

下面是一个演示代码

```java
package com.zeusjava.test;

import java.io.FileInputStream;
import java.io.FileOutputStream;
import java.io.IOException;
import java.io.ObjectInputStream;
import java.io.ObjectOutputStream;

import com.thoughtworks.xstream.XStream;
import com.thoughtworks.xstream.annotations.XStreamAlias;
import com.thoughtworks.xstream.io.xml.StaxDriver;

public class XStreamTestr {
   public static void main(String args[]){
   
      XStreamTestr tester = new XStreamTestr();
      XStream xstream = new XStream(new StaxDriver());
      
      xstream.autodetectAnnotations(true);
      
      Student student1 = new Student("赵","宏轩");
      Student student2 = new Student("赵","子龙");
      Student student3 = new Student("窦","建德");
      Student student4 = new Student("窦","娜娜");
      
      try {
      
         ObjectOutputStream objectOutputStream = xstream.createObjectOutputStream(new FileOutputStream("test.txt"));
         
         objectOutputStream.writeObject(student1);
         objectOutputStream.writeObject(student2);
         objectOutputStream.writeObject(student3);
         objectOutputStream.writeObject(student4);
         objectOutputStream.writeObject("Hello World");
         
         objectOutputStream.close();
         
         ObjectInputStream objectInputStream = xstream.createObjectInputStream(new FileInputStream("test.txt"));
         
         Student student5 = (Student)objectInputStream.readObject();
         Student student6 = (Student)objectInputStream.readObject();
         Student student7 = (Student)objectInputStream.readObject();
         Student student8 = (Student)objectInputStream.readObject();
         
         String text = (String)objectInputStream.readObject();
         
         System.out.println(student5);
         System.out.println(student6);
         System.out.println(student7);
         System.out.println(student8);
         System.out.println(text);
      
      } catch (IOException e) {
         e.printStackTrace();
         
      } catch (ClassNotFoundException e) {
         e.printStackTrace();
      }
   }
}

@XStreamAlias("student")
class Student {

   private String firstName;
   private String lastName;
   
   public Student(String firstName, String lastName) {
      this.firstName = firstName;
      this.lastName = lastName;
   }

   public String getFirstName() {
      return firstName;
   }

   public String getLastName() {
      return lastName;
   }   

   public String toString(){
      return "Student [ firstName: "+firstName+", lastName: "+ lastName+ " ]";
   }	
}
```

输出结果

	Student [ firstName: 赵, lastName: 宏轩 ]
	Student [ firstName: 赵, lastName: 子龙 ]
	Student [ firstName: 窦, lastName: 建德 ]
	Student [ firstName: 窦, lastName: 娜娜 ]
	Hello World

在`D:\workspace\MyTest`目录下的test.txt中的内容如下

	<?xml version="1.0" encoding="utf-8"?>
	<object-stream>
	  <student>
	    <firstName>赵</firstName>
	    <lastName>宏轩</lastName>
	  </student>
	  <student>
	    <firstName>赵</firstName>
	    <lastName>子龙</lastName>
	  </student>
	  <student>
	    <firstName>窦</firstName>
	    <lastName>建德</lastName>
	  </student>
	  <student>
	    <firstName>窦</firstName>
	    <lastName>娜娜</lastName>
	  </student>
	  <string>Hello World</string>
	</object-stream>

###XStream中XML与Json之间的转换
XStream supports JSON by initializing XStream object with an appropriate driver. 
XStream currently supports JettisonMappedXmlDriver and JsonHierarchicalStreamDriver.

```java
package com.tutorialspoint.xstream;

import java.io.Writer;

import com.thoughtworks.xstream.XStream;
import com.thoughtworks.xstream.annotations.XStreamAlias;
import com.thoughtworks.xstream.io.HierarchicalStreamWriter;
import com.thoughtworks.xstream.io.json.JsonHierarchicalStreamDriver;
import com.thoughtworks.xstream.io.json.JsonWriter;

public class XStreamTester {
   public static void main(String args[]){
   
      XStreamTester tester = new XStreamTester();
      XStream xstream = new XStream(new JsonHierarchicalStreamDriver() {
      
         public HierarchicalStreamWriter createWriter(Writer writer) {
            return new JsonWriter(writer, JsonWriter.DROP_ROOT_MODE);
         }
      });
      
      Student student = new Student("赵","小轩");
      
      xstream.setMode(XStream.NO_REFERENCES);
      xstream.alias("student", Student.class);
      
      System.out.println(xstream.toXML(student));
   }
}

@XStreamAlias("student")
class Student {

   private String firstName;
   private String lastName;
   
   public Student(String firstName, String lastName) {
      this.firstName = firstName;
      this.lastName = lastName;
   }
   
   public String getFirstName() {
      return firstName;
	}

	public String getLastName() {
		return lastName;
	}   
	
	public String toString(){
		return "Student [ firstName: "+firstName+", lastName: "+ lastName+ " ]";
	}	
}
```

控制台输出的结果为：

	{
	   "firstName": "赵",
	   "lastName": "小轩"
	}