---
layout: post
title:  "XML和Java对象之间的换转利器--XStream的使用"
keywords: "java xstream"
description: "XStream是一个基于java的XML和java对象之间序列化的类库，容易学习，性能好，可选择的格式化（json）输出等优点。"
category: java
tags: jvm-serializers xstream
---
###使用XStream所需要的Jar包

	xstream-1.4.7.jar

###使用步骤

####第一步 创建XStream对象
通过传给XStream一个StaxDriver实例来创建XStream对象，StaxDriver使用StAX转换器，SAX基于事件模式效率性能方面较高。

```java
XStream xstream = new XStream(new StaxDriver());
```

####第二步 将Java对象转换为XML
使用toXML()方法得到java对象的XML字符串。

```java
//Object to XML Conversion
String xml = xstream.toXML(student);
```

####第三步 将XML转换为Java对象
使用fromXML() 方法将Java对象转换为XML文本

```java
//XML to Object Conversion		
Student student1 = (Student)xstream.fromXML(xml);
```
下面是一个简单的Demo

```java
package com.zeusjava.test;

import java.io.ByteArrayInputStream;
import java.io.ByteArrayOutputStream;

import javax.xml.transform.OutputKeys;
import javax.xml.transform.Source;
import javax.xml.transform.Transformer;
import javax.xml.transform.sax.SAXSource;
import javax.xml.transform.sax.SAXTransformerFactory;
import javax.xml.transform.stream.StreamResult;

import org.junit.Test;
import org.xml.sax.InputSource;

import com.thoughtworks.xstream.XStream;
import com.thoughtworks.xstream.io.xml.StaxDriver;
import com.zeusjava.entity.Address;
import com.zeusjava.entity.Student;

public class XStreamTest {
   private XStream xstream = new XStream(new StaxDriver());
	@Test
	public void testXStream(){
		Student student=new Student();
		Address address=new Address();
		address.setCountry("中国");
		address.setProvince("河南省");
		address.setCity("洛阳市");
		address.setDistrict("伊川县");
		student.setAddress(address);
		student.setClassName("高三一班");
		student.setAge(18);
		student.setId("100001");
		student.setName("赵宏轩");
		//Java 对象转换为XML
	    String xml = xstream.toXML(student);
	    System.out.println(formatXml(xml));
	    //XML转换为Object
	    Student student1 = (Student)xstream.fromXML(xml);
	    System.out.println(student1);
	}
	 public static String formatXml(String xml){
	      try{
	         Transformer serializer= SAXTransformerFactory.newInstance().newTransformer();
	         serializer.setOutputProperty(OutputKeys.INDENT, "yes");
	         serializer.setOutputProperty("{http://xml.apache.org/xslt}indent-amount", "2");
	         Source xmlSource=new SAXSource(new InputSource(new ByteArrayInputStream(xml.getBytes())));
	         StreamResult res =  new StreamResult(new ByteArrayOutputStream());            
	         serializer.transform(xmlSource, res);
	         return new String(((ByteArrayOutputStream)res.getOutputStream()).toByteArray());
	      }
	      catch(Exception e){
	         return xml;
	      }
	   }

}


```

需要用到的`Student`和`Address` 类如下

```java
package com.zeusjava.entity;

public class Student {
	   private String id;
	   private String name;
	   private int age;
	   private String className;
	   private Address address;

	   public String getClassName() {
	      return className;
	   }
	   
	   public void setClassName(String className) {
	      this.className = className;
	   }
	   
	   public Address getAddress() {
	      return address;
	   }
	   
	   public void setAddress(Address address) {
	      this.address = address;
	   }
	   
	   public String getId() {
		return id;
	}

	public void setId(String id) {
		this.id = id;
	}

	public String getName() {
		return name;
	}

	public void setName(String name) {
		this.name = name;
	}

	public int getAge() {
		return age;
	}

	public void setAge(int age) {
		this.age = age;
	}

	@Override
	public String toString() {
		return "Student [id=" + id + ", name=" + name + ", age=" + age
				+ ", className=" + className + ", address=" + address + "]";
	}

	
}


```

