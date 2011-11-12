--- 
layout: post
title: Getting Started with JAXB
published: true
meta: 
  _edit_last: "2"
tags: 
- Java
- jaxb
- maven
- schema
- xml
- xsd
type: post
status: publish
---
The <a href="https://jaxb.dev.java.net/">Java Architecture for XML Binding</a> (or JAXB) is an extremely useful library for converting from a Java POJO-style object model to XML and back. In fact it has become so popular that it is now included in the J2EE platform. In this introduction, I will focus mainly on getting your Java objects into an XML representation -- but this is just scratching the surface when it comes to the features of JAXB.
<!--more-->

The first step is creating a simple <a href="http://maven.apache.org/">Maven</a> quickstart project and adding the following dependencies to your POM file:

{% highlight xml %}
<dependency>
     <groupId>javax.xml.bind</groupId>
     <artifactId>jaxb-api</artifactId>
     <version>2.2</version>
</dependency>
<dependency>
     <groupId>com.sun.xml.bind</groupId>
     <artifactId>jaxb-impl</artifactId>
     <version>2.2</version>
</dependency>
{% endhighlight %}

Let's start with a basic example. Personally, I like to start with my Java object model and work my way down. Here are my objects (getters and setters are necessary but not shown).

{% highlight java %}
@XmlRootElement
public class Prison
{
	private Cell cell;
	private Guard guard;
    
        ...
}

public class Cell
{
	private String id;
	private Inmate inmate;

        ...
}

public class Inmate
{
	private String name;
	private String id;
	private String sentence;
	private String description;
	private String history;

        ...
}

public class Guard
{
	private String name;
	private String assignment;

        ...
}
{% endhighlight %}

Notice that only one JAXB annotation is used in my entire object model definition. All I need to do is specify the root element and JAXB will do the rest.

In order to see what XML is generated, just write a simple unit test:

{% highlight java %}
public void testXml() throws JAXBException
{
	// instantiate model
	Prison prison = new Prison();

	Guard guard = new Guard();
	guard.setName( "Jim" );
	guard.setAssignment( "Toilet scrubbing" );

	Inmate inmate = new Inmate();
	inmate.setName( "Billy the Knife" );

	Cell cell = new Cell();
	cell.setId( "CB4" );
	cell.setInmate( inmate );

	prison.setGuard( guard );
	prison.setCell( cell );

	// get instance of JAXBContext based on root class
	JAXBContext context = JAXBContext.newInstance( Prison.class );
		
	// marshall into XML via System.out
	Marshaller marshaller = context.createMarshaller();
	marshaller.setProperty( Marshaller.JAXB_FORMATTED_OUTPUT, true );
	marshaller.marshal( prison, System.out );
}
{% endhighlight %}

The resulting XML will look something like this:

{% highlight xml %}
<prison>
    <cell>
        <id>CB4</id>
        <inmate>
            <name>Billy the Knife</name>
        </inmate>
    </cell>
    <guard>
        <assignment>Toilet scrubbing</assignment>
        <name>Jim</name>
    </guard>
</prison>
{% endhighlight %}

As you can see, JAXB is extremely simple to use and very quick to get started with. If you have a simple model, you only really need to know one annotation! This is a relief for those of us who have struggled with the dozens of tags in Spring and Hibernate. However, JAXB is also useful for more complex models, in which case an XML schema should be provided. I will discuss this further in the future.