```java
package com.zeusjava.entity;

public class Address {
	   private String country;
	   private String province;
	   private String city;
	   private String district;
	public String getCountry() {
		return country;
	}
	public void setCountry(String country) {
		this.country = country;
	}
	public String getProvince() {
		return province;
	}
	public void setProvince(String province) {
		this.province = province;
	}
	public String getCity() {
		return city;
	}
	public void setCity(String city) {
		this.city = city;
	}
	public String getDistrict() {
		return district;
	}
	public void setDistrict(String district) {
		this.district = district;
	}
	@Override
	public String toString() {
		return "Address [country=" + country + ", province=" + province
				+ ", city=" + city + ", district=" + district + "]";
	}

}

```

控制台输出结果如下：

	<?xml version="1.0" encoding="UTF-8"?>
	<com.zeusjava.entity.Student>
	  <id>100001</id>
	  <name>赵宏轩</name>
	  <age>18</age>
	  <className>高三一班</className>
	  <address>
	    <country>中国</country>
	    <province>河南省</province>
	    <city>洛阳市</city>
	    <district>伊川县</district>
	  </address>
	</com.zeusjava.entity.Student>

	Student [id=100001, name=大神, age=18, className=高三一班, address=Address [country=中国, province=河南省, city=洛阳市, district=伊川县]]

###在XStream中使用Alias（别名）
别名是一种技术可以自定义生成XML文件或者可以使用特殊的格式化的XStream.
别名有两种一种是`类别名`，一种是`域别名`,`隐式集合别名`,`属性别名`，`包别名`
其中
类别名用来在XML中创建一个完全的类（包含包名）。

	xstream.alias("Teacher", Teacher.class);
	xstream.alias("note", Note.class);

域别名用来在XML中创建一个字段的别名。

	xstream.aliasField("teacherName", Teacher.class, "name");

隐式集合别名用来在XML中当集合将不表示根节点的时候。

	xstream.addImplicitCollection(Teacher.class, "notes");

属性别名用来将一个XML属性序列化一个变量

	xstream.useAttributeFor(Teacher.class, "teacherName");

包别名用来将创建一个完全类名的别名。

	xstream.aliasPackage("com.zeusjava.entity", "com.zeusjava.test");


```java
 @Test
		 public void testAliasing(){
		 Teacher t1=new Teacher("张三");
		 Note n1=new Note("笔记1","数学");
		 Note n2=new Note("笔记2","语文");
		 Note n3=new Note("笔记3","英语");
		 t1.addNote(n1);
		 t1.addNote(n2);
		 t1.addNote(n3);
		 xstream.alias("teacher", Teacher.class);
		 xstream.alias("note", Note.class);
		 //xstream.aliasPackage("com.zeusjava.test", "com.zeusjava.entity");
		 xstream.aliasField("name", Teacher.class, "teacherName");
		 xstream.addImplicitCollection(Teacher.class, "notes");
		 xstream.useAttributeFor(Teacher.class, "teacherName");
		
		 String xml = xstream.toXML(t1);
		 System.out.println(formatXml(xml));
		 
	 }
```
对应的`Teacher`类和`Note`类如下:

```java
package com.zeusjava.entity;

import java.util.ArrayList;
import java.util.List;

public class Teacher {
	private String teacherName;
	   private List<Note> notes = new ArrayList<Note>();
	   
	   public Teacher(String teacherName) {
	      this.teacherName = teacherName;
	   }
	   
	   public void addNote(Note note) {
	      notes.add(note);
	   }
	   
	   public String getTeacherName(){
	      return teacherName;
	   }
	   
	   public List<Note> getNotes(){
	      return notes;
	   }
}

```
Note类如下：

```java
package com.zeusjava.entity;

public class Note {
	private String title;
	private String description;

	public Note(String title, String description) {
		this.title = title;
		this.description = description;
	}

	public String getTitle() {
		return title;
	}

	public String getDescription() {
		return description;
	}
}

```

运行的结果如下


	<?xml version="1.0" encoding="UTF-8"?><teacher name="张三">
	  <note>
	    <title>笔记1</title>
	    <description>数学</description>
	  </note>
	  <note>
	    <title>笔记2</title>
	    <description>语文</description>
	  </note>
	  <note>
	    <title>笔记3</title>
	    <description>英语</description>
	  </note>
	</teacher>

